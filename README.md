## k2p荒野无灯版启用frp客户端：

由于无灯版自带FRP是Xfrp版本，配置后无法使用，后发现网上小伙伴已经有解决办法。

1、将配置好的frpc.ini上传到K2P目录“/etc/storag”

2、高级设置－自定义设置－脚本－在 WAN 上行/下行启动后执行:  中加上以下代码
```
sleep 10 && wget -P /tmp http://opt.cn2qq.com/opt-file/frpc && chmod 777 /tmp/frpc
/tmp/frpc -c /etc/storage/frpc.ini >/dev/null 2>&1 &
```
或
```
sleep 10 && wget -P /tmp https://raw.githubusercontent.com/pqguanyinli/frpc/master/frpc && chmod 777 /tmp/frpc
/tmp/frpc -c /etc/storage/frpc.ini >/dev/null 2>&1 &
```
3、重启路由

##
frpc frps.ini和frpc.ini 为本人自用，k2p路由器frpc最新客户端下载：http://opt.cn2qq.com/opt-file/frpc

frp 官方使用详细说明：https://github.com/fatedier/frp/blob/master/README_zh.md

frp官方下载地址：https://github.com/fatedier/frp/releases

## FRP内网穿透不到两分钟就学会及扩展运用，轻松实现外网访问esxi后台管理界面、lede软路由后台、群晖NAS及ds photo登录

frp内网穿透比ngrok要简单的多，无需多复杂的配置就可以达到比较好的穿透效果，扩展性也很强。

注意：用国内服务器搭建，需要域名备案才能使用

一、frp的作用

利用处于内网或防火墙后的机器，对外网环境提供 http 或 https 服务。

对于 http, https 服务支持基于域名的虚拟主机，支持自定义域名绑定，使多个域名可以共用一个80端口。

利用处于内网或防火墙后的机器，对外网环境提供 tcp 和 udp 服务，例如在家里通过 ssh 访问处于公司内网环境内的主机

二、配置说明

1、实现功能

（1）外网通过ssh访问内网机器

（2）自定义绑定域名访问内网web服务

2、配置前准备

（1）公网服务器1台

（2）内网服务器1台

（3）公网服务器绑定域名1个

（4）内网服务器部署一个web服务

三、安装frp
1、公网服务器与内网服务器都需要下载frp进行安装

2、下载地址是https://github.com/fatedier/frp/releases

也可以通过命令下载：
```
wget https://github.com/fatedier/frp/releases/download/v0.16.1/frp_0.16.1_linux_amd64.tar.gz
```
3.在WINDOWS下用winscp软件登录，上传frp_0.16.1_linux_amd64.tar.gz至root目录

4.解压文件：
```
tar -zxvf frp_0.16.1_linux_amd64.tar.gz
```
5.进入解压目录：
``` 
cd frp_0.16.1_linux_amd64
```
frps、frps.ini这个两个是服务端文件，frpc、frpc.ini这两个是客户端文件

6.配置服务端：
```
vi ./frps.ini
```
```
[common]
bind_port = 7000        #与客户端绑定的进行通信的端口
vhost_http_port = 80    #访问客户端web服务自定义的端口号
vhost_https_port = 443  
```
按”i”键进行编辑，按“esc”退出编辑状态，输入“:wq”退出

四、启动服务端

临时启动
```
./frps -c ./frps.ini
```
后台保持启动
```
nohup ./frps -c ./frps.ini &
```
五、配置客户端
``` 
vi ./frpc.ini
```
```
[common]  
server_addr = 120.56.37.48      #公网服务器ip  
server_port = 7000              #与服务端bind_port一致  
  
#公网通过ssh访问内部服务器  
[ssh]  
type = tcp                      #连接协议  
local_ip = 127.0.0.1            #内网服务器ip  
local_port = 22                 #ssh默认端口号  
remote_port = 6000              #自定义的访问内部ssh端口号  
  
#公网访问内部web服务器以http方式  
[web]  
type = http                     #访问协议
local_ip = 127.0.0.1            #内网服务器ip 
local_port = 80                 #内网web服务的端口号  
custom_domains = www.veelove.cn,veelove.cn   
#所绑定的公网服务器域名，一级、二级域名都可以，绑定多个域名时用英文“,”分开
```
六、启动客户端

