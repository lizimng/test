# test删除classroom  server  desktop 虚拟机器

######################################################
搭建教学环境

1. 创建两台虚拟机
[root@room9pc01 ~]# clone-vm7
Enter VM number: 8

[root@room9pc01 ~]# clone-vm7
Enter VM number: 9

2.将两台虚拟机，修改名字，一个取名A  一个取名B

3.利用root用户，密码：123456   登陆两台虚拟系统

4. 设置虚拟机A
       为eth0IP地址：192.168.4.7/24
       永久的主机名：svr7.tedu.cn

   设置虚拟机B
       为eth0IP地址：192.168.4.207/24
       永久的主机名：pc207.tedu.cn

5.默认情况，教学环境。真机可以直接与clone-vm7创建的虚拟机通信，虚拟机eth0的ip地址必须为192.168.4.0/24网络中的ip地址

6.进行真机远程管理两台虚拟机

7.Yum仓库的设置

   服务端：真机搭建Web服务，共享光盘所有内容

   1.安装httpd软件
  # rpm -q httpd

   2.启动httpd服务,设置开机起动
  # systemctl status httpd       #查看当前的状态
  # systemctl is-enabled httpd   #查看是否是开机自起动
  # firefox 127.0.0.1            #自己访问自己

  3.挂载访问测试
  # ls /var/www/html/rhel7

  # mount /iso/rhel-server-7.4-x86_64-dvd.iso /var/www/html/rhel7/

  # ls /var/www/html/rhel7

  # firefox 127.0.0.1/rhel7

   客户端： 虚拟机A   虚拟机B
[root@svr7 ~]# vim /etc/yum.repos.d/rhel7.repo

	[rhel7]
	name=rhel7
	baseurl=http://192.168.4.254/rhel7
	enabled=1
	gpgcheck=0


真机上完成开机自动挂载，   光盘文件默认的文件系统为iso9660
   vim   /etc/fstab

 /var/lib/libvirt/images/iso/rhel-server-7.4-x86_64-dvd.iso   /var/www/html/rhel7   iso9660   defaults   0  0

验证：
1.将所有的loop设备全部卸载
[root@room9pc01 ~]# df -h
/dev/loop0      3.8G  3.8G     0  100% /var/ftp/rhel7
/dev/loop1      3.8G  3.8G     0  100% /var/www/html/rhel7

[root@room9pc01 ~]# umount /var/ftp/rhel7
[root@room9pc01 ~]# umount /var/www/html/rhel7
[root@room9pc01 ~]# df -h     #查看是否卸载成功
[root@room9pc01 ~]# mount -a
[root@room9pc01 ~]# df -h     #查看是否挂载成功

######################################################
 自定义Yum仓库

1.具备非光盘的软件包
   将真机的数据传递给虚拟机A

   # scp /路径/源文件    root@IP地址 :/路径/

真机上操作：
   # scp /root/桌面/tools.tar.gz   root@192.168.4.7:/root/
   root@192.168.4.7's password: 

2.验证在虚拟机A上,查看/root目录下是否有tools.tar.gz

3.虚拟机A上，tar解包到根目录下
[root@svr7 ~]# tar -xf  /root/tools.tar.gz -C   /
[root@svr7 ~]# ls /
[root@svr7 ~]# ls /tools/
[root@svr7 ~]# ls /tools/other/

4.建立仓库数据文件

[root@svr7 ~]# createrepo /tools/other/
[root@svr7 ~]# ls /tools/other/repodata/

5.虚拟机A  /tools/other 是一个仓库
[root@svr7 ~]# vim /etc/yum.repos.d/rhel7.repo 
  [rhel7]
  name=rhel7
  baseurl=http://192.168.4.254/rhel7
  enabled=1
  gpgcheck=0

  [other]
  name=hahaxixi
  baseurl=file:///tools/other     #指定本机为Yum仓库
  enabled=1
  gpgcheck=0
[root@svr7 ~]#  yum repolist

6. 使用Yum仓库
    
    小火车   
  [root@svr7 ~]# yum -y install sl

    星球大战背景    
  [root@svr7 ~]# yum -y install cmatrix

    猫
  [root@svr7 ~]# yum -y install oneko
  [root@svr7 ~]# killall oneko       #杀死所有的oneko程序
  [root@svr7 ~]# oneko &             #放入后台运行

###################################################
目录结构
• 认识Linux的目录层次:
– man hier

