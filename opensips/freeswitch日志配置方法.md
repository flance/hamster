### LOG级别是一个0-7之间的整型

| 数字 | 含义 |  
| -- : | :-- |
|0 | CONSOLE |
|1 | ALERT |
|2 | CRIT |
|3 | ERR |
|4 | WARNING |
|5 | NOTICE |
|6 | INFO |
|7 | DEBUG |

> 级别越高，打印的信息越多。最少也得将级别设置为0，但没办法完全关闭。

## esl的日志级别调整：
	eslSetLogLevel($loglevel)

## console的日志级别调整：
	console loglevel console|alert|crit|err|warning|notice|info|debug
	console loglevel[0-7]
## 日志文件的日志级别调整：
	vi /etc/freeswitch/conf/autoload_configs/logfile.conf.xml

- 取消 `logfile`代码行的注释，使日志输出到指定路径的文件中
- 调整 `all` 代码行的日志级别，根据实际情况选配几个不同的类型或全部

>日志调整后F6重载配置不生效，需要重启freeswitch服务

### sip消息跟踪开关
	fs_cli
	sofia global siptrace on (开启跟踪)
	sofia global siptrace off (关闭跟踪)
### sip消息捕获
	fs_cli
	sofia global capture on (开启捕获)
	sofia global capture off (关闭捕获)