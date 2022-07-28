---
layout:     post
title:      CentOS安装ORACLE（史上最详细！！！！！！！！！！！！！！！！！！！！没有之一！！！！！！！！）
subtitle:   oracle 
date:       2022-07-28
author:     熊金
header-img: img/post-bg-swift2.jpg
catalog: true
tags:
    - Tylor
---




@[TOC](文章目录)
## 一，安装所需软件
			

 1. centos安装平台，vmware esxi 6.5 （阿里云）
 	

```java
https://www.aliyundrive.com/s/QEB5TMaLJRv
```
**centos也可以安装在vmware虚拟机上**

 2. CentOS-7-x86_64-DVD-1810.iso（阿里云）

```java
https://www.aliyundrive.com/s/SqmtWGCXPGU
```

 3. oracle安装包（阿里云）

```java
https://www.aliyundrive.com/s/QBEw8utfFjo
```

## 二，配置环境（root用户下操作）



2，安装httpd

```java
yum install httpd -y
```

 1. 配置epel源

```java
rpm -vih https://dl.fedoraproject.org/pub/epel/7Server/x86_64/Packages/e/epel-release-7-12.noarch.rpm
```

 2. 关闭防火墙

```java
systemctl stop firewalld.service
systemctl disable firewalld
systemctl stop firewalld
systemctl status firewalld
```

 3. 依赖库安装

```java
yum install -y compat-libstdc++-33 elfutils-libelf-devel gcc gcc-c++ glibc-devel
yum install -y ksh libaio-devel numactl-devel
yum install -y unixODBC unixODBC-devel oracleasm oracleasmlib oracleasm-support
```

 4. 创建Oracle用户组和用户账号

```java
groupadd oinstall
groupadd dba
useradd -g oinstall -G dba oracle
passwd oracle
```

 5. 配置内核参数
		**修改文件/etc/sysctl.conf ,vim /etc/sysctl.conf，加上下面的命令**

```java
fs.file-max = 6815744

kernel.shmall = 2097152

kernel.shmmax = 536870912

kernel.shmmni = 4096

kernel.sem = 250 32000 100 128

net.ipv4.ip_local_port_range = 9000 65500

net.core.rmem_default = 262144

net.core.rmem_max = 4194304

net.core.wmem_default = 262144

net.core.wmem_max = 1048576

net.ipv4.tcp_wmem = 262144 262144 262144

 net.ipv4.tcp_rmem = 4194304 4194304 4194304
```
**执行命令： /sbin/sysctl -p 使参数生效。**

 6. 修改文件/etc/security/limits.conf， vim /etc/security/limits.conf，添加如下命令

```java
oracle              soft    nproc   2047
oracle              hard    nproc   16384
oracle              soft    nofile  1024
oracle              hard    nofile  65536
oracle              soft    stack   10240
oracle              hard   stack    32768
oracle              hard   memlock    134217728
oracle              soft   memlock    134217728
```

 7. 创建Oracle的安装目录，并赋予其可读写的权限

```java
mkdir -p /u01/app/oracle
chown -R oracle:oinstall /u01/app/oracle
chmod -R 777 /u01/app/oracle
chown -R oracle:oinstall /home/oracle
chmod -R 777 /home/oracle
mkdir -p /u01/app/oraInventory
chown -R oracle:oinstall /u01/app/oraInventory
mkdir -p /home/oracle/Downloads
chmod -R 777 /home/oracle/Downloads
```

 8. 编辑Oracle用户环境
**在root用户下输入 su oracle 进入oracle用户环境
编辑.bash_profile文件 vim /home/oracle/.bash_profile**

```java
PATH=$PATH:$HOME/.local/bin:$HOME/bin

export PATH
#For Oracle（下面是需要添加的部分）
export DISPLAY=:0.0
export TMP=/tmp;
export TMPDIR=$TMP;
export ORACLE_BASE=/u01/app/oracle;
export ORACLE_HOME=$ORACLE_BASE/product/11.2.0/dbhome_1;
export ORACLE_SID=ORADB11G;
export ORACLE_TERM=xterm;
export PATH=/usr/sbin:$PATH;
export PATH=$ORACLE_HOME/bin:$PATH;
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib;
```

 9. 查看环境变量是否配置成功(oracle用户)

```java
echo $ORACLE_HOME
```
**如果正常输出了配置的路径，则成功，如果输出为空白，可以在本次手动执行
 source /home/oracle/.bash_profile 执行环境变量的加载。但是仅在当前终端窗口有效。**

10.设置主机hosts映射，root下更改 vi /etc/hosts，更改后的效果应如下所示

