###现象描述
对于`spring boot`新手来说，可能会遇到以下情况：  

直接使用springboot的Application启动服务(jar运行方式，war形式不存在此问题)，服务启动完成后，没有报错，服务自动关闭了，就像运行完成一样。

###原因分析
* 第一种可能，没有引入`spring-boot-starter-web`的依赖，导致缺少web运行环境。
* 第二种可能，引入了tomcat依赖但是依赖范围错写为`<scope>provided</scope>`。

###解决办法
* 第一，添加`spring-boot-starter-web`的依赖。
* 第二，若引入了tomcat依赖，去掉`<scope>provided</scope>`定义即可。


