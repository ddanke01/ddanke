# linux学习笔记

## 一、VMware如何安装

1. 第一步，打开博通个人页进行登录或者注册，必须得注册博通账号才可以使用
2. 第二步，直接复制链接转跳到博通可以直接下载：https://support.broadcom.com/group/ecx/productfiles?subFamily=VMware%20Workstation%20Pro&displayGroup=VMware%20Workstation%20Pro%2025H2%20for%20Windows&release=25H2&os=&servicePk=&language=EN&freeDownloads=true
3. 也可以直接登录账号下载，选择VMware Workstation Pro
4. 选择25H2版本下载即可!
5. 下载后直接安装即可使用

## 二、如何在在VMware里面安装Linux

1. 浏览器搜索Cent OS，进入Cent OS官网并点击CentOS Stream

2. 选择x86_64 ISOs版本，下载

3. 打开 VMware → Create a New Virtual Machine

4. 选择刚才下载的镜像文件

5. 硬件配置

   | 项目     | 建议                |
   | -------- | ------------------- |
   | CPU      | 2 核                |
   | 内存     | **4 GB（最低 2G）** |
   | 硬盘     | **40–60 GB**        |
   | 硬盘类型 | **SCSI（默认）**    |
   | 网络     | NAT（默认）         |

6. 系统配置选择

   系统语言：选择英文/中文

   输入法：选择英文/中文，默认英文

   账户：管理员root，普通用户ddanke，设置好密码

   磁盘：选择刚才分配的磁盘即可

   系统版本二选一

   ​	GUI：UI界面的版本，比较省心

   ​	Minimal：最简洁版本，什么都没有，适合新手练手和服务器

   练手的话其他可以不用安装

7. 安装完成后进入系统登录用户即可，输入用户名称和密码，密码输入的时候不会显示，直接回车即可进入系统

   （此处指令可参考系统用户指令笔记）

## 三、最基础的Linux应该配置哪些东西（Linux学习路线）

### 第 1 阶段

- Minimal Install

- 只用命令行

  学习基础的命令并整理笔记

  https://www.runoob.com/linux/linux-tutorial.html

- 手动装工具

  安装基础工具并整理笔记

  https://www.oryoy.com/news/centos-zui-xiao-hua-an-zhuang-hou-bi-zhuang-ruan-jian-bao-qing-dan-bian-cheng-huan-jing-yu-chang-yon.html

------

### 第 2 阶段

- 学服务管理（systemd）
- 学网络 / 防火墙 / SELinux
- 学用户、权限、日志

------

### 第 3 阶段

- Docker / Podman
- Nginx / MySQL / Redis
- 自动化脚本（Shell）

## 四、Linux基础命令

### 配置语言环境

```
locale			#查看当前locale状态
大概概率看到如下
LANG=
LC_ALL=

安装语言包
sudo dnf install -y glibc-langpack-en glibc-langpack-zh

生产并设置英文环境
sudo localectl set-locale LANG=en_US.UTF-8

退出登录并重新登录
exit

验证
locale
大概率看到
LANG=en_US.UTF-8
```



### Minimal Linux安装ssh服务

```
查看网卡状态和ip
ip a
找到类似
inet 192.168.xxx.xxx

测试外网
ping -c 3 baidu.com

安装ssh服务
sudo dnf install -y openssh-server

验证是否安装成功
rpm -qa | grep openssh

设置为开机自启动
sudo systemctl start sshd
sudo systemctl enable sshd

验证开机自启动状态
systemctl status sshd
看到如下状态就对了
active (running)

防火墙放心ssh
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload

验证是否放行
sudo firewall-cmd --list-services

确认ssh端口
ss -lntp | grep ssh
会看到0.0.0.0:22

剩下的就可以用xshell连接了
如果连接失败排查如下问题
sshd 没启动
防火墙没放行
IP 写错
```

### Minimal Linux安装tree服务

```
yum install -y tree
或者
dnf install -y tree

安装完成后tree -L 1/

如果是普通用户
sudo dnf install -y tree
```

### Minimal Linux安装Vim服务

```
vim --version				#检查系统有无vim
dnf install -y vim			#安装vim
vim --version				#安装完成后检查是否安装
```

### Minimal Linux 安装排错工具

