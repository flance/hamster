### 环境准备
	192.168.91.129	fs1
	192.168.91.135	fs2
	192.168.91.134	opensips/mysql
> 负载均衡路由选择逻辑主要基于负载信息。为了对负载情况（处理中的呼叫请求）进行跟踪了解，LB模块使用了DIALOG模块。因此配置opensips的负载均衡时必须首先加载dialog模块（默认情况下opensips自动加载了dialog模块）

### dispatcher 和 loadbalancer 的区别
`dispatcher` 与 `loadbalancer` 都可以实现高可用，但是 `dispatcher` 可用作无状态负载平衡器，不能计算每个目的地的负载情况，因此不能保证公平分配，而 `loadbalancer` 可根据负载提供流量路由，会考虑负荷较小的目标。

在本案例中选择 `loadbalancer` 模块作为负载均衡服务，因此 `dispatcher` 要卸载掉。

### 登陆mysql，插入数据增加节点，用于发呼叫消息

	insert into load_balancer (group_id, dst_uri, resources, probe_mode, description) values (1,'sip:192.168.91.138', 'pstn=32', 2,'fs1');
	insert into load_balancer (group_id, dst_uri, resources, probe_mode, description) values (1,'sip:192.168.91.139', 'pstn=32', 2,'fs2');

### 修改opensips.cfg，配置loadbalancer参数
	### LOAD_BALANCER module
	loadmodule "load_balancer.so"
	modparam("load_balancer", "db_url", "mysql://root:123456@localhost/opensips") # CUSTOMIZE ME
	modparam("load_balancer", "probing_method", "OPTIONS")
	modparam("load_balancer", "probing_interval", 30) #心跳探测间隔秒数
> 一定要注意 `opensips.cfg` 文件中 `loadbalancer` 的心跳检测参数探测模式`probing_mode`的参数值不能为0(0代表不做心跳检测，意味着fs永远收不到探测包)，可以根据需要设置为1或2。

### 配置完成后重启opensips生效
	opensipsctl restart

### 负载均衡状态检查
	opensipsctl fifo lb_list 列出所有目标以及目标的每个资源的最大和当前负载。

> 示例如下：  
> opensipsctl fifo lb_list  
> Destination:: sip:192.168.91.138 id=1 group=1 enabled=yes auto-reenable=on  
>        Resources::   
>                Resource:: pstn max=32 load=0  
>Destination:: sip:192.168.91.139 id=2 group=1 enabled=no auto-reenable=on  
>        Resources::   
>                Resource:: pstn max=32 load=0  

除了在opensips服务器上使用 `opensipsctl fifo lb_list`命令检测状态之外，还可以在每台freeswitch服务上跟踪sip消息，可以看到一直有opensips发来的心跳探测报文。

### 收尾
负载均衡配置完成后，sip电话客户端可以直接拨至 `192.168.91.134:5060` 即可，根据负载情况，opensips会自动将请求路由到 `192.168.91.129:5060` 或 `192.168.91.129:5060` 上。
