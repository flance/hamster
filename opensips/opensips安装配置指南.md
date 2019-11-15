## opensips基于centos7的安装配置指南

### 基础环境

- CentOS Linux release 7.7.1908 (Core)
- opensips-2.4.6
- yum, 联网
- 全程使用 `root` 帐号操作

### 修改主机名为 `opensips-proxy`
	hostnamectl set-hostname opensips-proxy
> opensips.cfg若全程绑定的主机名，在特定情况下，服务器重启之后可能会意外绑定到一个错误的映射ip上，导致服务器重启后opensips因ip错误无法启动。
> 因此，一般情况下建议优先使用固定ip地址来配置、绑定opensips.cfg中的各项参数。

### 关闭并停用防火墙(有安全隐患，可使用iptables只开放对应端口)
	systemctl stop firewalld
	systemctl disable firewalld
	vi /etc/selinux/config ，将其中的 SELINUX=enforcing 改为 SELINUX=disabled

### 安装mysql数据库服务

1、centos7缺省没有把mysql的安装源放入库中，因此需要另外安装源：  

	wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm (备用慢速网址：wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm)
	rpm -ivh mysql-community-release-el7-5.noarch.rpm

2、安装mysql

	yum -y install mysql-server

3、启动mysql

	systemctl restart mysqld 或者 service mysqld restart
4、mysql安装之后默认没有密码，需要设置密码：

	mysql -uroot
	set password for 'root'@'localhost' =password('123456');

	// 设置远程访问及全表权限
	grant all privileges on *.* to root@'%'identified by '123456';

	// 更新权限
	flush privileges;


> 操作过程示例如下：  

>	mysql> set password for 'root'@'localhost' =password('123456');  
>	Query OK, 0 rows affected (0.00 sec)

>	mysql> grant all privileges on *.* to root@'%'identified by '123456';  
>	Query OK, 0 rows affected (0.00 sec)

>	mysql> flush privileges;  
>	Query OK, 0 rows affected (0.00 sec)

>	mysql> quit


### 安装opensips服务

1、安装依赖

	yum install -y mysql mysql-server mysql-libs mysql-devel libxml2-devel
	yum install -y gcc make gcc-c++ lynx
	yum install -y flex bison ncurses libncurses-dev ncurses-devel

2、下载opensips

	cd /usr/local/src
	wget https://opensips.org/pub/opensips/2.4.6/opensips-2.4.6.tar.gz
	tar -xvf opensips-2.4.6.tar.gz

3、编译安装opensips

	cd opensips-2.4.6
	make menuconfig

根据命令提示依次进入：`Configure Compile Options` -> `Configure Excluded Modules` -> `db_mysql`
按照此顺序选择 `db_mysql`（空格键为选择键方向键为前进后退和确定）  
选择`db_mysql`后一定要`save`。  
最后选择`Compile And Install Opensips` 等待安装完毕。

### 配置opensip相关参数和服务
1、配置opensips和mysql的连接参数：

	vi /usr/local/etc/opensips/opensipsctlrc

  随后编辑修改以下对应信息即可：

	SIP_DOMAIN= YOUR_PC_IP   //此处写本机地址，如果有代理填写proxy地址
	#数据库 MYSQL ORACLE PGSQL DB_BERKELEY DBTEXT均可
	DBENGINE=MYSQL

	#数据库所在服务器主机名/IP均可
	DBHOST=localhost

	#数据库名称，默认使用opensips，可配置其它名称
	DBNAME=opensips

	#数据库访问用户，主要用于数据的读写，为简便可以统一使用root，若使用其他用户则必须mysql中添加对应用户
	DBWUSER=root

	#数据库访问密码，密码必须为DBWUSER对应用户的密码
	DBWPW=“123456”

	#数据库管理用户，用于数据库、表创建与数据读写等，默认使用root，如有需要可以自行配置mysql用户权限
	DBROOTUSER=“root”

