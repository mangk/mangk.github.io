+++
title = '小米路由器4A千兆版获取ssh权限'
date = 2024-05-21T10:15:35+08:00
draft = false
categories = ['没事就折腾']
+++

# 最近在干什么
最近被人问到怎么将自己电脑上的服务暴露到公网上，加上此前对内网穿透相关技术的了解。决定基于光猫的 IPV6 尝试一下。

经过尝试并结合网上的文章我发现小米路由器本身存在 IPV6 防火墙，所以这里主要介绍如何通过 ssh 登录路由器并关闭防火墙.

网上的文章有很多，但有些重点内容不注意就无法成功，所以这里做一个总结

有关使用移动宽带 IPV6 在公网暴露端口和服务看这里：[传送门](/post/%E4%BD%BF%E7%94%A8%E7%A7%BB%E5%8A%A8%E5%AE%BD%E5%B8%A6ipv6%E5%9C%A8%E5%85%AC%E7%BD%91%E6%9A%B4%E9%9C%B2%E7%AB%AF%E5%8F%A3%E5%92%8C%E6%9C%8D%E5%8A%A1/)

# 获取 SSH 权限

其中的重点包括：

1. `执行程序的电脑要通过网线连接到路由器不能使用 wifi`
2. `程序要在 Mac 或 Linux 环境下运行`
3. `要在 Windows 下运行需要借助 Docker 环境`
4. `程序运行第一次可能不成功，多试两次`
5. `我的路由器固件版本是 2.30.500 亲测这个版本成s功了`
6. `路由器重启后，已经开放的 ssh 端口会关闭，需要重新执行脚本`

### 下载 OpenwrtInvasion 脚本，该脚本用于通过漏洞对官方固件开启 ssh

Github地址：[https://github.com/acecilia/OpenWRTInvasion](https://github.com/acecilia/OpenWRTInvasion)

### 执行命令的两种方式

Windows:

```bash
# 借助 Docker 
# 构建一个包含程序所需运行环境的镜像
docker build -t openwrtinvasion https://github.com/acecilia/OpenWRTInvasion.git
# 使用这个镜像运行
docker run --network host -it openwrtinvasion
```

Mac or Linux:

```bash
# 需要 Python 环境
# 安装依赖
pip3 install -r requirements.txt
# 执行程序
python3 remote_command_execution_vulnerability.py
```

期间需要输入路由器地址，登录密码等信息
![图](/static/img/miwifiroot.png)

成功之后你可以通过以下方式登录到路由器

```bash
done! Now you can connect to the router using several options: (user: root, password: root)
* telnet 192.168.31.1
* ssh -oKexAlgorithms=+diffie-hellman-group-shal -oHostKeyAlgorithms=+ssh-rsa -c 3des-cbc -o UserKnownHostsFile=/dev/null root@192.168.31.1
* ftp: using a program like cyberduck
```

### 关闭防火墙

```bash
ip6tables -F
ip6tables -X
ip6tables -P INPUT ACCEPT
ip6tables -P OUTPUT ACCEPT
ip6tables -P FORWARD ACCEPT
```

[引用点这里](https://blog.csdn.net/jonehoo/article/details/136150009)