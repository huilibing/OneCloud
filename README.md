
## 固件包含的插件
```ini
┌ VSSR                  [ Hello World ]  
├ DNSFilter             [ 支持 AdGuardHome/Host/DNSMASQ/Domain 规则, 对广告进行过滤 ]
├ KMS Activator         [ KMS 激活服务器，可用于激活 Windows 或 Office ]  
└ Netease Music         [ 解锁网易云灰色歌曲 ]
```




## 宿主机以及 Docker 相关设置
开启网卡混杂模式
```ini
ip link set eth0 promisc on
```
创建 `Docker` 虚拟网络  

- 虚拟网络名称为 `macnet`，驱动为 `macvlan` 模式  
- 将 `subnet 10.10.10.0`  修改为你自己主路由的网段  
- 将 `geteway 10.10.10.1` 修改为你自己的主路由网关
```ini
docker network create -d macvlan --subnet=10.10.10.0/24 --gateway=10.10.10.1 -o parent=eth0 macnet
```

拉取镜像配置容器
```ini
# 拉取 `OpenWrt` 镜像
docker pull w8ves/openwrt:onecloud

# 启动 `OpenWrt` 容器
docker run --restart always --name openwrt -d --network macnet --ip 10.10.10.11 --privileged w8ves/openwrt:onecloud /sbin/init
```

修改容器网络配置
```ini
# 查看运行中的容器
docker ps -a

# 运行中的容器
CONTAINER ID   IMAGE           COMMAND        CREATED         STATUS        PORTS     NAMES
0bd53ac6f3e8   w8ves/openwrt   "/sbin/init"   9 minutes ago   Up 19 minutes           openwrt

# 进入容器命令行
docker exec -it openwrt /bin/bash

# 修改网络配置
vi /etc/config/network
```
```ini
[按下键盘 Ins 键切至输入状态]

config interface 'loopback'
	option ifname 'lo'
	option proto 'static'
	option ipaddr '127.0.0.1'
	option netmask '255.0.0.0'

config globals 'globals'

config interface 'lan'
	option ifname 'eth0'
	option _orig_ifname 'eth0'
	option _orig_bridge 'true'
	option proto 'static'
	option ipaddr '10.10.10.11'  # 修改为你自己主路由的网段IP
	option netmask '255.255.255.0'
	option gateway '10.10.10.1'  # 修改为你自己主路由的IP
	option dns '10.10.10.1'      # DNS 可填主路由IP 也可填公共DNS

[按下键盘 Esc 键退出输入状态，英文键盘输入 :wq 回车 保存退出]
```
```ini
# 重启容器网络服务
/etc/init.d/network restart

# 退出容器
exit
```

> 永久开启网卡混杂模式
```ini
# 修改配置
vi /etc/network/interfaces
```
```ini
[按下键盘 Ins 键切至输入状态]
source /etc/network/interfaces.d/*

# Wired adapter #1
allow-hotplug eth0
up ip link set eth0 promisc on  # 添加这一行代码
no-auto-down eth0
iface eth0 inet dhcp
......

[按下键盘 Esc 键退出输入状态，英文键盘输入 :wq 回车 保存退出]
```

## 容器 OpenWrt 的相关设置

接口

- `网络` > `接口` > `修改`  
- ☑️ `忽略此接口` > `保存&应用` 

防火墙
- `网络` > `防火墙` > `自定义规则`  
- `复制粘贴下列代码` > `保存&应用` 
```ini
# 国内慢速或无法联网则添加此条命令
iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
```

网络加速
- `网络` > `网络加速`  
- `启用 BBR`☑️ > `保存&应用` 