```
dnf install -y \
iproute \
net-tools \
curl \
wget \
bind-utils

iproute
提供ip命令
有没有 IP
网卡是不是 up
默认网关是谁
路由是否正确

net-tools
看端口监听（老方式）
兼容老教程 / 老脚本
临时排查

curl
服务到底有没有响应？
是 nginx 问题，还是程序问题？
接口返回什么内容？

wget
下载程序包
拉测试文件
拉配置模板

已安装
dnf install -y \
iproute \
net-tools \
curl \
wget \
bind-utils
```



### 关机命令

```
poweroff
shutdown -h now   	#立刻关机
shutdown -h +10  	#10分钟后关机
shutdown -h 20:35	#系统会在今天20:35关机
shutdown -c       	# 取消关
```

### 重启命令

```
reboot
shutdown -r now
shutdown -r +10		#系统会在10钟后重启
```

### 注销/退出登录命令

```
exit            #退出当前用户，每次su-就得exit一次
logout          #直接退出登录
login			#登录
whoami			#查看当前用户
```

### 用户切换命令

```
不完整切换❌不加载环境，容易路径、变量异常
su root			#切换root用户
su ddanke		#切换ddanke用户

完整切换，完整加载该用户环境，相当于“重新登录一次”
su - root		#切换root用户
su - 			#直接切换到管理员用户
su - ddanke		#切换ddanke用户

sudo的方式（生产环境首选），不切换用户临时执行一条命令
sudo 命令			#临时用root执行一条命令
sudo -i			#切换到root用户，并且输入当前用户密码，确认当前用户身份
sudo -u 用户名 命令	#以某某用户临时执行一条什么命令
sudo -u ddanke whoami
```

### linux系统不同目录含义

```
/                           # 根目录，一切的起点
├── bin                     # 基本命令目录（ls、cp、mv 等，系统启动必需）
├── boot                    # 启动相关文件（内核、引导程序，慎动）
├── dev                     # 设备文件目录（磁盘、终端、null 等，一切皆文件）
├── etc                     # 系统和服务的配置文件中心（最重要）
│   ├── ssh                 # SSH 配置目录
│   │   ├── sshd_config     # SSH 服务端配置文件
│   │   └── ssh_config      # SSH 客户端配置文件
│   ├── passwd              # 用户账户信息
│   ├── shadow              # 用户密码（加密）
│   ├── group               # 用户组信息
│   └── fstab               # 文件系统挂载配置
├── home                    # 普通用户家目录
│   └── user                # 示例普通用户目录（/home/user）
├── lib                     # 32 位系统库文件（命令运行依赖）
├── lib64                   # 64 位系统库文件
├── media                   # 自动挂载目录（U 盘、光盘）
├── mnt                     # 临时手动挂载点（管理员使用）
├── opt                     # 第三方独立软件安装目录
├── proc                    # 内核信息（虚拟文件系统，不是真实文件）
│   ├── cpuinfo             # CPU 信息
│   └── meminfo             # 内存信息
├── root                    # root 用户的家目录（权限极高）
│   └── anaconda-ks.cfg     # 系统安装时生成的配置记录文件
├── run                     # 系统运行时数据（PID、socket，重启清空）
├── sbin                    # 系统管理命令（多需 root 权限）
├── srv                     # 服务数据目录（Web/FTP 数据，较少使用）
├── sys                     # 内核和硬件信息接口（比 /proc 更底层）
├── tmp                     # 临时文件目录（所有用户可写，可能被清空）
├── usr                     # 用户程序和应用主体（非常重要）
│   ├── bin                 # 用户级命令（大多数命令在这里）
│   ├── sbin                # 管理命令
│   ├── lib                 # 程序依赖库
│   └── local               # 本地安装的软件（/usr/local）
│       ├── bin             # 本地安装的可执行文件
│       ├── sbin            # 本地管理命令
│       └── lib             # 本地库文件
└── var                     # 经常变化的数据
    ├── log                 # 日志文件目录（排错核心）
    │   ├── messages        # 系统日志
    │   ├── secure          # 安全/认证日志
    │   └── dnf.log         # 包管理日志
    ├── lib                 # 程序运行数据
    ├── tmp                 # 程序临时文件
    └── spool               # 队列数据（cron、mail 等）
```



### ls目录查看命令

