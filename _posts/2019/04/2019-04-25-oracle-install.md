---
layout: post
title: CentOS7静默安装、停止与删除oracle
category: tools
tags: [tools]
---

参考[https://blog.csdn.net/liby_sunny/article/details/86635844](https://blog.csdn.net/liby_sunny/article/details/86635844)
参考[https://blog.51cto.com/13001751/1983190](https://blog.51cto.com/13001751/1983190)
参考[https://blog.csdn.net/storyteller321/article/details/81162032](https://blog.csdn.net/storyteller321/article/details/81162032)

# 一.准备工作

### 1\. centos7服务器一台（自备）

### 2\. oracle安装包

官方下载地址：[http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html)

根据自己服务器的情况选择对应的安装包下载，以我的为例，是64位操作系统的。下载完成后，有两个压缩文件linux.x64_11gR2_database_1of2.zip 和 linux.x64_11gR2_database_2of2.zip。

### 3.安装位置的选择

oracle尽量安装在剩余空间充足的位置，因此首先要查看服务器硬盘情况，如下：

    [root@bogon ~]# df -h
    Filesystem               Size  Used Avail Use% Mounted on
    /dev/mapper/centos-root   50G  8.5G   42G  17% /
    devtmpfs                  16G     0   16G   0% /dev
    tmpfs                     16G     0   16G   0% /dev/shm
    tmpfs                     16G  9.0M   16G   1% /run
    tmpfs                     16G     0   16G   0% /sys/fs/cgroup
    /dev/sda1               1014M  180M  835M  18% /boot
    /dev/mapper/centos-home  142G   37M  142G   1% /home
    tmpfs                    3.2G   40K  3.2G   1% /run/user/0
    /dev/sdb1                493G  825M  467G   1% /data 

对比后发现/data余量最大，因此决定将oracle安装在/data目录

### 4.上传安装包

把下载下来的两个安装包通过xftp上传到/data目录（开始上传后继续执行后边的目录即可，无需等待上传完成）。

### 5.**安装依赖包**

安装依赖包之前，建议将yum源修改为aliyun源，下载速度快些：

    [root@iz8vb8edqeyilgy4r9zci6z ~]# cd /etc
    [root@iz8vb8edqeyilgy4r9zci6z etc]# mv yum.repos.d yum.repos.d.bak
    [root@iz8vb8edqeyilgy4r9zci6z etc]# mkdir yum.repos.d
    [root@iz8vb8edqeyilgy4r9zci6z etc]# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
    --2019-01-25 09:15:40--  http://mirrors.aliyun.com/repo/Centos-7.repo
    Resolving mirrors.aliyun.com (mirrors.aliyun.com)... 101.37.183.142, 101.37.183.145, 101.37.183.169, ...
    Connecting to mirrors.aliyun.com (mirrors.aliyun.com)|101.37.183.142|:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 2523 (2.5K) [application/octet-stream]
    Saving to: ‘/etc/yum.repos.d/CentOS-Base.repo’
    
    100%[=========================================================================================================================================================>] 2,523       --.-K/s   in 0s      
    
    2019-01-25 09:15:40 (289 MB/s) - ‘/etc/yum.repos.d/CentOS-Base.repo’ saved [2523/2523]
    
    [root@iz8vb8edqeyilgy4r9zci6z etc]# yum clean all
    Loaded plugins: fastestmirror
    Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
    Cleaning repos: base extras updates
    Cleaning up everything
    Cleaning up list of fastest mirrors
    [root@iz8vb8edqeyilgy4r9zci6z etc]# yum makecache
    Loaded plugins: fastestmirror
    base                                                                                                                                                                        | 3.6 kB  00:00:00     
    extras                                                                                                                                                                      | 3.4 kB  00:00:00     
    updates                                                                                                                                                                     | 3.4 kB  00:00:00     
    (1/12): base/7/x86_64/group_gz                                                                                                                                              | 166 kB  00:00:00     
    (2/12): base/7/x86_64/primary_db                                                                                                                                            | 6.0 MB  00:00:00     
    (3/12): base/7/x86_64/filelists_db                                                                                                                                          | 7.1 MB  00:00:00     
    (4/12): base/7/x86_64/other_db                                                                                                                                              | 2.6 MB  00:00:00     
    (5/12): extras/7/x86_64/prestodelta                                                                                                                                         |  36 kB  00:00:00     
    (6/12): extras/7/x86_64/filelists_db                                                                                                                                        | 189 kB  00:00:00     
    (7/12): extras/7/x86_64/primary_db                                                                                                                                          | 156 kB  00:00:00     
    (8/12): extras/7/x86_64/other_db                                                                                                                                            | 107 kB  00:00:00     
    (9/12): updates/7/x86_64/prestodelta                                                                                                                                        | 190 kB  00:00:00     
    (10/12): updates/7/x86_64/filelists_db                                                                                                                                      | 1.4 MB  00:00:00     
    (11/12): updates/7/x86_64/primary_db                                                                                                                                        | 1.3 MB  00:00:00     
    (12/12): updates/7/x86_64/other_db                                                                                                                                          | 204 kB  00:00:00     
    Determining fastest mirrors
     * base: mirrors.cloud.aliyuncs.com
     * extras: mirrors.cloud.aliyuncs.com
     * updates: mirrors.cloud.aliyuncs.com
    Metadata Cache Created

修改成功后，安装如下依赖包：

    yum -y install binutils compat-libstdc++-33 elfutils-libelf elfutils-libelf-devel expat gcc gcc-c++ glibc glibc-common glibc-devel glibc-headers libaio libaio-devel libgcc libstdc++ libstdc++-devel make pdksh sysstat unixODBC unixODBC-devel

安装完成后界面如下：

![](https://img-blog.csdnimg.cn/20190125092614971.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpYnlfc3Vubnk=,size_16,color_FFFFFF,t_70)

使用如下命令检查依赖是否安装完整：

    rpm -q binutils compat-libstdc++-33 elfutils-libelf elfutils-libelf-devel expat gcc gcc-c++ glibc glibc-common glibc-devel glibc-headers libaio libaio-devel libgcc libstdc++ libstdc++-devel make pdksh sysstat unixODBC unixODBC-devel | grep "not installed"

发现pdksh没有安装，如下图：

![](https://img-blog.csdnimg.cn/20190125093030573.png)

通过wget命令直接下载pdksh的rpm包，我下载到了/tmp/，然后安装

    [root@bogon ~]# wget -O /tmp/pdksh-5.2.14-37.el5_8.1.x86_64.rpm http://vault.centos.org/5.11/os/x86_64/CentOS/pdksh-5.2.14-37.el5_8.1.x86_64.rpm
    --2019-01-25 09:31:23--  http://vault.centos.org/5.11/os/x86_64/CentOS/pdksh-5.2.14-37.el5_8.1.x86_64.rpm
    Resolving vault.centos.org (vault.centos.org)... 208.100.23.71, 2607:f128:40:1600:225:90ff:fe00:bde6
    Connecting to vault.centos.org (vault.centos.org)|208.100.23.71|:80... connected.
    HTTP request sent, awaiting response... 302 Found
    Location: http://120.52.51.16/vault.centos.org/5.11/os/x86_64/CentOS/pdksh-5.2.14-37.el5_8.1.x86_64.rpm [following]
    --2019-01-25 09:31:24--  http://120.52.51.16/vault.centos.org/5.11/os/x86_64/CentOS/pdksh-5.2.14-37.el5_8.1.x86_64.rpm
    Connecting to 120.52.51.16:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 210877 (206K) [application/x-rpm]
    Saving to: ‘/tmp/pdksh-5.2.14-37.el5_8.1.x86_64.rpm’
    
    100%[=========================================================================================================================================================>] 210,877     --.-K/s   in 0.03s   
    
    2019-01-25 09:31:24 (7.63 MB/s) - ‘/tmp/pdksh-5.2.14-37.el5_8.1.x86_64.rpm’ saved [210877/210877]
    [root@bogon ~]# cd /tmp/
    [root@bogon tmp]# rpm -ivh pdksh-5.2.14-37.el5_8.1.x86_64.rpm
    warning: pdksh-5.2.14-37.el5_8.1.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID e8562897: NOKEY
    Preparing...                          ################################# [100%]
    Updating / installing...
       1:pdksh-5.2.14-37.el5_8.1          ################################# [100%]

如果rpm安装pdksh-5.2.14报错

    [root@oracle ~]# rpm -lvh pdksh-5.2.14-37.el5_8.1.x86_64.rpm  
    rpm: --hash (-h) may only be specified during package installation
    [root@oracle ~]# rpm -ivh pdksh-5.2.14-37.el5_8.1.x86_64.rpm   
    warning: pdksh-5.2.14-37.el5_8.1.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID e8562897: NOKEY
    error: Failed dependencies:
            pdksh conflicts with ksh-20120801-33.el6.x86_64
解决办法

    [root@oracle ~]#  rpm -qa | grep ksh
    ksh-20120801-33.el6.x86_64
    [root@oracle ~]# rpm -e ksh-20120801-33.el6.x86_64
    [root@oracle ~]#  rpm -qa | grep ksh   
再次尝试

    [root@oracle ~]# rpm -ivh pdksh-5.2.14-37.el5_8.1.x86_64.rpm 
    warning: pdksh-5.2.14-37.el5_8.1.x86_64.rpm: Header V3 DSA/SHA1 Signature, key ID e8562897: NOKEY
    Preparing...                ########################################### [100%]
       1:pdksh                  ########################################### [100%]
                  
再次检查依赖包是否安装完整

    rpm -q binutils compat-libstdc++-33 elfutils-libelf elfutils-libelf-devel expat gcc gcc-c++ glibc glibc-common glibc-devel glibc-headers libaio libaio-devel libgcc libstdc++ libstdc++-devel make pdksh sysstat unixODBC unixODBC-devel | grep "not installed"

发现所有依赖都已经安装完毕，至此依赖包安装完成

![](https://img-blog.csdnimg.cn/20190125093730973.png)

### 6.**添加oracle用户组和用户**

设置密码时pssswd oracle后直接输密码回车即可。
    
    [root@bogon tmp]# groupadd oinstall
    [root@bogon tmp]# groupadd dba
    [root@bogon tmp]# useradd -g oinstall -G dba oracle -d /home/oracle
    [root@bogon tmp]# passwd oracle
    Changing password for user oracle.
    New password: 
    BAD PASSWORD: The password is shorter than 8 characters
    Retype new password: 
    passwd: all authentication tokens updated successfully.
    [root@bogon tmp]# id oracle
    uid=1001(oracle) gid=1001(oinstall) groups=1001(oinstall),1002(dba)
### 7.**配置hostname**
    
    [root@bogon tmp]# vim /etc/hosts
    # 编辑状态添加如下内容
    服务器内网IP oracle
    [root@bogon tmp]# ping -c 2 oracle
    PING oracle (服务器内网IP) 56(84) bytes of data.
    64 bytes from oracle (服务器内网IP): icmp_seq=1 ttl=64 time=0.049 ms
    64 bytes from oracle (服务器内网IP): icmp_seq=2 ttl=64 time=0.032 ms
    
    --- oracle ping statistics ---
    2 packets transmitted, 2 received, 0% packet loss, time 999ms
    rtt min/avg/max/mdev = 0.032/0.040/0.049/0.010 ms
### 8.**优化OS内核参数**

**_kernel.shmmax 参数设置为物理内存的一半_**
    
    [root@bogon tmp]# vim /etc/sysctl.conf 
    # 编辑状态输入如下内容/或修改对应的值
    fs.aio-max-nr=1048576
    fs.file-max=6815744
    kernel.shmall=2097152
    kernel.shmmni=4096
    kernel.shmmax = 8589934592
    kernel.sem=250 32000 100 128
    net.ipv4.ip_local_port_range=9000 65500
    net.core.rmem_default=262144
    net.core.rmem_max=4194304
    net.core.wmem_default=262144
    net.core.wmem_max=1048586
    [root@bogon tmp]# sysctl -p
    fs.aio-max-nr = 1048576
    fs.file-max = 6815744
    kernel.shmall = 2097152
    kernel.shmmni = 4096
    kernel.shmmax = 8589934592
    kernel.sem = 250 32000 100 128
    net.ipv4.ip_local_port_range = 9000 65500
    net.core.rmem_default = 262144
    net.core.rmem_max = 4194304
    net.core.wmem_default = 262144
    net.core.wmem_max = 1048586
### 9.**创建oracle安装目录**
    
    [root@bogon data]# mkdir -p /data/oracle/oracle/product/11.2.0
    [root@bogon data]# mkdir /data/oracle/oracle/oradata
    [root@bogon data]# mkdir /data/oracle/oracle/inventory
    [root@bogon data]# mkdir /data/oracle/oracle/fast_recovery_area
    # 修改文件夹从属
    [root@bogon data]# chown -R oracle:oinstall /data/oracle/oracle/
    # 修改权限
    [root@bogon data]# chmod -R 775 /data/oracle/oracle/
### 10.**配置oracle用户环境变量**
    
    [root@bogon data]# su - oracle
    Attempting to create directory /home/oracle/perl5
    [oracle@bogon ~]$ vim .bash_profile 
    # 进入编辑状态后添加如下代码
    umask 022
    export ORACLE_HOSTNAME=oracle
    export ORACLE_BASE=/data/oracle/oracle
    export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/
    export ORACLE_SID=ORCL
    export PATH=.:$ORACLE_HOME/bin:$ORACLE_HOME/OPatch:$ORACLE_HOME/jdk/bin:$PATH
    export LC_ALL="en_US"
    export LANG="en_US"
    export NLS_LANG="AMERICAN_AMERICA.ZHS16GBK"
    export NLS_DATE_FORMAT="YYYY-MM-DD HH24:MI:SS"
    [oracle@bogon ~]$ source .bash_profile

**此步完成建议重启一下服务器，如果未重启，进行下边操作的时候先切换回root用户**

### 11.**解压oracle压缩文件，进行安装前的配置**
    
    [root@bogon data]# cd /data
    [root@bogon data]# unzip linux.x64_11gR2_database_1of2.zip -d /data/oracle/
    [root@bogon data]# unzip linux.x64_11gR2_database_2of2.zip -d /data/oracle/
    # 解压完成后进行安装前的配置
    [root@bogon data]# mkdir /data/oracle/etc
    [root@bogon data]# cp /data/oracle/database/response/* /data/oracle/etc/
    [root@bogon data]# vim /data/oracle/etc/db_install.rsp
    # 进入编辑状态修改如下项的值
    oracle.install.option=INSTALL_DB_SWONLY
    DECLINE_SECURITY_UPDATES=true
    UNIX_GROUP_NAME=oinstall
    INVENTORY_LOCATION=/data/oracle/oracle/inventory
    SELECTED_LANGUAGES=en,zh_CN
    ORACLE_HOSTNAME=oracle
    ORACLE_HOME=/data/oracle/oracle/product/11.2.0
    ORACLE_BASE=/data/oracle/oracle
    oracle.install.db.InstallEdition=EE
    oracle.install.db.isCustomInstall=true
    oracle.install.db.DBA_GROUP=dba
    oracle.install.db.OPER_GROUP=dba
# 二.安装

### 1.执行静默安装

    su - oracle
    cd /data/oracle/database
    
# 可以看见database文件夹下有三个模板其中dbca.rsp是用来创建数据库的。db_install.rsp是用来安装Oracle软件的。netca.rsp是用来创建监听器的
    
    ./runInstaller -silent -ignorePrereq -responseFile /data/oracle/etc/db_install.rsp

正常会进入如下过程：

![](https://img-blog.csdnimg.cn/20190125104217493.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpYnlfc3Vubnk=,size_16,color_FFFFFF,t_70)

最下边一行为日志文件的位置，安装过程中可以通过tail -f xxx.log查看当前日志，当前情况如下

    tail -f /data/oracle/oracle/inventory/logs/installActions2019-01-25_10-41-53AM.log

安装成功后情况如下：

![](https://img-blog.csdnimg.cn/20190125104858664.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpYnlfc3Vubnk=,size_16,color_FFFFFF,t_70)

在此步骤经常遇到的问题：

centos 安装oracle 报Checking swap space: 0 MB available, 150 MB required. Failed
参考[https://www.cnblogs.com/a9999/p/6957280.html](https://www.cnblogs.com/a9999/p/6957280.html)


按照上边提示的内容进行操作：
    
1.打开一个新的shell窗口

2\. 登录root账号

3.执行脚本（脚本位置在红框前两行有标注）

4.回到当前窗口按回车
    
    [root@bogon ~]# sh /data/oracle/oracle/inventory/orainstRoot.sh
    Changing permissions of /data/oracle/oracle/inventory.
    Adding read,write permissions for group.
    Removing read,write,execute permissions for world.
    
    Changing groupname of /data/oracle/oracle/inventory to oinstall.
    The execution of the script is complete.
    [root@bogon ~]# sh /data/oracle/oracle/product/11.2.0/root.sh
    Check /data/oracle/oracle/product/11.2.0/install/root_bogon_2019-01-25_10-51-48.log for the output of root script

### 2.**配置静默监听**
在另外一个窗口用root执行那两个脚本后，之前oracle的窗口需要关掉重新开，否则监听配置不成功
    
    [root@bogon ~]# su - oracle
    Last login: Fri Jan 25 10:40:23 CST 2019 on pts/0
    [oracle@bogon ~]$ netca /silent /responsefile /data/oracle/etc/netca.rsp 
    
    Parsing command line arguments:
        Parameter "silent" = true
        Parameter "responsefile" = /data/oracle/etc/netca.rsp
    Done parsing command line arguments.
    Oracle Net Services Configuration:
    Profile configuration complete.
    Listener "LISTENER" already exists.
    Oracle Net Services configuration successful. The exit code is 0
    
### 3.**静默创建数据库**
    
**注：修改配置文件为root用户，静默建库为oracle用户**
    
    [root@bogon ~]# vim /data/oracle/etc/dbca.rsp
    # 修改如下配置
    GDBNAME = "orcl"
    SID = "orcl"
    SYSPASSWORD = "oracle"
    SYSTEMPASSWORD = "oracle"
    SYSMANPASSWORD = "oracle"
    DBSNMPPASSWORD = "oracle"
    DATAFILEDESTINATION =/data/oracle/oracle/oradata
    RECOVERYAREADESTINATION=/data/oracle/oracle/fast_recovery_area
    CHARACTERSET = "AL32UTF8"
    TOTALMEMORY = "1638"
    
修改完成后执行静默建库 GDBNAME为ORCL

    [oracle@bogon etc]$ dbca -silent -createDatabase -templateName General_Purpose.dbc -gdbName ORCL -sysPassword oracle -systemPassword oracle
### 4.查看监听状态
    
    [oracle@iz8vb8edqeyilgy4r9zci6z ~]$ lsnrctl status
    
    LSNRCTL for Linux: Version 11.2.0.1.0 - Production on 25-JAN-2019 16:28:33
    
    Copyright (c) 1991, 2009, Oracle.  All rights reserved.
    
    Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=IPC)(KEY=EXTPROC1521)))
    STATUS of the LISTENER
    ------------------------
    Alias                     LISTENER
    Version                   TNSLSNR for Linux: Version 11.2.0.1.0 - Production
    Start Date                26-DEC-2018 23:40:53
    Uptime                    29 days 16 hr. 47 min. 40 sec
    Trace Level               off
    Security                  ON: Local OS Authentication
    SNMP                      OFF
    Listener Parameter File   /data/oracle/oracle/product/11.2.0/network/admin/listener.ora
    Listener Log File         /data/oracle/oracle/diag/tnslsnr/iz8vb8edqeyilgy4r9zci6z/listener/alert/log.xml
    Listening Endpoints Summary...
      (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
      (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=iz8vb8edqeyilgy4r9zci6z)(PORT=1521)))
    Services Summary...
    Service "ORCL" has 1 instance(s).
      Instance "ORCL", status READY, has 1 handler(s) for this service...
    Service "orclXDB" has 1 instance(s).
      Instance "ORCL", status READY, has 1 handler(s) for this service...
    The command completed successfully
### 5.启动数据库

   
    [oracle@iz8vb8edqeyilgy4r9zci6z ~]$ sqlplus / as sysdba
    
    SQL*Plus: Release 11.2.0.1.0 Production on Fri Jan 25 16:30:40 2019
    
    Copyright (c) 1982, 2009, Oracle.  All rights reserved.
    
    Connected to an idle instance.
    
    SQL> startup
    ORACLE instance started.
    
    Total System Global Area 3273641984 bytes
    Fixed Size		    2217792 bytes
    Variable Size		 1795164352 bytes
    Database Buffers	 1459617792 bytes
    Redo Buffers		   16642048 bytes
    Database mounted.
    Database opened.
    
在此步骤经常遇到的问题：

查询告警日志位置:SQL> show parameter background_dump_dest;

ORA-00845: MEMORY_TARGET not supported on this system 

参考[https://www.cnblogs.com/a9999/p/6957280.html](https://www.cnblogs.com/a9999/p/6957280.html)

ORA-01102 cannot mount database in EXCLUSIVE mode 

参考 [https://blog.csdn.net/lzwgood/article/details/26368323](https://blog.csdn.net/lzwgood/article/details/26368323)

ORA-01507

    查了一下，说在alter database 时，还没有mount。
    
    然后到命令行下设置oracle_sid
    
    sqlplus /nolog
    
    connect / as sysdba
    
    startup mount;
    
    alter database open;
    
    shutdown;
    
    startup

### 6.开端口号
    
    [oracle@iz8vb8edqeyilgy4r9zci6z ~]$ firewall-cmd --zone=public --add-port=1521/tcp --permanent
    [oracle@iz8vb8edqeyilgy4r9zci6z ~]$ firewall-cmd --reload
    
    在此步骤经常遇到的问题：
    需要查看防火墙状态
### 7.修改charset

参考[https://blog.csdn.net/yixia/article/details/4029822](https://blog.csdn.net/yixia/article/details/4029822)

参考[https://blog.csdn.net/u014330421/article/details/78655468](https://blog.csdn.net/u014330421/article/details/78655468)

#创建表空间被navicat连接

   
参考[https://www.showdoc.cc/page/1932265630854490](https://www.showdoc.cc/page/1932265630854490)
参考[https://www.cnblogs.com/little-mat/articles/2193767.html](https://www.cnblogs.com/little-mat/articles/2193767.html)
    
### 8.强制停止ORACLE数据库

# 适用场景

      shutdown immediate停止数据库失败
    
    # 操作命令
    
      1、kill掉oracle实例相关进程
    
     ps -ef | grep ora_ | grep -v grep | awk '{print $2}' | xargs kill -9
    
      2、清除oracle占用的共享内存段
    
     ipcs -m | grep oracle | grep -v grep | awk '{print $2}' | xargs -n 1 ipcrm -m
    
      3、清除oracle占用的共享信号量
    
     ipcs -s | grep oracle | grep -v grep | awk '{print $2}' | xargs -n 1 ipcrm -s

### 9.建立测试表空间及账户
    
     create tablespace TMP_LZX datafile '/data/oracle/oracle/oradata/TMP_LZX.dbf' size 50M autoextend on next 5M maxsize 100M;
    
     create user TMP_LZX identified by TMP_LZX default tablespace TMP_LZX;
    
     grant connect,resource,dba,sysdba to TMP_LZX;
     
# 三.重启

    用root以ssh登录到linux，打开终端输入以下命令：
    
    cd $ORACLE_HOME   #进入到oracle的安装目录  
    dbstart           #重启服务器  
    lsnrctl start     #重启监听器  
    cd $ORACLE_HOME   #进入到oracle的安装目录
    dbstart           #重启服务器
    lsnrctl start     #重启监听器
    
# 四.删除


**1、停数据库，关监听**
    
    1\. [oracle@Boss 桌面]$ sqlplus / as sysdba  
    2\. SQL*Plus: Release 12.1.0.2.0 Production on Sun Jul 22 23:21:39 2018  
    3\. Copyright (c) 1982, 2014, Oracle.  All rights reserved.  
    4\. Connected to:  
    5\. Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production
    6\. With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options  
    7\. SQL> shutdown immediate  
    8\. Database closed.  
    9\. Database dismounted.  
    10\. ORACLE instance shut down.  
    11\. SQL> exit  
    12\. Disconnected from Oracle Database 12c Enterprise Edition Release 12.1.0.2.0 - 64bit Production  
    13\. With the Partitioning, OLAP, Advanced Analytics and Real Application Testing options  
    14\. [oracle@Boss 桌面]$ lsnrctl stop  
    15\. 16\. LSNRCTL for Linux: Version 12.1.0.2.0 - Production on 22-JUL-2018 23:22:30  
    17\. Copyright (c) 1991, 2014, Oracle.  All rights reserved.  
    18\. Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=localhost)(PORT=1521)))  
    19\. The command completed successfully 
    
    **2、使用deinstall工具删除oracle可执行和配置文件**
    
    20. [oracle@Boss 桌面]$ cd /u01/app/oracle/12C/deinstall/  
    21. [oracle@Boss deinstall]$ ./deinstall  
    22. Checking for required files and bootstrapping ...  
    23. Please wait ...  
    24. 日志的位置 /u01/app/oraInventory/logs/  
    25. ############ ORACLE DECONFIG TOOL START ############ 
    26. ######################### DECONFIG CHECK OPERATION START ###################
    27. ## [开始] 安装检查配置 ## 
    28. ...省略
    29. ...省略
    30. CCR 检查已完成  
    31. 是否要继续 (是 - 是, 否 - 否)? [否]: 是  
    32. 此会话的日志将写入: '/u01/app/oraInventory/logs/deinstall_deconfig2018-07-22_11-30-51-PM.out'  
    33. Oracle 卸载工具已成功清除临时目录。  
    34. ############# ORACLE DEINSTALL TOOL END ############# 
    
    **3、删除/etc目录下的oraInst.loc、oratab，删除/opt目录下的ORCLfmap**
    
    35. [oracle@Boss deinstall]$ rm -rf /etc/oraInst.loc   
    36. rm: 无法删除"/etc/oraInst.loc": 权限不够  
    37. [oracle@Boss deinstall]$ su -  
    38. 密码：  
    39. [root@Boss ~]# rm -rf /etc/oraInst.loc 
    40. [root@Boss ~]# rm -rf /opt/ORCLfmap 
    41. [root@Boss ~]# rm -rf /etc/oratab 
    
    **4、删除/usr/local/bin下面Oracle的所有文件**
    
    42. [root@Boss ~]# rm -rf /usr/local/bin/dbhome 
    43. [root@Boss ~]# rm -rf /usr/local/bin/oraenv 
    44. [root@Boss ~]# rm -rf /usr/local/bin/coraenv 
    
    **5．删除/tmp目录下Oracle的相关文件**
    
    45.  [root@Boss ~]# rm -rf /tmp/OraInstall2018-07-* 
    46. [root@Boss ~]# rm -rf /tmp/deinstall2018-07-22_11-* 
    47. [root@Boss ~]# rm -rf /tmp/hsperfdata_oracle/ 
    
    **6、删除Oracle安装目录**
    
    48. [root@Boss ~]# rm -rf /u01/oracle 
    
    **7、 删除Oracle用户及dba、oinstall、oper用户组**
    
    49. [root@Boss ~]# userdel oracle 
    50. [root@Boss ~]# groupdel dba 
    51. [root@Boss ~]# groupdel oper 
    52. [root@Boss ~]# groupdel oinstall 
    
    **8、重启**
    
    53. [root@Boss ~]# reboot
    
    

    
