[TOC]



## 一、Kerberos概述

### 1.kerberos简介

```
	kerberos是一种计算机网络认证协议，用来在非安全网络中，对个人通信以安全的手段进行身份认证。这个词又指MIT学院为这个协议开发的一套计算机软件。软件设计上采用客户端/服务器结构，并且能够进行相互认证，即客户端和服务器端均可对对方进行身份认证。可以用于防止窃听、防止重放攻击、保护数据完整性等场合，是一种应用对称密钥体制进行密钥管理的系统。
```

### 2.Kerberos术语

​		Kerberos中有以下一些概念需要了解：

- KDC（Key Distribute Center）：密钥分发中心，负责存储用户信息，管理方法票据
- Realm：Kerberos所管理的一个领域或范围，称之为一个Realm
- Principal：Kerberos所管理的一个用户或者一个服务，可以理解为Kerberos中保存的一个账号，其格式通常如下：primary/instance@realm
- keytab：Kerberos中的用户认证，可以通过密码或者密钥文件证明身份，keytab指密钥文件

### 3.Kerberos认证原理

![](https://yingziimage.oss-cn-beijing.aliyuncs.com/img/202206292023058.png)

## 二、Kerberos安装

### 1.安装Kerberos相关服务

选择一台主机做为Kerberos服务端，安装KDC

```
yum install -y kerb5-server
```

所有客户端主机都需要部署Kerberos客户端

```
yum install -y krb5-workstation krb5-libs
```

### 2.修改配置文件

在服务端主机

```
vim /var/kerberos/krb5kdc/kdc.conf

[kdcdefaults]
 kdc_ports = 88
 kdc_tcp_ports = 88

[realms]
 EXAMPLE.COM = {
  #master_key_type = aes256-cts
  acl_file = /var/kerberos/krb5kdc/kadm5.acl
  dict_file = /usr/share/dict/words
  admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
  supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal camellia256-cts:normal camellia128-cts:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
 }
```

在所有客户端主机

```
vim /etc/krb5.conf

内容如下
# Configuration snippets may be placed in this directory as well
includedir /etc/krb5.conf.d/

[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 dns_lookup_realm = false
 dns_lookup_kdc = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 rdns = false
 pkinit_anchors = FILE:/etc/pki/tls/certs/ca-bundle.crt
 default_realm = EXAMPLE.COM
 #default_ccache_name = KEYRING:persistent:%{uid}

[realms]
 EXAMPLE.COM = {
  kdc = hadoop102
  admin_server = hadoop102
 }
[domain_realm]
# .example.com = EXAMPLE.COM
# example.com = EXAMPLE.COM
```

### 3.初始化KDC数据库

在服务端主机执行以下命令，并根据提示输入密码

```
kdb5_util create -s
```

### 4.修改管理权限配置文件

在服务端主机修改/var/kerberos/krb5kdc/kadm5.acl文件

```
vim /var/kerberos/krb5kdc/kadm5.acl

*/admin@EXAMPLE.COM     *
```

### 5.启动Kerberos相关服务

在主服务端启动KDC，并配置开机自启

```
systemctl start krb5kdc
systemctl enable krb5kdc
```

在主服务端开启Kadmin，该服务为KDC数据库访问入口，并配置开机自启

```
systemctl start kadmin
systemctl enable kadmin
```

### 6.创建Kerberos管理员用户

在主服务端（KDC所处）上执行以下命令，并安装提示输入密码

```
kadmin.local -q "addprinc admin/admin"
```

## 三、Kerberos数据库操作

### 1.登录数据库

1）本地登录（无需认证）

```
kadmin.local
```

2）远程登录（需进行主体认证，认证操作见下文）

```
kadmin
```

### 2.创建Kerberos主体

登录数据库，输入以下命令，并按照提示输入密码

```
kadmin.local:addprinc test
```

### 3.修改主体密码

在数据库中，修改test的密码

```
cpw test
```

### 4.删除Kerberos主体

在数据库中，删除test主体，并输入yes，确定删除

```
delete_principal test
```

### 5.查看所有主体

在数据库中，展示所有创建过的主体

```
list_principals
```

## 四、Kerberos认证操作

### 1.密码认证

1）使用kinit进行主体认证，并按照提示输入密码，例如：对test主体进行认证

```
kinit test
```

2）查看认证凭证

```
klist
```

### 2.密钥文件认证

1）生成主体test的keytab文件到指定目录/root/test.keytab

```
kadmin.local -q "xst -norandkey -k  /root/test.keytab test@EXAMPLE.COM"
```

注：-norandkey的作用是声明不随机生成密码，若**不加该参数**，会导致之前的**密码失效**。

2）使用keytab进行认证

```
kinit -kt /root/test.keytab test
```

### 3.销毁凭证

```
kdestroy
```