```
ls：列目录
-l：看权限
-a：看隐藏
-h：看大小
-lah：最常用组合

----
ls				#列出 当前目录、不显示隐藏文件、简略显示（只显示名字）
ls /etc
ls /var/log 	#查看指定目录
ls /etc /home	#查看多个目录

---
ls -l			#列出详细列表
会展示如下
-rw-------  1 root root  1234 anaconda-ks.cfg
含义如下
-rw-------		#文件类型 + 权限
1				#硬链接数
root			#所有者
root			#所属组
1234			#文件大小（字节）
anaconda-ks.cfg	#文件名

---
ls -a
会显示如下
.  ..  .bashrc  .ssh
含义如下
.				#当前目录
..				#上级目录
.				#开头的文件是隐藏文件

---
ls -lt			#按修改时间顺序排序，最新的在最上边
ls -ltr			#修改时间顺序反向排序
ls -lhs			#按照大小排序

---
ls -d */		#只看目录


---
ls -ld /etc		#只看目录本身信息
```

### tree目录查看命令

```
tree					#查看当前目录下所有
tree [目录名]			#查看指定目录

tree -L 1 [目录名]		#限制指定层级
tree -L 2 [目录名]

tree -d [目录名]		#只看目录
	
tree -a [目录名]		#显示隐藏文件

tree -f [目录名]		#显示完整路径

tree -p [目录名]		#显示权限
```



### cd目录切换命令

```
符号   | 含义      |

.			#当前目录
..			#上一级目录
~			#当前用户家目录
-			#上一次所在目录

----
pwd				#查看当前所在位置

---
切换到家目录
cd
cd ~
普通用户 → /home/用户名
root 用户 → /root

---
切换到指定目录（绝对路径）
cd /etc
cd /var/log

切换到某个目录（相对路径）
cd ../log

---
返回上级目录
cd ..

---
切换到当前目录
cd .				#一般用于脚本或占位

---
快速返回上一个目录
cd -
```

### mkdir创建目录命令

```
创建目录
mkdir 文件名			#创建目录，只能创建 不存在的目录、默认继承父目录权限策略
mkdir test	

---
创建多个目录
mkdir dir1 dir2 dir3

---
创建多级目录
mkdir -p a/b/c		#创建多级目录
mkdir -p /data/app/logs
普通用户没有权限在根目录下创建目录，需要sudo提权
sudo mkdir -p /test1/test2
---
创建多级目录并显示创建过程
mkdir -pv a/b/c

---
创建时指定权限
mkdir -m 755 test1
相当于
mkdir mydir
chmod 755 mydir
```

###  rmdir删除目录命令

```
删除
rmdir 目录名			#删除目录，只能删除空目录
rmdir test

---
删除多级目录
rmdir -p a/b/c

---
一般很少用rmdir指令，大部分目录下方都有文件一般都用rm -r命令
rm -r 目录名
rm -r test
```

### touch文件操作命令

```
创建一个空文件
如果文件已存在：更新时间戳
touch 文件名
touch a.txt

----
创建多个空文件
touch a.txt b.txt c.txt

----
实际使用
touch /var/log/test.log
```

### cp文件/目录复制命令

```
参数	含义
-r	递归复制目录
-v	显示过程
-i	覆盖前询问
-a	保留权限/时间（归档）
常用
cp -av src/ dst/

---
复制文件
cp 源 目标
cp a.txt b.txt

---
复制到目录
cp a.txt /tmp/

---
复制目录（必须加 -r）
cp -r dir1 dir2
```

### tar打包压缩解压命令

```
打包：把很多文件 → 一个文件（tar）
压缩：让文件体积变小（gzip / bzip2 / xz）

-c	创建压缩包
-x	解压
-v	显示过程
-f	指定文件名（**必须最后跟文件名**）
-C	指定解压目录
-z	gzip（.gz）
-j	bzip2（.bz2）
-J	xz（.xz）


打包
tar -cvf [压缩后的文件名].tar [文件名]			#打包
tar -zcvf [压缩后的文件名].tar.gz [文件名]		#打包并压缩

解压
tar -zxvf app.tar.gz							#解压到当前目录
加压到指定目录										
tar -zxvf app.tar.gz -C /usr/local/				#解压到指定目录
```

### mv文件/目录移动命令

```
常用参数
-i	覆盖前询问
-v	显示过程

移动文件或目录，同目录下 = 重命名
mv 源 目标
mv a.txt b.txt			#重命名文件
mv a.txt /tpm/			#移动文件

---
移动目录
mv dir1 /opt/
```