```java
127.0.0.1 localhost

127.0.1.1 centos-oracle11
```
11.重启centos，root用户下输入如下命令

```java
init 6
```

 
## 三，安装步骤

 1. 将压缩文件传输到/home/oracle/Downloads
 2. cd /home/oracle/Downloads，解压缩，文件会自动解压到/home/oracle/Downloads/database

```java
unzip linux.x64_11gR2_database_1of2.zip
unzip linux.x64_11gR2_database_2of2.zip
```
**一定要在linux解压，否则安装到7%时报错：
Error in invoking target 'libasmclntsh19.ohso libasmperl19.ohso client_sharedlib' of makefile '/u01/app/rdbms/lib/ins_rdbms.mk'. See '/tmp/InstallActions2021-07-10_07-46-40AM/installActions2021-07-10_07-46-40AM.log' for details.^**


 
 3. root用户下，新建swap分区（否则安装过程报错，具体安装要求大小没研究过，以下分区大小16400000KB）

```java
1、使用dd命令创建一个swap分区

          2、#dd if=/dev/zero of=/home/swap bs=1024 count=16400000
          （要等待很长一段时间，大概二十分钟，不能打断现在正在操作的终端窗口，直到输出：
			16400000+0 records in
			16400000+0 records out
			16793600000 bytes (17 GB) copied, 274.484 s, 61.2 MB/s
          ，创建成功）

          3、格式化刚才创建的分区

          4、# mkswap /home/swap

          5、再使用swapon命令把这个文件分区变成swap分区

          6、#swapon /home/swap

          7、（关闭SWAP分区的命令为：#swapoff /home/swap)

          8、再用free -m 查看已经扩容的了swap分区。

          9、为了能够让swap自动挂载，需要修改etc/fstab文件，用vi /etc/fstab

          10、在文件末尾加上 /home/swap swap swap default 0 0

          11、这样就算重启系统，swap分区也不用手动挂载了

          12、但是我感觉好像我重启了系统 swap就没有了，然后我又百度了一下，要执行下面一段命令
            #echo   "swapon  /home/swap" >> /etc/inittab 

          13、然后在 vi /etc/inittab

          14、最后一行是swapon  /home/swap，这样就万事大吉了。
```

 4.进入centos桌面，退出现在的账号，用oracle账号登录centos， 
 cd /home/oracle/Downloads/database， 执行 ./runInstaller 进入安装界面，此终端窗口不能被打断，需要输入新的命令请打开另一个终端窗口
 5，取消沟-----下一步------yes（提示没有填写邮箱）
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713213035172.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713213147747.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
6，选择“Create and configure a database”，点击“Next”
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021071321330584.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
7，选择“Server Class”，点击“Next”
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713213418433.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
8，选择"single instance database installation"，点击"next"
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021071321362920.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
9，选择"Advanced install"，点击"next"
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713213742858.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
10，默认为"English"，添加"Engglish(United Kindom)"，点击"next"
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713213914325.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
11，选择"Enterprise Edition(4.29G)"，点击"next"
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713214056289.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
12，一般Oracle base  和Oracle home directory 会被使用
Oracle Base: /u01/app/oracle
Software location: /u01/app/oracle/product/11.2.0/dbhome_1
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021071321432382.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
13，
存储Oracle 安装文件的目录 被叫做“Oracle inventory directory”.

Inventory目录: /u01/app/oraInventory

oraInventory用户组名: oinstall
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713214607276.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
14，选择你要创建的数据库类型：
General Purpose / Transaction Processing
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713214740607.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
15，global database name 和 Oracle Service Identifier可以定义为一样的值
Global database name: ORADB11G

Oracle Service Identifier (SID): ORADB11G
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713215006212.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
16，
这一步有四个选项，你可以按照如下所示配置每一个选项
Memory: 勾选 Automatic Memory Management (默认).
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713215621837.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)

Character 设置: 勾选Unicode (AL32UTF8).
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713215634206.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)

Security: 勾选Assert all new security settings (default).
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713215645721.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)

Sample schemas: 勾选Create database with sample schemas (**然而, 你可以不做更改，这里不勾选**).
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713215659178.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
17，管理选项
不做任何操作，下一步
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713221354723.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
18，数据库存储
阅读如图所示的建议并选择File system
Specify database file location: /u01/app/oracle/oradata
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713221626779.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
19，备份和恢复
选择默认选项：Do not enable automated backups (default).
备份是很重要，但是最好是在centos上安装完oracle以后再来做数据备份
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713222001446.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
20，Schema密码
选择：Use the same password for all accounts.输入密码并确认密码
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021071322225463.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
21，操作系统用户组
阅读窗口所示内容并选择用户组
Database Administrator (OSDBA) Group: dba