常见一级目录的用途
主要用途
	/boot        存放系统引导必需的文件,包括内核、启动配置
	/bin、/sbin  存放各种命令程序
	/dev         存放硬盘、键盘、鼠标、光驱等各种设备文件
	/etc         存放Linux系统及各种程序的配置文件
	/root、/home/用户名  分别是管理员root、普通用户的默认家目录
	/var         存放日志文件、邮箱目录等经常变化的文件
	/proc        存放内存中的映射数据,不占用磁盘
	/tmp         存放系统运行过程中使用的一些临时文件
######################################################
权限的数值表示
• 权限的数值化
– 基本权限:r = 4,w = 2,x = 1

[root@svr7 ~]# mkdir /nsd01
[root@svr7 ~]# ls -ld /nsd01

[root@svr7 ~]# chmod 777 /nsd01
[root@svr7 ~]# ls -ld /nsd01

[root@svr7 ~]# chmod 007 /nsd01
[root@svr7 ~]# ls -ld /nsd01

[root@svr7 ~]# chmod 700 /nsd01
[root@svr7 ~]# ls -ld /nsd01

[root@svr7 ~]# chmod 755 /nsd01
[root@svr7 ~]# ls -ld /nsd01

– 附加权限:SUID = 4,SGID = 2,Sticky Bit = 1

################################################        
历史命令
• 管理/调用曾经执行过的命令
– history:查看历史命令列表
– history -c:清空历史命令
– !str:执行最近一次以str开头的历史命令

• 调整历史命令的数量
[root@svr7 ~]# vim   /etc/profile
 HISTSIZE=1000     //默认记录1000条

[root@svr7 ~]# history      #查看历史命令
  
[root@svr7 ~]# history -c   #清空历史命令

[root@svr7 ~]# cat /etc/redhat-release
[root@svr7 ~]# ls -l /etc/redhat-release
[root@svr7 ~]# history 

[root@svr7 ~]# !ls     #执行最近一条历史命令以ls开头的

[root@svr7 ~]# !cat

#######################################################
统计目录的大小

[root@svr7 ~]# du -sh /boot/

[root@svr7 ~]# du -sh /root/

[root@svr7 ~]# du -sh /etc

[root@svr7 ~]# du -sh /

######################################################
 zip归档工具,跨平台的压缩工具
  命令格式：  zip  -r  /路径/压缩包名    /路径/被压缩的源文件

#  zip -r /root/test.zip   /home/  /mnt/  /etc/passwd
#  ls /root

  命令格式： unzip  /路径/压缩包名   -d  /释放的路径

#  unzip /root/test.zip   -d /opt
#  ls /opt

#####################################################
软连接与硬连接，快捷方式
  

 制作快捷方式：  ln  -s   /路径/源文件     /路径/快捷方式的名字

[root@svr7 /]# ln -s  /etc/redhat-release       /abc
[root@svr7 /]# ls -l  /abc
[root@svr7 /]# cat  /abc


  i节点: 编号标识硬盘的一块存储区域


[root@svr7 /]# echo 123 > /opt/A
[root@svr7 /]# cat /opt/A
123
[root@svr7 /]# ln -s /opt/A /opt/B
[root@svr7 /]# ls /opt/



ln,创建软连接
– 软连接 --> 原始文档 --> i节点 --> 文档数据
– ln -s  原始文件或目录   软连接文件
  若原始文件或目录被删除,连接文件将失效
  软连接可存放在不同分区/文件系统

• ln,创建硬连接
– 硬连接 --> i节点 --> 文档数据
– ln  原始文件   硬连接文件
  若原始文件被删除,连接文件仍可用
  硬连接与原始文件必须在同一分区/文件系统

##############################################
[root@svr7 /]#  echo 123 > /opt/A

[root@svr7 /]#  cat  /opt/A

[root@svr7 /]#  ln -s /opt/A   /opt/B   #软连接

[root@svr7 /]#  ln  /opt/A   /opt/C     #硬连接

[root@svr7 /]#  ls -i  /opt/

[root@svr7 /]#  rm -rf /opt/A   #当删除源文件，只有硬连接可用

[root@svr7 /]#  ls -i  /opt/

###################################################
 wc 统计命令 

  wc -l : 只统计行数

[root@svr7 /]# wc  -l  /etc/passwd  

  请统计/etc/以 .conf结尾的文件有多少个（包含子目录）
