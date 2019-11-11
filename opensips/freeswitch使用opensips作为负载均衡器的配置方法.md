### 环境准备
	192.168.91.129	fs1
	192.168.91.135	fs2
	192.168.91.134	opensips/mysql

### 增加节点，用于分发注册消息
	/usr/local/sbin/opensipsctl dispatcher addgw 1 sip:192.168.91.129 '' 0 50 'fs1' 'freeswitch 1'
	/usr/local/sbin/opensipsctl dispatcher addgw 1 sip:192.168.91.135 '' 0 50 'fs2' 'freeswitch 2'
### 登陆mysql，插入数据增加节点，用于发呼叫消息

	insert into load_balancer (group_id, dst_uri, resources, probe_mode, description) values (1,'sip:192.168.91.129', 'pstn=32', 2,'fs1');
	insert into load_balancer (group_id, dst_uri, resources, probe_mode, description) values (1,'sip:192.168.91.135', 'pstn=32', 2,'fs2');
### FAQ
- 500 command 'ds_reload' not available  
忽略，重启opensips即可，该命令用于重载配置。
