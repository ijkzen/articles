---
title: nginx的https证书配置指南
categories: tutorial
---



## 说明

https越来越流行，https的各种好处也就不再赘述，一言以蔽之——**安全**；下面我们来对一个nginx服务器进行https的配置；

## 配置

```shell
git clone https://github.com/certbot/certbot
```

首先下载Let's Encrypt官方工具certbot，certbot需要验证你是这个域名的所有者，所以会让你添加域名的解析记录，一般来说你可以在控制台手动修改，但是证书只有90天的有效期，过期就要重新申请；对于程序员来说，应该采用自动化的方法去做这件事；

有动手能力强的程序员做了这件事，[https://github.com/tengattack/certbot-dns-aliyun](https://github.com/tengattack/certbot-dns-aliyun)；

首先访问 [https://ram.console.aliyun.com](https://ram.console.aliyun.com/)，建立一个新角色，并且为它分配`AliyunDNSFullAccess`权限，与此同时请保存 `AccessKeyId`和`AccessKeySecret`

```shell
git clone https://github.com/tengattack/certbot-dns-aliyun
cd certbot-dns-aliyun
sudo python setup.py install
```

但是在这里由于我们使用的是`certbot-auto`，所以需要运行不同的命令：

```shell
git clone https://github.com/tengattack/certbot-dns-aliyun
cd certbot-dns-aliyun

virtualenv --no-site-packages --python \\
"python2.7" "/opt/eff.org/certbot/venv"

/opt/eff.org/certbot/venv/bin/python2.7 setup.py install
```

然后将之前记录的`AccessKeyId`和`AccessKeySecret`以下面的形式保存到文件aliyun-key.ini：

```yaml
certbot_dns_aliyun:dns_aliyun_access_key = 12345678
certbot_dns_aliyun:dns_aliyun_access_key_secret = 1234567890abcdef1234567890abcdef
```

然后设置权限：

```shell
chmod 600 aliyun-key.ini
```

最后，进行证书的申请：

```shell
certbot-auto certonly -a certbot-dns-aliyun:dns-aliyun \
    --certbot-dns-aliyun:dns-aliyun-credentials /path/to/aliyun-key.ini \
    -d example.com \
    -d "*.example.com"
```

