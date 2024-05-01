---
layout: post
title:  "Linux/CentOS 升级 OpenSSH 版本"
date:   2024-05-01 11:44:00 +0800
categories: Linux CentOS
permalink: /archivers/LinuxUpdateOpenSSH
---

# 背景

因为近期 CentOS 服务器被扫描出低版本 OpenSSH 有安全漏洞，需要升级到没有漏洞的新版本。

## 步骤

1. 升级 OpenSSL
2. 升级 OpenSSH

## 下载文件

OpenSSL：https://www.openssl.org/source/

OpenSSH：https://www.openssh.com/portable.html

OpenSSH 国内 mirror：https://mirrors.aliyun.com/pub/OpenBSD/OpenSSH/portable/ 

本次我下载了 openssl-1.1.1w.tar.gz 和 openssh-9.5p1.tar.gz


## 安装依赖

`yum install -y gcc gcc-c++ glibc make autoconf pcre-devel  pam-devel automake makedepend perl-Test-Simple perl zlib zlib-devel`

## 升级 OpenSSL

### 备份文件

此步骤根据 `find / -name openssl` 命令执行的实际情况备份文件

```
find / -name openssl
cp /etc/pki/ca-trust/extracted/openssl /etc/pki/ca-trust/extracted/openssl.bak
cp /usr/bin/openssl /usr/bin/openssl.bak
cp /usr/lib64/openssl /usr/lib64/openssl.bak
```

### 编译安装

```
openssl version
tar -zxvf openssl-1.1.1w.tar.gz
cd openssl-1.1.1w
./config shared -fPIC
make depend
make
make install
echo $?
cp -rvf include/openssl /usr/include/
ln -s /usr/local/bin/openssl /usr/bin/openssl
ln -snf /usr/local/lib64/libssl.so.1.1 /usr/lib64/libssl.so
ln -snf /usr/local/lib64/libssl.so.1.1 /usr/lib64/libssl.so.1.1
ln -snf /usr/local/lib64/libcrypto.so.1.1 /usr/lib64/libcrypto.so
ln -snf /usr/local/lib64/libcrypto.so.1.1 /usr/lib64/libcrypto.so.1.1
echo "/usr/local/lib64" >> /etc/ld.so.conf
ldconfig
openssl version
```

## 升级 OpenSSH

### 备份文件

```
ssh -V

cp /usr/bin/ssh /usr/bin/ssh.bak
cp /usr/sbin/sshd /usr/sbin/sshd.bak
cp /etc/ssh /etc/ssh.bak
```

### 编译安装

```
tar -zxvf openssh-9.5p1.tar.gz
cd openssh-9.5p1
./configure --prefix=/usr/ --sysconfdir=/etc/ssh --with-ssl-dir=/usr/local/ssl --with-zlib --with-md5-passwords --with-pam --with-ssl-engine
make
make install
echo $?
ssh -V
vi /etc/ssh/sshd_config
    添加或修改：
    PermitRootLogin yes
    PasswordAuthentication yes
    UseDNS no
    UsePAM yes
cp -a ./contrib/redhat/sshd.init /etc/init.d/sshd
cp -a ./contrib/redhat/sshd.pam /etc/pam.d/sshd.pam
systemctl stop sshd.service ##不会影响已经连接的会话
mv /usr/lib/systemd/system/sshd.service /usr/lib/systemd/system/sshd.service.bak
systemctl daemon-reload
/etc/init.d/sshd start
cp /run/systemd/generator.late/sshd.service  /usr/lib/systemd/system/sshd.service
systemctl daemon-reload
systemctl restart sshd
systemctl status sshd
systemctl enable sshd
```