Database Operator (OSOPER) Group: oinstall
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713222457281.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
22，检查
现在Oracle安装程序检查是否系统配置满足条件，这一步将会出现警告，OS Kernel Parameter semmni 之前配置过，但是现在显示的状态仍然是失败
OS Kernel Parameter: semmni Failed
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713222947585.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
23，用如下命令（root用户）检查semmni parameter，/sbin/sysctl -a | grep sem或者
cat /proc/sys/kernel/sem，如果最后一个值大于等于128，那么你可以忽略这个告警，比如：
kernel.sem = 250        
32000   100     128
or
cat /proc/sys/kernel/sem
250     32000   100     128
**由于某种原因，Oracle 先决条件检查器无法确定 semmni 参数是否正确以及是否安装了所需的包。 如果 Prerequisite Checks 窗口的内容与上面屏幕截图中显示的窗口类似，请勾选 Ignore All 复选框并单击 Next 继续。**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713223815426.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)

24，汇总
检查配置摘要以确保您已根据需要配置了所有设置。 您可以保存 response file(db.rsp)，因为当您需要在多台 Linux 机器上安装 Oracle 或者您打算在没有 GUI 的 centos 服务器上安装 Oracle 时，它可能会派上用场。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713224125357.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021071322421626.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
现在您可以监视 Oracle 安装程序如何复制文件，直到安装程序进入链接二进制文件阶段。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713224333210.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
25,**为 Oracle 链接二进制文件——修复链接和编译错误**
这个阶段是在centos Linux 上安装Oracle 数据库最困难的阶段之一。 这一步出错的概率很高。 在当前例子中，我们在centos 7上安装Oracle 11g时遇到如下错误：
Error in invoking target 'install' of makefile '/u01/app/oracle/product/11.2.0/dbhome_1/ctx/lib/ins_ctx.mk'. See '/u01/app/oraInventory/logs/installActions2021-07-13_09-27-42AM.log' for details.
**请注意，即使您在 Oracle Linux 上安装 Oracle，也会发生类似的错误。 您的日志文件的名称应该不同。**
**注意：带有 .mk 扩展名的文件是 makefile，用于编译程序。 makefile 确定要编译的程序“部分”以及必须编译和链接程序的哪些文件。 Oracle 软件的一些组件是用 Java 编写的（例如，具有 GUI 的 Oracle Universal Installer），一些组件是用 C 编写的。 用 Java 编写的组件必须在不编译的情况下进行解释（Java 是多平台的），而编写在 C 必须被编译并且链接二进制文件和库是这个过程的必需阶段。 Oracle 使用这种方法来增加更多的灵活性，并允许在不同的操作系统（Windows、Linux、Solaris）上安装 Oracle。 另一个好处是减小了安装包的大小。 缺点是在Oracle安装过程中链接二进制阶段可能会出现错误。 在大多数情况下，您应该通过编辑 .mk 文件来添加适当的标志来修复链接问题。**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210713225710362.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
让我们来看看如何修复在 centos Linux 上安装 Oracle 数据库时出现的链接二进制文件错误。
1，通过 SSH 以 oracle 身份登录到 centos 并检查日志文件：
tail -n 100 /u01/app/oraInventory/logs/installActions2021-07-13_09-27-42AM.log

```java
INFO: make: *** [ctxhx] Error 1
Exception String: Error in invoking target 'install' of makefile '/u01/app/oracle/product/dbhome_1/ctx/lib/ins_ctx.mk'. See '/u01/app/oracle/oraInventory/logs/installActions2014-01-11_01-24-04PM.log' for details.
Exception Severity: 1
```
解决方法：
让我们打开 ins_ctx.mk 文件并检查该文件的第 11 行。
vim /u01/app/oracle/product/11.2.0/dbhome_1/ctx/lib/ins_ctx.mk
在vim的导航模式下输入:set number查看行号。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021071422581932.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
让我们找到第 11 行。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210714225842600.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
在 vim 中使用导航模式时，可以使用以下命令找到所需的字符串：
/LINK_CTXHX
编辑以下部分：
ctxhx: $(CTXHXOBJ)

$(LINK_CTXHX) $(CTXHXOBJ) $(INSO_LINK)
必须按如下方式编辑第 11 行：
-static $(LINK_CTXHX) $(CTXHXOBJ) $(INSO_LINK) /usr/lib64/stdc.a
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210714230017585.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
保存 ins_ctx.mk 并退出 vim。
现在返回到 Oracle 安装错误窗口并单击重试。
运行脚本 并点击retry