### rm删除命令

```
删除文件
rm 文件
rm a.txt

---
删除目录（必须 -r）
rm -r dir

----
强制删除（⚠️ 慎用）
rm -rf dir
```

### 用户管理

#### 什么是shell

```
Shell 是“你和 Linux 内核之间的翻译官”，把我下发的指令翻译给内核

最常用的两个shell

人用账号
/bin/bash
Linux 事实标准
几乎所有服务器默认
教程、脚本最多
稳定、可靠
----
程序用账号
/sbin/nologin
登录直接拒绝
给 程序用户 用
```

#### 什么是组

```
组 = 权限批量管理单位
dev 组 → 10 个开发
ops 组 → 5 个运维
权限赋予到组上，组绑定用户，改一次权限等于批量修改了用户操作用户的权限

为什么不能只用“用户”？
如果只用用户，会变成：
	每加一个人
	就要改一堆文件权限
	完全不可维护
```

#### 什么是权限

```
1.权限由如下四部分组成
谁在操作？
👉 用户（User）

2.他是不是某个群体的一员？
👉 组（Group）
组 = 权限批量管理单位

3.这个资源允许做什么？
👉 文件权限（rwx）
为什么权限“绑在文件上”，而不是用户上？
因为资源才是被保护的对象。

“资源声明自己允许什么，
用户只负责证明自己是谁”

4.有没有被系统“临时授权”？
👉 sudo
在“不改变用户身份”的前提下，
临时借用 root 权限
```

#### 用户查看

```
root	0	超级管理员
系统用户	1–999	系统 / 服务使用
普通用户	≥1000	人登录使用

---
用户信息存储位置
/etc/passwd		用户基本信息
/etc/shadow		用户密码（加密）
/etc/group		用户组信息

---
当前用户是谁？
whoami

---
查看当前用户的 ID 和组
id
id admin
id ddanke

---
查看系统中有哪些用户
cat /etc/passwd

返回信息含义
root:x:0:0:Super User:/root:/bin/bash bin:x:1:1:bin:/bin:/usr/sbin/nologin daemon:x:2:2:daemon:/sbin:/usr/sbin/nologin adm:x:3:4:adm:/var/adm:/usr/sbin/nologin
ddanke:x:1000:1000:ddanke:/home/ddanke:/bin/bash
用户名:密码占位:UID:GID:描述信息:家目录:登录Shell

---
只看系统中的用户名
cut -d: -f1 /etc/passwd
```

#### 用户管理

```
用户新增
useradd 选项 用户名
useradd ddanke		#新增ddanke用户
设置完用户必须设置密码
passwd ddanke		#设置ddanke密码

创建用户ddanke
自动分配 UID
自动创建家目录：/home/devuser
默认 shell：/bin/bash

useradd -d /data/test ddanke			#创建时指定家目录
useradd -s /bin/bash ddanke				#创建时指定shell
useradd -r ddanke						#创建系统用户
useradd -g groupA ddanke				#创建时候指定用户组
useradd -g dev -G ops,test devuser		#创建时指定主组和附加组
-g dev：主组
-G ops,test

---
用户删除
userdel ddanke
userdel -r ddanke					#家目录/home/ddanke也一起删除

---
用户修改
修改用户信息
usermod [选项] 用户名
usermod -d /home/newdir ddanke		#修改家目录

usermod -s /bin/bash devuser
usermod -s /bin/bash ddanke			#修改登录shell
cat /etc/shells						#查看可用shell
grep ddanke /etc/passwd				#查看用户当前shell


usermod -g [新组名][用户名]
usermod -g newgroup username		#修改用户主组
usermod -g test ddanketest

usermod -aG [附加组1],[附加组2] [用户名]
usermod -aG group1,group2 username	#给用户添加附加组
usermod -aG wheel devuser			#把用户加入sudo组

gpasswd -d [用户名] [用户组名]			#从用户组中移除用户


usermod -L ddanke					#锁定用户ddanke
usermod -U ddanke					#解锁用户ddanke，锁定以后不能登录但是文件都还在
```

### 密码管理

```
passwd							#修改自己的密码

passwd username					#修改某个用户的密码
passwd ddanke

passwd -e devuser				#强制用户下次登录修改密码
```

### 用户组管理