临时启动
```
./frpc -c ./frpc.ini
```
后台保持启动
```
nohup ./frpc -c ./frpc.ini &
```
七、穿透公司代理内网

1.修改服务端配置文件
```
vi ./frps.ini
```
```
[common]
bind_port = 443        #端口号修改为443
vhost_http_port = 80    #访问客户端web服务自定义的端口号 
```
2.修改客户端配置文件
```
vi ./frpc.ini
```
```
[common]
server_addr = 118.24.127.138
server_port = 443                            #端口号修改为443
http_proxy = http://10.168.57.241:8088       #加入公司代理地址

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000

[web]
type = http
local_ip = 127.0.0.1
local_port = 80
custom_domains = www.veelove.cn
```
服务端与客户端启动方式不变，参照四、六。

八、群晖NAS使用frp穿透服务

①.设置ROOT密码，获取群晖的ROOT权限

1.打开控制面板,开启SSH功能
![image](https://github.com/pqguanyinli/frpc/blob/master/images/1.jpg)

 

2.终端输入命令ssh admin@192.168.1.201登录，密码为群辉NAS的用户密码（地址修改为自己的NAS地址，win用户用Putty这个软件登录）


3.输入命令
sudo -i
4.设置root密码
synouser --setpw root XXX
【XXX便是你要修改的密码】

②.客户端调试
1.使用root用户登录群晖6.1
ssh root@192.168.1.201
(地址修改为自己的群晖NAS地址)

2.群晖NAS登陆后台配置文件
[common]
server_addr = 118.24.127.138
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000

[nasweb]  
type = http                   
local_ip = 127.0.0.1           
local_port = 5000                     #群晖NAS登陆地址端口是5000
custom_domains = nas.veelove.cn
2.使用群晖NAS手机APP的DS photo软件在外网访问配置文件
[common]
server_addr = 118.24.127.138
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000

[nasweb]  
type = http                   
local_ip = 127.0.0.1           
local_port = 5000                     #群晖NAS登陆地址端口是5000
custom_domains = nas.veelove.cn

[nasphoto]  
type = tcp                             #协议为TCP协议
local_ip = 127.0.0.1           
local_port = 80
remote_port = 8000                    #需要做一个端口转发才可以实现APP登陆，端口自定义
custom_domains = photo.veelove.cn
九、外网访问esxi后台管理、群晖NAS、群晖NAS客户端DS Photo、LEDE软路由后台
 

[common]
server_addr = 118.24.127.138             #更换为自己服务器IP地址
server_port = 7000

[lede]
type = http                              #协议为http
local_ip = 192.168.1.1                   #保持不变，如果您更换过lede后台地址，请自行修改
local_port = 80
custom_domains = lede.veelove.cn         #更换为自己的域名

[esxi]
type = https                             #协议为https
local_ip = 192.168.1.101                 #更换为自己esxi后台管理地址
local_port = 443
custom_domains = esxi.veelove.cn         #更换为自己的域名

[dsphoto]
type = tcp                               #协议为tcp
local_ip = 127.0.0.1
local_port = 80
remote_port = 8000                       #转发端口可以自己设定
custom_domains = photo.veelove.cn        #更换为自己的域名

[dsm]
type = http                              #协议为http
local_ip = 127.0.0.1
local_port = 5000
custom_domains = nas.veelove.cn          #更换为自己的域名
十、用谷歌云和vultr服务器搭建frp无法使用 需要开放7000端口
解决方法直通车

 

十一、如何让Frp在群晖中自动开机运行
1.进入群晖控制面板》任务计划》新增》触发任务》用户定义的脚本


 

2.添加脚本
 

/root/frp/frpc -c /root/frp/frpc.ini
上面的frp修改为你frp目录的文件名称，我在视频里面建议是修改为frp方便记忆


 

3.按照下图的序号号顺序操作，重启群晖NAS即可


vediotalk大神的frp安装方法 https://drive.google.com/open?id=1dQYczQITgDQPM3YlGAPovSrzhU7WQFvs


