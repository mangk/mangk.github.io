---
title: 免费泛域名SSL证书
comments: true
banner_img_height: 30
date: 2023-03-09 19:20:34
categories:
tags:
- Server
---
# 使用acme.sh配置泛域名证书
## 安装acme.sh
### 安装
```shell
curl https://get.acme.sh | sh -s email=youEmail@email.com
```
### 生成泛域名证书：
1. 执行命令
```shell
acme.sh --issue --dns -d *.youDomin.com \
 --yes-I-know-dns-manual-mode-enough-go-ahead-please
```
2. 这时候会输出生成如下内容
```shell
Using CA: https://acme.zerossl.com/v2/DV90
Single domain='*.youDomin.com'
Getting domain auth token for each domain
Getting webroot for domain='*.youDomin.com'
Add the following TXT record:
Domain: '_acme-challenge.youDomin.com'
TXT value: 'fdlkajflajfkdlJFKLDSAJFLKJDSLKJF'
Please be aware that you prepend _acme-challenge. before your domain
so the resulting subdomain will be: _acme-challenge.youDomin.com
Please add the TXT records to the domains, and re-run with --renew.
Please add '--debug' or '--log' to check more details.
See: https://github.com/acmesh-official/acme.sh/wiki/How-to-debug-acme.sh
```
去域名管理后台添加dns记录`_acme-challenge.youDomin.com`类型为`TXT`值为`fdlkajflajfkdlJFKLDSAJFLKJDSLKJF`
3. 执行命令
```shell
acme.sh --renew -d *.youDomin.com --yes-I-know-dns-manual-mode-enough-go-ahead-please
```
完成验证。
此时证书会被下载到服务器上
### 配置nginx自动替换证书
创建目录及文件`/etc/nginx/conf.d/ssl/key.pem`,`/etc/nginx/conf.d/ssl/cert.pem`
```shell
acme.sh --install-cert -d *.youDomin.com \
--key-file       /etc/nginx/conf.d/ssl/key.pem  \
--fullchain-file /etc/nginx/conf.d/ssl/cert.pem \
--reloadcmd     "nginx -s reload"
```
此时证书会被复制到`/etc/nginx/conf.d/ssl/`目录下。并自动通过命令`nginx -s reload`重启nginx。所以这里都要替换成自己真是的。
### 这样就算完成了，acme.sh创建的定时任务会自动续期证书并更新证书重启nginx

### 参见：[https://github.com/acmesh-official/acme.sh](https://github.com/acmesh-official/acme.sh)