2，安装过程继续，但几秒钟后，当构建代理库时显示另一个错误：
Error in invoking target 'agent nmhs' of makefile '/u01/app/oracle/product/11.2.0/dbhome_1/sysman/lib/ins_emagent.mk'.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210714230140689.png)

以oracle用户运行以下命令：
1，**export ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1**
2，sed -i 's/^\(\s*\$(MK_EMAGENT_NMECTL)\)\s*$/\1 -lnnz11/g' $ORACLE_HOME/sysman/lib/ins_emagent.mk
**注意：sed 是一个流文本编辑器，与使用 vim 或其他交互式文本编辑器编辑它们相比，它用于节省编辑 .mk 文件的时间。**
运行以上两条命令后，在Oracle安装错误窗口中点击Retry。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021071423025789.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210714231031500.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)


3，执行配置脚本
我们快要在 centos上安装 Oracle 数据库了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210714231240699.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)

以 root 身份运行这两个脚本：
**/u01/app/oraInventory/orainstRoot.sh
/u01/app/oracle/product/11.2.0/dbhome_1/root.sh**
在下面的屏幕截图中，您可以看到脚本已成功执行。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210714231558598.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
26，配置监听器
oracle安装成功后需要配置监听器，直接键入 netca命令，如果提示bash:netca:command not found ，手工执行source /home/oracle/.bash_profile，就可以进入配置监听器的用户界面了，根据需要配置监听器。
首先配置tnsnames.name
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210715072516222.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)

```java
ORADB11G =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 你的主机ip)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = ORADB11G)
    )
  )
```
配置listener.ora
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210715072637236.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)

```java
SID_LIST_LISTENER =
(SID_LIST =
  (SID_DESC =
  (GLOBAL_DBNAME = ORADB11G)
  (ORACLE_HOME=/u01/app/oracle/product/11.2.0/dbhome_1)
  (SID_NAME = ORADB11G)
  )
)


LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 你的主机ip)(PORT = 1521))
    )
  )

ADR_BASE_LISTENER = /u01/app/oracle
```



27，检查 Oracle 是否在 centos上运行
在运行 Oracle 的 Ubuntu 机器上打开 Web 浏览器，并通过打开 Oracle Database Control Manager 的 Web 界面检查 Oracle 是否正在运行。 以下链接可用于执行此操作：
https://localhost:1158/em

https://centos-oracle11:1158/em

https://centos的ip/em
您还可以尝试从网络的另一台主机连接到运行 Oracle 的 centos机器。

定义连接参数。
User Name: SYS

Password: 你之前在安装oracle的过程中安装的密码

Connect As: SYSDBA
您可以在下方看到 Oracle 企业管理器在 Oracle 正常运行时的屏幕截图。
28，Oracle post-installation configuration
现在安装了 Oracle，现在我们应该将数据库配置为在 centos启动后自动启动。 您应该编辑 /etc/oratab 配置文件以将 Oracle 设置为在系统引导期间启动。
以root用户运行以下命令
vim /etc/oratab
Replace N to Y at the end of the line:
ORADB11G:/u01/app/oracle/product/11.2.0/dbhome_1:Y
**有用的命令**
在设置 Oracle 自动启动之前，您应该了解如何手动启动 Oracle 组件。 这也将有助于诊断。 以 oracle 用户身份运行这些命令。
运行数据库：
dbstart $ORACLE_HOME
/etc/init.d/oracle start


停止数据库：
dbshut $ORACLE_HOME
启动为数据库控制提供 Web 界面的 Database Control Enterprise Manager：
emctl start dbconsole
停止Database Control Manager:
emctl stop dbconsole
检查Database Control Manager状态：
emctl status dbconsole
开始监听：
lsnrctl start
停止监听：
lsnrctl stop
检查监听状态：
$ORACLE_HOME/bin/lsnrctl status
启动数据库配置助手（在 GUI shell 中，而不是在 SSH 控制台中）：
dbca
可以通过编辑文件来配置 Oracle 监听：
vim $ORACLE_HOME/network/admin/listener.ora
在控制台连接数据库：
sqlplus / as sysdba;
**在centos中创建Oracle的启动脚本**
下面我们来探讨一下如何设置Oracle在centos启动时自动启动。 编辑/etc/oratab 后，您应该在/etc/init.d/ 目录中创建一个启动脚本。
在centos中新建Oracle启动脚本文件（以root身份运行命令）：
**vim /etc/init.d/oracle**
增加以下内容到文件里

