### 基础环境

- 操作系统：CentOS 7.7
- 主服务器 `opensips1`：192.168.1.180
- 备服务器 `opensips2`：192.168.1.184
- 虚拟IP(不是服务器)：192.168.1.110
- keepalived：keepalived-2.0.15.tar.gz

### 注意事项

- 使用root帐号操作
- 主备服务器都要安装keepalived，且安装保持一致
- `keepalived.conf`中的字段`state`、`priority`在主备服务器上有区别，细节见下文

### 关闭并停用防火墙
	systemctl stop firewalld
	systemctl disable firewalld
	vi /etc/selinux/config ，将其中的 SELINUX=enforcing 改为 SELINUX=disabled

### 安装keepalived

方式一：源码编译安装(逐行执行下述命令)

	yum install -y gcc make openssl openssl-devel 
	yum install -y libnl libnfnetlink-devel libnl-devel libnl3-devel
	cd /usr/local/src/
	wget https://www.keepalived.org/software/keepalived-2.0.15.tar.gz	
	tar -xvf keepalived-2.0.15.tar.gz
	cd keepalived-2.0.15
	./configure --prefix=/usr/local/keepalived
	make && make install

	cp /usr/local/src/keepalived-2.0.15/keepalived/etc/init.d/keepalived /etc/init.d/
	cp /usr/local/keepalived/etc/sysconfig/keepalived /etc/sysconfig/
	mkdir /etc/keepalived
	cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/
	cp /usr/local/keepalived/sbin/keepalived /usr/sbin/

	systemctl enable keepalived
	

方式二：yum自动安装(废弃)

	yum install -y keepalived

### 主服务器配置keepalived.conf

	vi /etc/keepalived/keepalived.conf

配置文件中写入如下内容：

	! Configuration File for keepalived
	global_defs {
	   router_id LVS_DEVEL
	   vrrp_skip_check_adv_addr
	   vrrp_garp_interval 0
	   vrrp_gna_interval 0
	}
	
	vrrp_script chk_opensips {  #定义监测opensips服务状态的脚本
	          script "/etc/keepalived/chk_opensips.sh" 
	          interval 2 
	          weight 50 
	}
	
	vrrp_instance VI_1 {
	    state MASTER
	    interface	ens33 #网卡id，需要通过ifconfig命令查看自己的网卡信息
	    virtual_router_id 51
	    priority 100
	    advert_int 1
	    authentication {
	        auth_type PASS
	        auth_pass 1111
	    }
	    virtual_ipaddress {
			192.168.1.110
	    }
		track_script {#列出需要执行的所有脚本
	        chk_opensips
	    }
	}

### 备服务器配置keepalived.conf

	vi /etc/keepalived.conf

配置文件中写入如下内容：

	! Configuration File for keepalived
	
	global_defs {
	   router_id LVS_DEVEL
	   vrrp_skip_check_adv_addr
	   vrrp_garp_interval 0
	   vrrp_gna_interval 0
	}
	
	vrrp_script chk_opensips {  #定义监测opensips服务状态的脚本
	          script "/etc/keepalived/chk_opensips.sh" 
	          interval 2 
	          weight 50 
	}
	 
	vrrp_instance VI_1 {
	    state	BACKUP 
	    interface	ens33 
	    virtual_router_id 51
	    priority	90 
	    advert_int 1
	    authentication {
	        auth_type PASS
	        auth_pass 1111
	    }
	    virtual_ipaddress {
		192.168.1.110
	    }
	    track_script {#列出需要执行的所有脚本
	        chk_opensips
	    }
	}

### 编写chk_opensips.sh

将以下内容写入/etc/keepalived/chk_opensips.sh：

	#!/bin/bash
	#keepalive调用该脚本检测opensips是否已停止
	
	status=$(ps -ef|grep opensips | grep -v grep | grep -v bash | wc -l)
	
	if [ $status = 0 ]; then
		echo "=====检测到系统并未启动opensips相关服务，opensips进程数：" $status
	    opensipsctl start
	    status=$(ps -ef|grep opensips | grep -v grep | grep -v bash | wc -l)
		echo "=====系统尝试自动重启opensips相关服务，opensips进程数：" $status
	    if [ $status = 0  ]; then
			echo "=====系统无法启动opensips相关服务，keepalived将自动停止!!!"
			systemctl stop keepalived 
	    fi
		else
			echo "=====opensips服务正常，opensips进程数：" $status
	fi

保存脚本后改变其权限：

	chmod u+x /etc/keepalived/chk_opensips.sh

### 修改opensips.conf

需要修改每台opensips服务器的配置文件，使其对虚拟IP`192.168.1.110`进行监听，这样才能让客户端访问`192.168.1.110`时正确映射至opensips服务上：

	vi /usr/local/etc/opensips/opensips.cfg

根据实际需要加入对应的监听配置，如下所示：

	listen=tcp:192.168.1.110:5060
	listen=udp:192.168.1.110:5060
	listen=tcp:192.168.1.110:5080
	listen=udp:192.168.1.110:5080

### 修改freeswitch domain

需要修改每台freeswitch服务器的配置文件，增加对虚拟IP`192.168.1.110`的domain支持(可以直接复制原有domain进行修改)，这样才能让来自`192.168.1.110`的呼叫正常路由接起。

### 启动keepavlied

	systemctl start keepalived

### 验证测试
主备服务器各执行如下命令查看ip详情：

	ip add
可以看到虚拟IP当前在主服务器上运行(虚拟IP存在)，然后停止主服务器的keepalived：

	systemctl stop keepalived

再次查看主备服务器ip详情可以看到虚拟IP已经飘至备服务器运行了。

确认上述过程正常后，再次启动主服务器的keepalived，可以看到主服务器又自动接管了虚拟IP，进入master状态：

	systemctl start keepalived

### 连接
在xlite等电话客户端中设置domain为虚拟IP`192.168.1.110`，其他保持不变。

这样电话接入时，系统自动连接虚拟IP`192.168.1.110`，keepalived服务会根据监测情况自动指向正确的opensips服务器，随后opensips服务又会根据各freeswitch服务器的负载情况进行正确的请求路由，最终将客户来电路由至正确的freeswitch服务器上，实现双方通话。