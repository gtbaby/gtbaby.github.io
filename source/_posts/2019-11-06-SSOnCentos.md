---
title: CentOS 8中Shadowsocks服务器端的安装和优化
tags:
  - ShadowSocks
  - CentOS
date: 2019-11-06 20:55:21
categories: Linux系统
---

# 前言
本次将在Vultr上租的服务器（CentOS 8 环境）中安装Shadowsocks的过程记录下来。以便不时之需。
使用Python 3来安装ss，使用Systemd管理shadowsocks的启动。优化中使用BBR、吞吐量优化和TCP Fast Open.同时记录了将CentOS的某些端口打开的过程。

<!-- more -->

# 安装pip3
使用Python 3，所以要用Python 3的包管理器pip3,首先需要安装pip3：

```bash
sudo yum install python3-pip
```

# 安装Shadowsocks
因为因Shadowsocks作者不再维护`pip`中的Shadowsocks（定格在了2.8.2），我们使用下面的命令来安装最新版的Shadowsocks:
```Bash
pip3 install https://github.com/shadowsocks/shadowsocks/archive/master.zip
```
安装完后，检查下SSServer的版本：
```bash
sudo ssserver --version
```
应该会显示`Shadowsocks 3.0.0`

# 创建配置文件

创建配置文件所在的文件夹，并创建配置文件：
```bash
sudo mkdir /etc/shadowsocks
sudo vim /etc/shadowsocks/config.json
```
填入下面的内容：
```json
{
    "server":"::",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": true
}
```
修改密码，修改端口后，保存即可。

# 测试Shadowsocks

1. 使用`ifconfig`查看下IP地址。
2. 启动Shadowsocks：
```bash
ssserver -c /etc/shadowsocks/config.json
```
3. 回到客户端来，设置shadowsocks，添加服务器，填写服务端IPv4地址，端口号填写上面配置文件里写的`8388`，填写密码，加密方法`aes-256-cfb`，启动系统代理，使用全局模式
4. 用浏览器试试，看能不能访问Google.
5. 测试完毕后，在服务端按`Ctrl + C` 关闭shadowsocks

# 防火墙设置
在上一步中，无法访问的原因有多种，很可能的原因就是那个`8388`端口在服务端没有打开。
在CentOS 8中，使用的是`firewall`进行防火墙设置。
我们需要使用`firewall-cmd`对端口进行开放。
```bash
sudo firewall-cmd --zone=public --add-port=8388/tcp --permanent
sudo firewall-cmd --zone=public --add-port=8388/udp --permanent # 不知道是否使用了udp,就顺便把这个端口的udp也开了吧
```
重启防火墙
```bash
firewall-cmd --reload
```
参数说明：
`–zone` #作用域
`–add-port=8080/tcp`  #添加端口，格式为：端口/通讯协议
`–-permanent` #永久生效，没有此参数重启后失效

如果想要关闭某个端口，则需要使用`--remove-port`关闭某个端口

开启端口后，再次尝试用客户端连接，发现可用了。

# 配置Systemd管理Shadowsocks
新建Shadowsocks管理文件
```bash
sudo vim /etc/systemd/system/shadowsocks-server.service
```
复制粘贴：
```
[Unit]
Description=Shadowsocks Server
After=network.target

[Service]
ExecStart=/usr/local/bin/ssserver -c /etc/shadowsocks/config.json
Restart=on-abort

[Install]
WantedBy=multi-user.target
```
保存退出`:wq`
启动Shadowsocks
```bash
sudo systemctl start shadowsocks-server
```
设置开机启动
```bash
sudo systemctl enable shadowsocks-server
```

# 优化
在使用Shadowsocks时感觉到延迟较大，或吞吐量较低时，可以考虑对服务器端进行优化。

## 开启BBR
BBR系Google最新开发的TCP拥塞控制算法，目前有着较好的带宽提升效果，甚至不比老牌的锐速差。
首先检查服务器Kernel版本：
```bash
uname -r
```
如果其显示版本在4.9.0之下，则需要升级Linux内核。我们的系统内核大于4.9.0故不需要升级。

运行`lsmod | grep bbr`，如果结果中没有`tcp_bbr`，则先运行：
```bash
modprobe tcp_bbr
echo "tcp_bbr" >> /etc/modules-load.d/modules.conf
```
运行:
```bash
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p  #保存生效
```
运行：
```bash
sysctl net.ipv4.tcp_available_congestion_control
sysctl net.ipv4.tcp_congestion_control
```
若均有`bbr`，则开启BBR成功。

## 优化吞吐量
新建配置文件：
```bash
sudo vim /etc/sysctl.d/local.conf
```
复制粘贴：
```
# max open files
fs.file-max = 51200
# max read buffer
net.core.rmem_max = 67108864
# max write buffer
net.core.wmem_max = 67108864
# default read buffer
net.core.rmem_default = 65536
# default write buffer
net.core.wmem_default = 65536
# max processor input queue
net.core.netdev_max_backlog = 4096
# max backlog
net.core.somaxconn = 4096

# resist SYN flood attacks
net.ipv4.tcp_syncookies = 1
# reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
# turn off fast timewait sockets recycling
net.ipv4.tcp_tw_recycle = 0
# short FIN timeout
net.ipv4.tcp_fin_timeout = 30
# short keepalive time
net.ipv4.tcp_keepalive_time = 1200
# outbound port range
net.ipv4.ip_local_port_range = 10000 65000
# max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
# max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
# turn on TCP Fast Open on both client and server side
net.ipv4.tcp_fastopen = 3
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1

net.ipv4.tcp_congestion_control = bbr
```
运行：
```bash
sysctl --system
```
编辑之前的shadowsocks-server.service文件：
```bash
sudo vim /etc/systemd/system/shadowsocks-server.service
```
在`ExecStart`前插入一行，内容为：
```
ExecStartPre=/bin/sh -c 'ulimit -n 51200'
```
保存退出`:wq`
重载shadowsocks-server.service：
```bash
sudo systemctl daemon-reload
```
重启Shadowsocks：
```bash
sudo systemctl restart shadowsocks-server
```

## 开启TCP Fast Open
TCP Fast Open可以降低Shadowsocks服务器和客户端的延迟。实际上在上一步已经开启了TCP Fast Open，现在只需要在Shadowsocks配置中启用TCP Fast Open。
把`/etc/shadowsocks/config.json` 里`fast_open`值改为`true`，保存后，重启shadowsocks服务即可
```bash
sudo systemctl restart shadowsocks-server
```
注意：TCP Fast Open同时需要客户端的支持，即客户端Linux内核版本为3.7.1及以上；你可以在Shadowsocks客户端中启用TCP Fast Open。

# 参考
[Ubuntu 16.04下Shadowsocks服务器端安装及优化](https://www.polarxiong.com/archives/Ubuntu-16-04%E4%B8%8BShadowsocks%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%AB%AF%E5%AE%89%E8%A3%85%E5%8F%8A%E4%BC%98%E5%8C%96.html)