```java
#!/bin/bash

#

# Run-level Startup script for the Oracle Instance and Listener

#

### BEGIN INIT INFO

# Provides:          Oracle

# Required-Start:    $remote_fs $syslog

# Required-Stop:     $remote_fs $syslog

# Default-Start:     2 3 4 5

# Default-Stop:      0 1 6

# Short-Description: Startup/Shutdown Oracle listener and instance

### END INIT INFO

#ORACLE_UNQNAME="ORADB11G"

#export $ORACLE_UNQNAME

echo "ORACLE_UNQNAME is $ORACLE_UNQNAME"

ORACLE_HOME="/u01/app/oracle/product/11.2.0/dbhome_1"

ORACLE_OWNR="oracle"

# if the executables do not exist -- display error

if [ ! -f $ORACLE_HOME/bin/dbstart -o ! -d $ORACLE_HOME ]

then

echo "Oracle startup: cannot start"

exit 1

fi

# depending on parameter -- startup, shutdown, restart

# of the instance and listener or usage display

case "$1" in

start)

# Oracle listener and instance startup

echo -n "Starting Oracle: "

echo "dbstart"

source "/home/oracle/.bashrc" && su $ORACLE_OWNR -c "$ORACLE_HOME/bin/dbstart $ORACLE_HOME"

echo "lsnrctl start"

source "/home/oracle/.bashrc" && su $ORACLE_OWNR -c "$ORACLE_HOME/bin/lsnrctl start"

#Optional : for Enterprise Manager software only

echo "emctl start dbconsole"

source "/home/oracle/.bashrc" && su $ORACLE_OWNR -c "$ORACLE_HOME/bin/emctl start dbconsole"

touch /var/lock/oracle

echo "OK - a script has been executed"

;;

stop)

# Oracle listener and instance shutdown

echo -n "Shutdown Oracle: "

#Optional : for Enterprise Manager software only

source "/home/oracle/.bashrc" && su $ORACLE_OWNR -c "$ORAClE_HOME/bin/emctl stop dbconsole"

source "/home/oracle/.bashrc" && su $ORACLE_OWNR -c "$ORACLE_HOME/bin/lsnrctl stop"

source "/home/oracle/.bashrc" && su $ORACLE_OWNR -c "$ORACLE_HOME/bin/dbshut $ORACLE_HOME"

rm -f /var/lock/oracle

echo "OK - a script has been executed"

;;

reload|restart)

$0 stop

$0 start

;;

*)

echo "Usage: $0 start|stop|restart|reload"

exit 1

esac

exit 0
```
设置正确的权限：
chown oracle:oinstall /etc/init.d/oracle

chmod 0775 /etc/init.d/oracle
在操作系统启动后立即启动此脚本（可用于默认运行级别）：
update-rc.d oracle defaults
如果需要，您可以编辑启动优先级。
运行脚本以停止 Oracle（您可以以 root 身份运行此脚本）：
/etc/init.d/oracle stop
如果要启动 Oracle，请使用以下命令运行此脚本：
**/etc/init.d/oracle start**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210716233337299.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0Mzg1OTY0,size_16,color_FFFFFF,t_70)
**注意：注意 Oracle 启动脚本中的以下行和类似行：**
source "/home/oracle/.bashrc" && su $ORACLE_OWNR -c "$ORACLE_HOME/bin/dbstart $ORACLE_HOME"
首先，我们设置 shell 以读取存储在 oracle 用户的 .bashrc 系统配置文件中的设置，包括 Oracle 组件正常运行所需的变量，例如 ORACLE_HOSTNAME、ORACLE_BASE、PATH 等。 将oracle用户的bash设置应用到当前shell会话后，执行下一条命令启动Oracle数据库。
su - 和 su 和有什么不一样？
user1@hostname:~$ su - username2 – 此命令以所选用户 (username2) 的身份运行命令行 shell 会话，并使用所选用户 (username2) 的设置，就像您在直接创建新 shell 会话时以 username2 登录一样 （从头开始）。 此 shell 会话中使用 username2 的环境变量。
user1@hostname:~$ su username2 – 此命令以选定用户（username2）的身份运行命令行shell，当前用户（user1）的设置和user1的环境变量在此shell会话中由username2继承。
su -c (--command) 表示必须以所选用户身份运行指定的命令。
NAKIVO 备份和复制的数据保护
NAKIVO Backup & Replication 通过多种备份、复制和恢复功能，包括 VMware 备份、Hyper-V 备份、Office 365 备份等，为中小型企业和企业提供高端数据保护。