2、配置opensips的安全控制(适用于不使用opensips的负载均衡服务时)

	/usr/local/sbin/osipsconfig

依次选择–> `Generate OpenSIPS Script` –> `Residential Script`–> `Configure Residential Script`，
随后选中如下配置（选择键为空格键）:

	[*] ENABLE_TCP 
	[*] USE_AUTH  
	[*] USE_DBACC  
	[*] USE_DBUSERLOC  
	[*] USE_DIALOG

返回上一级菜单，选择 –> `Save Residential Script` 保存配置项，然后选择 -> `Generate Residential Script` 回车，生成新的配置文件（在 `/usr/local/etc/opensips/` 目录下）

拷贝和替换原有配置文件：

	cd /usr/local/etc/opensips/
	mv opensips.cfg opensips.cfg.old
	mv opensips_residential_2018-5-3_1\:13\:3.cfg opensips.cfg (将自动生成的配置脚本重命名)

编辑opensips.cfg，修改对应的监听地址为机器域名或真实ip，将其中所有以 `CUSTOMIZE ME` 关键字标注释的配置项全部修改一遍(应有5、6处)，使其能正确连接数据库：

	vi opensips.cfg

> 配置示例如下：
>  
	listen=udp:opensips-proxy:5060   # CUSTOMIZE ME
	modparam("usrloc", "db_url",
		"mysql://root:123456@localhost/opensips") # CUSTOMIZE ME
	modparam("acc", "db_url",
		"mysql://root:123456@localhost/opensips") # CUSTOMIZE ME
	modparam("auth_db|uri", "db_url",
		"mysql://root:123456@localhost/opensips") # CUSTOMIZE ME
	modparam("dialog", "db_url",
		"mysql://root:123456@localhost/opensips") # CUSTOMIZE ME

3、初始化数据基础信息

	/usr/local/sbin/opensipsdbctl create

安装过程出现询问时一律回答y即可。
> 安装过程示例如下：  
>
	[root@opensips-proxy log]# /usr/local/sbin/opensipsdbctl create
		MySQL password for root: 
		INFO: test server charset
		INFO: creating database opensips ...
		INFO: Using table engine InnoDB.
		INFO: Core OpenSIPS tables successfully created.
		Install presence related tables? (Y/n): y
		INFO: creating presence tables into opensips ...
		INFO: Presence tables successfully created.
		Install tables for 
				b2b
				cachedb_sql
				call_center
				carrierroute
				cpl
				domainpolicy
				emergency
				fraud_detection
				freeswitch_scripting
				imc
				registrant
				siptrace
				userblacklist
		? (Y/n): y
		INFO: creating extra tables into opensips ...
		INFO: Extra tables successfully created.
		[root@opensips-proxy log]# 

### 启动opensips服务
	opensipsctl start

### 排错手段

默认情况下，opensips的所有日志都会输出到 `/var/log/messages`中，只要实时跟踪即可看到错误描述细节:

	tail -f /var/log/messages

### 重定向日志服务
opensips默认通过 `rsyslog` 服务将日志汇总在 `/var/log/message` 中，根据需要我们可以将其指向独立日志文件 `/var/log/opensips.log` 中：

	vi /etc/rsyslog.conf

在rsyslog.conf文件中追加一行如下配置：

	local0                                              /var/log/opensips.log

> 配置示例如下：
>
	# Save boot messages also to boot.log
	local7.*                                                /var/log/boot.log
	local0                                                  /var/log/opensips.log	
	# ### begin forwarding rule ###
	# The statement between the begin ... end define a SINGLE forwarding
	# rule. They belong together, do NOT split them. If you create multiple
	# forwarding rules, duplicate the whole block!
	# Remote Logging (we use TCP for reliable delivery)

重启rsyslog服务(只有 `opensips.cfg` 中的 `debug_mode=no` 及 `log_stderror=no`，日志才会写入)：

	systemctl restart rsyslog

