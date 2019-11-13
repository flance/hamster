### 环境准备
	192.168.91.129	fs1
	192.168.91.135	fs2
	192.168.91.134	opensips/mysql
> 负载均衡路由选择逻辑主要基于负载信息。为了对负载情况（处理中的呼叫请求）进行跟踪了解，LB模块使用了DIALOG模块。因此配置opensips的负载均衡时必须首先加载dialog模块

### 增加节点，用于分发注册消息
	/usr/local/sbin/opensipsctl dispatcher addgw 1 sip:192.168.91.138 '' 0 50 'fs1' 'freeswitch 1'
	/usr/local/sbin/opensipsctl dispatcher addgw 1 sip:192.168.91.139 '' 0 50 'fs2' 'freeswitch 2'
### 登陆mysql，插入数据增加节点，用于发呼叫消息

	insert into load_balancer (group_id, dst_uri, resources, probe_mode, description) values (1,'sip:192.168.91.138', 'pstn=32', 2,'fs1');
	insert into load_balancer (group_id, dst_uri, resources, probe_mode, description) values (1,'sip:192.168.91.139', 'pstn=32', 2,'fs2');

### 修改opensips.cfg，配置dispatcher和loadbalancer
	modparam("dispatcher", "ds_probing_mode", 1)
	modparam("dispatcher", "ds_ping_interval", 10)
> 一定要注意opensips.cfg文件中dispatcher和loadbalancer的心跳检测参数探测模式`probing_mode`的参数值不能为0(0代表不做心跳检测，意味着fs永远收不到探测包)

### FAQ
- 500 command 'ds_reload' not available  
没有安装dispatcher模块导致  