[root@svr7 /]# find /etc/ -name  "*.conf"  |  wc -l

[root@svr7 /]# find /etc/ -name  "*tab"  |  wc -l

[root@svr7 /]# find /boot/ -name "vm*"
[root@svr7 /]# find /boot/ -name "vm*" | wc -l

###################################################
vim文本编辑器

光标跳转
	Home 键 或 ^、数字 0           跳转到行首
	End 键 或“$”键                     跳转到行尾
	1G 或 gg                跳转到文件的首行
	G                      跳转到文件的末尾行
    12G                    跳转到文件的第12行

[root@svr7 /]# head -20 /etc/passwd > /opt/test.txt
[root@svr7 /]# vim /opt/test.txt 

复制/粘贴/删除

   复制 yy、#yy 	 复制光标处的一行、#行
   粘贴 p 		     粘贴到光标处之后
  x 或 Delete键 	 删除光标处的单个字符
  dd、#dd		 删除光标处的一行、#行
  d^ 			 从光标处之前删除至行首
  d$ 			 从光标处删除到行尾
  C(大写)         从光标处删除到行尾，并且进入插入模式
  u               撤销上一次操作
 
[root@svr7 /]# vim /opt/test.txt 


查找/撤销/保存

	/word 	 向后查找字符串“word”
	n、N 	 跳至后/前一个结果
	u 		 撤销最近的一次操作
	Ctrl + r  取消前一次撤销操作(反撤销)
	ZZ(大写)   保存修改并退出

末行模式操作

  :r /etc/filesystems 读入其他文件内容

[root@svr7 /]# echo 123 > /opt/1.txt
[root@svr7 /]# echo abc > /opt/2.txt
[root@svr7 /]# vim /opt/1.txt
             末行模式下     : r    /opt/2.txt
 				 :r    /etc/passwd


字符串替换

	:s/root/admin        替换光标所在行第一个“root”
	:s/root/admin/g      替换光标所在行所有的“root”
	:1,10s/root/admin/g  替换第1-10行所有的“root”
	:%s/root/admin/g     替换文件内所有的“root

[root@svr7 /]# vim /opt/test.txt


 开关参数的控制

	:set  nu|nonu   显示/不显示行号
	:set  ai|noai   启用/关闭自动缩进

######################################################
 源码编译安装

 rpm包：rpm  yum   简单  不灵活

          利用gcc与make
  源码包---------------------->可以执行的程序 ---------->运行安装    

     灵活，功能与安装位置 可以自由选择

源码包编译安装：

步骤1：安装gcc与make编译开发工具
[root@svr7 /]# yum -y install gcc make

步骤2:tar解包,释放源代码至指定目录
 # tar -xf /tools/inotify-tools-3.13.tar.gz -C /mnt/
 # ls /mnt/
 # cd /mnt/inotify-tools-3.13/

步骤3:  ./configure 配置,指定安装目录/功能模块等选项
                             作用1：检测系统是否安装gcc
                             作用2：功能与安装位置 可以自由选择 

            --prefix= 指定安装位置

 # cd /mnt/inotify-tools-3.13/
 # ./configure  --prefix=/opt/myrpm


常见报错：没有安装gcc
checking for gcc... no
checking for cc... no
checking for cl.exe... no

步骤4:make 编译,生成可执行的二进制程序文件
步骤5:make install 安装,将编译好的文件复制到安装目录


验证：
[root@svr7 inotify-tools-3.13]# ls /opt/
[root@svr7 inotify-tools-3.13]# ls /opt/myrpm/
[root@svr7 inotify-tools-3.13]# ls /opt/myrpm/bin/


####################################################
systemctl控制

服务的控制

  开启服务  systemctl  start  服务名
  停止服务  systemctl  stop   服务名
  重起服务  systemctl  restart  服务名

 
 设置开机自起      systemctl  enable   服务名
 设置开机不自起   systemctl  disable  服务名
 查看当前是不是开机自起   systemctl  is-enabled  服务名


模式的控制
    
    纯文本字符模式 ： multi-user.target 
    图形模式  ： graphical.target

当前切换模式  
 # systemctl isolate  graphical.target   #切换到图形模式
 # systemctl isolate  multi-user.target   #切换到字符模式


修改默认进入的模式
 # systemctl get-default   #查看默认进入的模式

 # systemctl set-default graphical.target  #修改默认的模式

 # systemctl get-default 

 # reboot  #重起验证
#######################################################





































