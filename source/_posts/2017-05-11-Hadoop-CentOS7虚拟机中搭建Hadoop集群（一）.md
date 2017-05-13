---
title: Hadoop - CentOS7虚拟机中搭建Hadoop集群（一）
date: 2017-05-11 21:39:16
tags: 
- hadoop
- linux
categories: linux
---

环境：

centos 64 1611

hadoop2.7.3a

jdk1.8.0_131

---

## 1. linux安装

### 1. 安装虚拟机软件（VirtualBox和VMWare都可以）

### 2. 新建centos虚拟机，挂载centos7镜像

### 3. 选择最小安装+开发工具

---

## 2. 安装Hadoop前的准备

安装hadoop可以用root用户也可以新建一个用户,建议新建一个用户。

	
### 1. 把ip地址改为静态ip，以免以后要改来改去

修改网络连接配置文件
    
    vi /etc/sysconfig/network-scripts/ifcfg-eth0
    ...
    BOOTPRORO=static
    IPADDR=ip地址
    ...
    
### 2. 设置DNS

    vi /etc/resolv.conf
    nameserver 114.114.114.114

### 3. 测试一下网络是不是正常

### 4. 修改主机名
	vi /etc/sysconfig/network
	NETWORKING=yes
	HOSTNAME=yourname
	
	:wq
	
### 5. 关防火墙

    #查看防火墙状态
    service iptables status
    #关闭防火墙
    service iptables stop
    #查看防火墙开机启动状态
    chkconfig iptables --list
    #关闭防火墙开机启动
    chkconfig iptables off
    
### 6. 配置ssh免密码登陆

在**每台**服务器上生成公钥密钥，再将**公钥**合并到master的authorized_keys文件上。

**注意！！master和slave都要用相同的user和group  (这里为hadoop)**

1. centos默认关闭ssh免密码登陆，需要到/etc/ssh/sshd_config文件把下面两行取消注释：
    
        RSAAuthentication yes
        PubkeyAuthentication yes

2. 然后，在master和每个slave上分别生成密钥：

        ssh-keygen -t rsa
    
3. 将slave的公钥合并到authorized_keys文件，在master中进入/home/hadoop/.ssh目录，用重定向符号合并：
    ```
    cat id_rsa.pub>> authorized_keys
    ssh hadoop@192.168.0.183 cat ~/.ssh/id_rsa.pub>> authorized_keys
    ssh hadoop@192.168.0.184 cat ~/.ssh/id_rsa.pub>> authorized_keys
    ```

4. 合并完成后，再将master的authorized_keys和known_hosts文件通过scp命令复制到每个slave的.ssh目录里，至此，每台slave和master都互有公钥。
    ```
    scp /home/hadoop/.ssh/known_hosts hadoop@192.168.5.134:/home/hadoop/.ssh/
    scp /home/hadoop/.ssh/authorized_keys hadoop@192.168.5.134:/home/hadoop/.ssh/
    ```
    
5. 回master测试，已经可以无需密码登陆其他slave了(第一次需要密码)，如果失败，看6。
        
        [hadoop@yang .ssh]$ ssh hadoop@192.168.5.132
        Last login: Thu May 11 20:51:47 2017 from 192.168.5.133
        [hadoop@yang01 ~]$

6. 如果免密码登陆失败，到slave把authorized_keys的权限改成“-rw-------”就可以了：
    
        chmod 600 authorized_keys
        [hadoop@yang01 .ssh]$ ll
        -rw-------. 1 hadoop hadoop 1183 5月  11 20:18 authorized_keys



### 7. 安装jdk

1. 到官网下载最新jkd版本的rpm包
2. rpm -ivh jdk**.rpm
3. 配置环境变量
4. java -version 测试


---