```
查看当前用户组
groups								#查看当前用户用户组
groups ddanke						#查看某个用户
cat /etc/group						#查看系统中有哪些用户组
cut -d: -f1 /etc/group				#只看组名

用户组新增
groupadd [用户组名]
groupadd dev						#用户组新增

用户组删除
groupdel [用户组名]
groupdel dev						#用户组删除

修改用户组名
groupmod -n [新用户组名] [旧用户组名]
修改用户组 GID
groupmod -g [GID] [用户组名]
从用户组中移除用户
gpasswd [用户组名]  			


usermod -aG dev ddanke				#把用户加入组
```

### 文件属性/权限管理

![img](https://www.runoob.com/wp-content/uploads/2014/06/file-llls22.jpg)

![363003_1227493859FdXT](https://www.runoob.com/wp-content/uploads/2014/06/363003_1227493859FdXT.png)

```
查看权限
ls -l [文件名]					#查看权限
ls -ld [目录名]				#查看目录本身

---
chmod修改权限
chmod -R xyz 文件或目录
chmod 755 [文件名]
chmod 644 [文件名]

x:owner
y:group
z:others
rwx:read/write/execute
	读	写		执行
	r:4	w:2		  x:1
数字  权限  
7   rwx 
6   rw- 
5   r-x 
4   r-- 
0   --- 


---
chown更改文件所属者/所属组
chown [用户名] [文件名]
chown -R [用户名] [文件名]			#递归修改
chown [用户名]:[用户组名] [文件名]
chown -R [用户名]:[用户组名] [文件名]
-R 递归更改文件属组，就是在更改某个目录文件的属组时，如果加上 -R 的参数，那么该目录下的所有文件的属组都会更改。
---
chgrp修改文件所属组
chgrp -R 属组名 文件名
-R 递归更改文件属组，就是在更改某个目录文件的属组时，如果加上 -R 的参数，那么该目录下的所有文件的属组都会更改。
```

### dnf程序安装命令

```
dnf install [程序名]			#安装单个程序
dnf install [程序名1] [程序名2] [程序名3]		#安装多个程序
dnf install -y [程序名]		#自动确认安装

dnf search [关键字]			#搜索程序是否存在
dnf info [程序名]				#查看程序信息
dnf list installed | grep [程序名]		#查看程序是否已经安装

which [命令名]					#看命令是否存在
[命令名] --version				#查看版本信息
rpm -ql [程序名]				#查看安装了哪些文件
```

### vim使用笔记

```
i			进入编辑	
Esc			退出编辑
w			保存		
:wq			保存并退出
:q!			不保存退出

i			光标前插入
a			光标后插入
o			下一行新建并插入
O			上一行新建并插入

h j k l		左 下 上 右
0			行首
$			行尾
gg			文件开头
G			文件结尾

Ctrl + f	向下翻页
Ctrl + b	向上翻页

x			删除一个字符
dd			删除整行
d$			删到行尾
dw			删除一个单词

u			撤销
Ctrl + r	重做

yy			复制一行
p			粘贴（下）
P			粘贴（上）

/关键字		查找
n			下一个
N			上一个

:set number		打开行号
:set nonumber	关闭行号
```

### wget使用笔记

```
wget [参数] [URL]
wget https://example.com/file.tar.gz					#基础下载，下载到当前目录
wget -O [文件名] https://example.com/file.tar.gz		#指定文件名
wget -P [目录名] https://example.com/file.tar.gz		#指定下载目录
wget -c https://example.com/file.tar.gz					#断点续传网络中断可继续下载
wget -b https://example.com/file.tar.gz					#后台下载
....
```

### Nginx安装使用笔记

```
安装nginx
sudo dnf install -y nginx					#安装nginx

sudo systemctl start nginx					#启动
sudo systemctl enable nginx					#设置开机自启动

systemctl status nginx						#查看状态
```

### JDK8安装配置环境笔记

```
java -version								#查看系统是否有java

安装jdk最新版本
sudo dnf search openjdk						#查看能提供哪些jdk
安装jdk（运行环境）和jdk devel（编译、工具）
sudo dnf install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel
验证是否安装成功
java -version
javac -version

如果无jdk8需要用scp传文件
1.获取jdk8安装包
去oracle官网https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html
2.下载Linux x64	jdk-8u202-linux-x64.tar.gz版本
3.打开管理员powershell
4.用cd指令切换到jdk安装包所在的目录
5.查看Linux ip地址
	ip addr
	192.168.102.128
6.在powershell里使用指令即可传输到linux里
	scp jdk-8u202-linux-x64.tar.gz root@192.168.102.128:/usr/local/src/


配置java环境变量
1.创建目录
sudo mkdir -p /usr/lib/jvm

2.解压到刚才创建的目录
sudo tar -zxvf /usr/local/src/jdk-8u202-linux-x64.tar.gz -C /usr/lib/jvm/

3.解压后会生成目录
/usr/lib/jvm/jdk1.8.0_202

4.新建环境变量文件
sudo vim /etc/profile.d/java8.sh

5.写入
export JAVA_HOME=/usr/lib/jvm/jdk1.8.0_202
export PATH=$JAVA_HOME/bin:$PATH

6.让环境立刻生效
source /etc/profile

7.验证是否成功后
查看JAVA_HOME
echo $JAVA_HOME
	输出/usr/lib/jvm/jdk1.8.0_202
查看java版本
java -version
javac -version
	输出
	java version "1.8.0_202"
	Java(TM) SE Runtime Environment

8.完成以上步骤即可配置成功
疑问解答：为什么没有前面提到安装jdk和jdk devel，为什么步骤里面没有体现安装jdk devel的步骤
现在用的是 Oracle 官方 JDK 压缩包
里面本来就包含：
bin/java
bin/javac
bin/jmap
bin/jstack
lib/
include/
也就是说👇
JDK 压缩包 = openjdk + openjdk-devel
```

### Mysql8.0安装使用笔记

[MySQL :: Download MySQL Community Server](https://dev.mysql.com/downloads/mysql/)

```
下载mysql安装包，安装包名字如下所示
mysql-8.0.xx-linux-glibc2.28-x86_64.tar.xz

用scp上传到/usr/local/src
scp mysql-8.0.45-linux-glibc2.28-x86_64.tar.xz root@192.168.102.129:/usr/local/src

/usr/local/src/
└── mysql-8.0.xx-linux-glibc2.28-x86_64.tar.xz

/usr/local/src：放源码 / 安装包
/usr/local/mysql：放解压后的程序目录


安装并配置mysql
1.准备mysql的专业用户
groupadd mysql
useradd -r -g mysql -s /sbin/nologin mysql

2.解压并规范安装目录
cd /usr/local/src
tar -xJvf mysql-8.0.xx-linux-glibc2.28-x86_64.tar.xz
mv mysql-8.0.xx-linux-glibc2.28-x86_64 /usr/local/mysql

检查
ls /usr/local/mysql

必须看到
bin  lib  include  share  support-files

授权
chown -R mysql:mysql /usr/local/mysql

3.创建数据目录（程序和数据分离）
mkdir -p /data/mysql

授权
chown -R mysql:mysql /data/mysql


4.编写配置文件
vim /etc/my.cnf

[mysqld]
user=mysql
basedir=/usr/local/mysql
datadir=/data/mysql
socket=/tmp/mysql.sock
port=3306
character-set-server=utf8mb4
log-error=/data/mysql/mysql.err
pid-file=/data/mysql/mysql.pid

[client]
socket=/tmp/mysql.sock

5.初始化Mysql（生成系统表 + 临时密码）
cd /usr/local/mysql
bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql

查看临时root密码
grep 'temporary password' /data/mysql/mysql.err

6.启动mysql
/usr/local/mysql/support-files/mysql.server start

7.登录并修改 root 密码
/usr/local/mysql/bin/mysql -uroot -p

ALTER USER 'root'@'localhost' IDENTIFIED BY '00000';

退出
exit

9.设置开机自启
sudo systemctl enable mysqld

加入 PATH（可选）
echo 'export PATH=/usr/local/mysql/bin:$PATH' > /etc/profile.d/mysql.sh
source /etc/profile.d/mysql.sh

开放3306端口
sudo firewall-cmd --add-port=3306/tcp --permanent
sudo firewall-cmd --reload

创建远程账号
CREATE USER 'ddanke'@'%' IDENTIFIED BY '000000';
GRANT ALL PRIVILEGES ON *.* TO 'appuser'@'%';
FLUSH PRIVILEGES;
```

### 程序部署笔记

```
测试
```

