## k2p荒野无灯版启用frp客户端：

1、将配置好的frpc.ini上传到K2P目录“/etc/storag”

2、高级设置－自定义设置－脚本－在 WAN 上行/下行启动后执行:  中加上以下代码
```
sleep 10 && wget -P /tmp http://opt.cn2qq.com/opt-file/frpc && chmod 777 /tmp/frpc
/tmp/frpc -c /etc/storage/frpc.ini >/dev/null 2>&1 &
```
或
```
sleep 10 && wget -P /tmp http://opt.cn2qq.com/opt-file/frpc && chmod 777 /tmp/frpc
/tmp/frpc -c /etc/storage/frpc.ini >/dev/null 2>&1 &
```
3、重启路由

## frpc.ini 为本人自用配置文件，k2p路由器frpc最新客户端下载：http://opt.cn2qq.com/opt-file/


