---
title: Linux系统命令及Shell脚本实践指南
date: 2022-04-25 14:28:05
tags:
  - linux系统
  - shell脚本
  - 读书笔记
---

## linux简介
### linux的特点
1. **免费开源。**Linux是一款完全免费的操作系统，任何人都可以从网络上下载到它的源代码，并可以根据自己的需求进行定制化的开发，而且没有版权限制。
2. **模块化程度高。**Linux的内核设计分成进程管理、内存管理、进程间通信、虚拟文件系统、网络5部分，其采用的模块机制使得用户可以根据实际需要，在内核中插入或移走模块，这使得内核可以被高度的剪裁定制，以方便在不同的场景下使用。
3. **广泛的硬件支持。**得益于其免费开源的特点，有大批程序员不断地向Linux社区提供代码，使得Linux有着异常丰富的设备驱动资源，对主流硬件的支持极好，而且几乎能运行在所有流行的处理器上。
4. **安全稳定。**Linux采取了很多安全技术措施，包括读写权限控制、带保护的子系统、审计跟踪、核心授权等，这为网络环境中的用户提供了安全保障。实际上有很多运行Linux的服务器可以持续运行长达数年而无须重启，依然可以性能良好地提供服务，其安全稳定性已经在各个领域得到了广泛的证实。
5. **多用户，多任务。**多用户是指系统资源可以同时被不同的用户使用，每个用户对自己的资源有特定的权限，互不影响。多任务是现代化计算机的主要特点，指的是计算机能同时运行多个程序，且程序之间彼此独立，Linux内核负责调度每个进程，使之平等地访问处理器。由于CPU处理速度极快，从用户的角度来看所有的进程好像在并行运行。
6. **良好的可移植性。**Linux中95%以上的代码都是用C语言编写的，由于C语言是一种机器无关的高级语言，是可移植的，因此Linux系统也是可移植的。


### linux启动流程
1. 首先，计算机会加载BIOS，这是计算机上最接近硬件的软件，各家主板制造商都会开发适合自己主板的BIOS，而BIOS中一项很重要的功能就是对自身的硬件做一次健康检查，只有硬件没有问题，才能运行软件，记住，操作系统也是一种软件
2. 机器自检通过后，下面就要引导系统了。这个动作是BIOS设定的，BIOS默认会从硬盘上的第0柱面、第0磁道、第一个扇区中读取被称为MBR的东西，即主引导记录
3. 第三步就是顺理成章地运行Grub了。Grub最重要的功能就是根据其配置文件加载kernel镜像，并运行内核加载后的第一个程序/sbin/init，这个程序会根据/etc/inittab来进行初始化的工作
4. Linux将根据/etc/inittab中定义的系统初始化配置si::sysinit:/etc/rc.d/rc.sysinit执行/etc/rc.sysinit脚本，该脚本将会设置系统变量、网络配置，并启动swap、设定/proc、加载用户自定义模块、加载内核设置等。
5. 根据第三步读到的runlevel值来启动对应的服务，如果值为3，就会运行/etc/rc3.d/下的所有脚本，如果值为5，就会运行/etc/rc5.d/下的所有脚本。
6. 将运行/etc/rc.local
7. 会生成终端或X Window来等待用户登录


### linux帮助命令
#### 使用man page
```sh
man ls
```

#### 使用info page
可以在命令行中输入info ls来显示ls命令的说明文
```sh
info ls
```

## linux用户管理
### linux用户和用户组
#### uid和gid
Linux系统中的用户分为3类，**即普通用户、根用户、系统用户。**
1. 普通用户: 通常普通用户的UID大于500，因为在添加普通用户时，系统默认用户ID从500开始编号。
2. 根用户: 根用户也就是root用户，它的ID是0，也被称为超级用户，root账户拥有对系统的完全控制权：可以修改、删除任何文件，运行任何命令。所以root用户也是系统里面最具危险性的用户，root用户甚至可以在系统正常运行时删除所有文件系统，造成无法挽回的灾难。
3. 系统用户：系统用户是指系统运行时必须有的用户，但并不是指真实的使用者。系统用户的ID范围是1~499 

```sh
#要确认自己的UID，可以使用以下id命令来获得：
id

#要确认自己所属的用户组，可以使用以下groups命令来获得
groups

#如果要查询当前在线用户，可在用户登录以后，使用命令who看到目前登录在系统中的所有用户
who
```

#### /etc/passwd 和 /etc/shadow
在登录Linux时必须要输入用户名和密码。而系统用来记录用户名、密码最重要的两个文件就是/etc/passwd 和 /etc/shadow(而且默认只有root用户才有读的权限，其他人完全没有读取这个文件的可能)
### linux账号管理
#### 新增和删除用户
1. 新增用户： useradd
2. 修改密码： passwd
3. 修改用户： usermod
4. 删除用户： userdel

#### 新增和删除用户组
1. 新增用户组： groupadd
2. 删除用户组： groupdel

#### 检查用户信息
1. 查看用户： users、who、w
```sh
#使用命令users可以查看当前系统有哪些用户。比如，在当前的系统中运行users命令，就会发现有两个root在当前机器上登录。
users

#第一列是登录用户的用户名，第二列是用户登录的终端，第三列是用户登录的时间。
who

#w命令的第一行会显示当前时间、系统运行时间、已登录的用户数量和系统负载。下面显示的信息分为8列
w
```

2. 调查用户：finger
```sh
#如果在finger后跟上某个用户名，则显示该用户更详细的信息，
finger user1
```

### linux切换用户
#### 切换其他用户
假如说我以普通用户john登录到系统中，这时候想使用useradd添加一个用户，怎么办？普通用户是没有添加用户的权限的，只有root用户才能创建用户。当然我们可以使用exit命令退出当前用户，然后使用root用户登录，再使用useradd添加用户。但是也有一种更方便的方式，那就是使用su命令，su是切换用户的意思。在不加参数的情况下，su命令默认表示切换到root用户，之后只要输入root密码就可以切换身份为root了，完成操作后，使用exit命令可以退出root切换到原先的用户。如下所示：
```sh
#su命令默认表示切换到root用户，之后只要输入root密码就可以切换身份为root了，
#完成操作后，使用exit命令可以退出root切换到原先的用户
su

#加上-这个参数后，切换成root用户时，不但身份变成了root，而且还能应用root的用户环境
su -
```
**使用su命令切换用户之后，当前的用户环境并没有发生变化，而使用su-命令切换用户后，用户环境变成root的了**
#### 用其他用户的身份执行命令：sudo
sudo并不是真的切换了用户，而是使用其他用户的身份和权限执行了命令。



### linux例行任务管理
>在Linux中也有处理这两种任务的方法。如果任务是周期性执行的，其命令为cron；如果只是在某一个特定的时间执行一次，其命令为at。
#### 单一时刻执行一次任务：at
*默认情况下，所有用户都可以使用at命令来调度自己的任务，如果由于特殊的原因需要禁止某些用户使用这个功能，可以将该用户的用户名添加至/etc/at.deny*

#### 周期性执行任务：cron
*有一些任务是需要周期性执行的，比如说每天早晨的闹钟会在设定的时间准时响起*
1. 启动crond进程
```sh
service crond start

service crond status
```
2. 编辑任务crontab -e
```sh
* * * * * command

#前面5个*可以用来定义时间，
#第一个*表示分钟，可以使用的值是1~59，每分钟可以使用*和*/1表示；
#第二个*表示小时，可以使用的值是0~23；
#第三个*表示日期，可以使用的值是1~31；
#第四个*表示月份，可以使用的值是1~12；
#第五个*表示星期几，可以使用的值是0~6，0代表星期日；
#最后是执行的命令
```

3. 示例
```sh
*  *  *  *  * service httpd restart
*/1  *  *  *  * service httpd restart
#这两种写法其实是一致的，都是每分钟重启httpd进程。请注意，这只是一个例子，除非你有确定的目的，否则不要在实际生产环境中这么设置

*  */1  *  *  * service httpd restart
#每小时重启httpd 进程


*  23-3/1  *  *  * service httpd restart
#从23点开始到3点，每小时重启httpd 进程


30 23 *  *  * service httpd restart
#每天晚上23点30分重启httpd进程


30 23 1  *  * service httpd restart
#每月的第一天晚上23点30分重启httpd进程


30 23 1  1  * service httpd restart
#每年1月1日的晚上23点30分重启httpd进程


30 23 *  *  0 service httpd restart
#每周日晚上23点30分重启httpd进程
```

4. 查看定时任务 crontab -l
   
*如果由于特殊的原因需要禁止某些用户使用这个功能，可以将该用户的用户名添加至/etc/cron.deny中*

#### /etc/crontab管理
我们知道用户可以通过crontab-e命令来编辑定义自己的任务，事实上，系统也有自己的例行任务，而其配置文件是/etc/crontab。
```sh
[root@localhost ~]# cat /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/

# run-parts
01 * * * * root run-parts /etc/cron.hourly
02 4 * * * root run-parts /etc/cron.daily
22 4 * * 0 root run-parts /etc/cron.weekly
42 4 1 * * root run-parts /etc/cron.monthly
```

## linux文件管理
### 文件和目录管理
#### 文件的相关操作
Linux遵循一切皆文件的规则，对Linux进行配置时，在很大程度上就是处理文件的过程，所以掌握文件的相关操作是非常有必要的
1. 创建文件：touch
2. 删除文件：rm
3. 移动或重命名文件： mv，后面需要跟两个参数，**第一个参数是要被移动的文件，第二个参数是移动到的目录**
```sh
# 把test.log文件移动到/mnt中
mv test.log /mnt/ 
# 把test.log重命名为test1.log
mv test.log test1.log
# 移动文件的同时重命名文件
mv test.log /mnt/test2.log
```
4. 查看文件：cat
5. 查看文件头： head
```sh
# 默认情况下，head将显示该文件前10行的内容;也可以使用-n参数指定显示的行数
head -n 20 /text.log
```
6. 查看文件尾: tail
```sh
# 要动态地查看文件，使用-f参数就可以做到
tail -f /text.log
```
7. 文件格式转换：dos2unix, 当把Windows下的文本文件移动到Linux下时，会由于系统之间文本文件的换行符不同而造成文件在Linux下的读写操作有问题

#### 目录的相关操作
1. 创建目录：mkdir
```sh
# 可以使用-p参数一次性创建所有目录，这样就不用费力地一个个创建了，命令如下所示
mkdir -p dir2/dir3
```
2. 删除目录：rmdir和rm
```sh
# rmdir 用来删除目录。但是需要注意的是，它只能删除空目录，如果目录不为空（存在文件或者子目录），那么该命令将拒绝删除指定的目录
rmdir dir1 #提示dir1为非空目录

# rm
rm -rf dir1
```
3. 文件和目录复制：cp
```sh
# 复制文件且重命名
cp a.txt /tmp/b.txt

# 复制文件不重命名
cp a.txt /tmp/

# 复制目录所有文件
cp -rf /dir1/* /dir2/ 
```

#### 文件时间戳
通过touch可以创建新文件。如果文件已经存在，那么touch命令仅仅会更新文件的创建时间而不会修改文件内容。请记住，在Linux下目录也是一种文件，所以如果touch一个目录，这个目录的创建时间也会被更新

### 文件和目录权限
#### 查看文件或者目录权限: ls -al
```sh
# -l参数表示要求ls命令列出每个文件的详细信息，
# -a参数则要求ls命令还要同时列出隐藏文件
ls -al 
[root@localhost ~]# ls -al
total 112
drwxr-x---  3 root root  4096 Oct  1 10:43 .
drwxr-xr-x 24 root root  4096 Oct  1 07:42 ..
-rw-------  1 root root  5659 Sep 24 02:07 .bash_history
-rw-r--r--  1 root root    24 Jan  6  2007 .bash_logout
-rw-------  1 root root    72 Oct  1 08:45 .lesshst
drwx------  2 root root  4096 Oct  1 08:48 .ssh

#第一列：是文件类别和权限，这列由10个字符组成，第一个字符表明该文件的类型
#第二列：代表“连接数”，除了目录文件之外，其他所有文件的连接数都是1，目录文件的连接数是该目录中包含其他目录的总个数+2，也就是说，如果目录A中包含目录B和C，则目录A的连接数为4
#第三列：代表该文件的所有人
#第四列：代表该文件的所有组
#第五列：是该文件的大小
#第六列：是该文件的创建时间或最近的修改时间
#第七列：是文件名
```

第一列含义：
<img src="/img/linux1.jpeg" style="max-width:95%" />

#### 改变文件权限：chmod
Linux下的每个文件都定义了文件拥有者（user）、拥有组（group）、其他人（others）的权限，我们使用字母u、g、o来分别代表拥有者、拥有组、其他人，而对应的具体权限则使用rwx的组合来定义，增加权限使用+号，删除权限使用-号，详细权限使用=号。图中用一些例子说明了如何使用chmod来改变文件的权限。
<img src="/img/linux2.jpeg" style="max-width:95%" />
如果要给用户组或其他人添加或删除相关权限，只需要将上面的u相应地更换成g或o即可。但是正如大家看到的，这种方式同一时刻只能给文件拥有者、文件拥有组或是其他所有人设置权限，如果要想同时设置所有人的权限就需要使用数字表示法了，我们定义r=4，w=2，x=1，如果权限是rwx，则数字表示为7，如果权限是r-x，则数字表示为5。假设想设置一个文件的权限是：拥有者的权限是读、写、执行（rwx），拥有组的权限是读、执行（r-x），其他人的权限是只读（r--），那么可以使用命令chmod 754 somefile来设置

```sh
# 修改某个目录权限
chmod -R 754 somedir
```
#### 改变文件的拥有者：chown
```sh
# 修改文件拥有者
chown chao a.txt

# 修改文件拥有组
chown :chao a.txt

# 同时修改文件拥有者和拥有组
chown chao:chao a.txt

# 同时修改目录拥有者和拥有组
chown -R chao:chao somedir
```
#### 改变文件的拥有者：chown
```sh
chgrp -R chao somedir
```

#### 查看文件类型：file
使用ls-l令可以通过查看第一个字符判断文件类型。字母d代表目录、字母l代表连接文件，字母b代表块文件，字母c代表字符文件，字母s代表socket文件，字符-代表普通文件，字母p代表管道文件

### 查找文件
#### 一般查找 find
```sh
# 从根目录开始寻找名为httpd.conf的文件
find / -name httpd.conf

# 从etc目录开始寻找名为httpd.conf的文件
find /etc -name httpd.conf

# 查找系统所有以conf结尾的文件
find / -name *.conf

# 查找系统所有以http开头文件
find / -name httpd*
```
<img src="/img/linux3.jpeg" style="max-width:95%" />


#### 数据库查找 locate
与find不同，locate命令依赖于一个数据库文件，Linux系统默认每天会检索一下系统中的所有文件，然后将检索到的文件记录到数据库中，所以使用locate命令要比find命令反馈更为迅速。
```sh
# 更新数据库 --> 查找文件
updatedb
find /etc -name httpd.conf
```

#### 查找执行文件 which/whereis
which用于从系统的PATH变量所定义的目录中查找可执行文件的绝对路径。比如说想查找node这个命令在系统中的绝对路径
```sh
which node

# 使用whereis也能查到其路径，但是和which不同的是，
# 它不但能找出其二进制文件，还能找出相关的man文件：
whereis node
```

### 文件压缩和打包
#### gzip/gunzip
gzip/gunzip是用来压缩和解压缩单个文件的工具
```sh
# 压缩install.log文件
gzip install.log  # 最终压缩的文件名：install.log.gz

# 解压 install.log.gz
gunzip install.log.gz
```
#### tar
tar不但可以打包文件，还可以将整个目录中的全部文件整合成一个包，整合包的同时还能使用gzip的功能进行压缩，比如说把整个/boot目录整合并压缩成一个文件
```sh
# 压缩命令: 压缩/boot目录成bott.tgz
# -z的含义是使用gzip压缩
# -c是创建压缩文件（create）
# -v是显示当前被压缩的文件
# -f是指使用文件名，也就是这里的boot.tgz文件
tar -zcvf boot.tgz /boot

# 压缩命令 讲zain目录 压缩为dist.tar.gz
tar -czf dist.tar.gz zain/


# 解压命令: 直接将boot.tgz在当前目录中解压成boot目录
tar -zxvf boot.tgz

# 解压命令：使用-c参数指定压缩后的目录存放位置。比如说将boot目录解压到/tmp目录中
tar -zxvf boot.tgz -c /tmp
```

## linux文件系统
### 文件系统
大家已经知道Linux使用了树形文件存储结构，在磁盘上存储文件的时候，使用的则是目录加文件的形式。但实际上对于磁盘等各种存储设备来说，无论是什么数据，都只有0和1的概念。而对用户来说，0和1同样毫无意义，那怎么办呢？这就需要一种类似于“翻译”的机制存在于用户和磁盘之间了，在Linux中采用的是文件系统+虚拟文件系统（Virtual File System，VFS）的解决方案。

### 磁盘分区、创建文件系统、挂载
#### 创建文件系统：fdisk

#### 磁盘挂载：mount
创建了文件系统的分区后，在Linux系统下还需要经过挂载才能使用，挂载设备的命令是mount，

#### 磁盘检查：fsck、badblocks
当磁盘出现逻辑错误时，可以使用fsck来尝试修复。出现此类错误比较典型的情况是当机器突然掉电时可能引发。

### 硬链接和软连接
#### 硬链接
硬链接（hard link）又称实际链接，是指通过索引节点来进行链接。在Linux文件系统中，所有的文件都会有一个编号，称为inode，多个文件名指向同一索引节点是被允许的，这种链接就是硬链接。硬链接的作用是允许一个文件拥有多个有效路径名，这样用户就可以建立硬链接指向同一文件，删除一个链接并不会影响索引节点本身和其他的链接，只有当最后一个链接被删除时，文件的数据块及目录的链接才会被释放。也就是说，文件真正删除的前提条件是与之相关的所有硬链接均被删除。**硬链接有两个限制：**
1. 不允许给目录创建硬链接；
2. 只有在同一文件系统中的文件之间才能创建链接，即不同分区上的两个文件之间不能够建立硬链接。

```sh
# 创建目录
mkdir -p hard
cd hard
touch hard01

# 查看inode
ls -li
3834061-rw-r--r-- 1 root root 0 Jan 15 10:50 hard01

# 创建hard01的硬链接hard01_hlink
# 硬链接hard01_hlink指向的inode和hard01指向inode是一致的
ln hard01 hard01_hlink 

# ls -li 查看文件inode
# 文件创建之初该值为1，该文件每增加一个硬链接该值将增1(变成2)，
# 当此数为0的时候该文件才能真正被文件系统删除
ls -li
3834061-rw-r--r-- 2 root root 0 Jan 15 10:50 hard01
3834061-rw-r--r-- 2 root root 0 Jan 15 10:50 hard01_hlink
```

#### 软连接
软链接（soft link）又称符号链接（symbolic link），是一个包含了另一个文件路径名的文件，可以指向任意文件或目录，也可以跨不同的文件系统。软链接和Windows下的“快捷方式”十分类似，删除软链接并不会删除其所指向的源文件，如果删除了源文件则软链接会出现“断链”。
```sh
# 创建目录
mkdir -p soft
cd soft
touch soft01

# 创建软连接使用-s，
ln -s soft01 soft01_slink

ls -li
3834063-rw-r--r-- 1 root root 0 Jan 15 10:50 soft01
3834064-rw-r--r-- 1 root root 0 Jan 15 10:50 soft01_slink
```
另外还请注意，在创建软链接的前后分别使用ls -li命令，会发现软链接的inode和源文件的inode不一样，这说明软链接本身就是一个文件。
读者可以尝试删除软链接的源文件，然后可以在终端中看到对应的软链接将会以闪烁的方式标记其已是一个断链。

## 字符处理
### 管道
在Linux中也存在着管道，它是一个固定大小的缓冲区，该缓冲区的大小为1页，即4K字节。管道是一种使用非常频繁的通信机制，我们可以用管道符“|”来连接进程，由管道连接起来的进程可以自动运行，如同有一个数据流一样，所以管道表现为输入输出重定向的一种方法，它可以把一个命令的输出内容当作下一个命令的输入内容，两个命令之间只需要使用管道符连接即可。
```sh
# 如果想要看一下/etc/init.d目录下文件的详细信息，
# 可以使用ls-l/etc/init.d命令，不过这可能会出现因输出内容过多而造成翻屏的情况，
# 这样一来，先输出的内容在屏幕上就看不到了。
# 其实这里就可以利用管道功能，将命令的输出使用more程序一页一页地显示出来
ls -l /etc/init.d | more
```
### 使用grep搜索文本
```sh
grep [-ivnc] '需要匹配字符' 文件名
# -i 不区分大小写
# -c 统计包括匹配的行数
# -n 输出行号
# -v 反向匹配
```

案例：
```sh
# 查找含有name的行
grep 'name' txt1.txt

# 查找含有name的行编号
grep -n 'name' txt1.txt

# 查找含有name的行编号， 忽略大小写
grep -ni 'name' txt1.txt

# 管道
cat txt1.txt | grep -ni 'name'
```

### 使用sort排序
```sh
sort [-ntkr] 文件名
# -n 采用数字排序
# -t 指定分隔符
# -k 指定第几行
# -r 反向排序
```

案例：
```sh
# cat sort.txt
b:3
c:2
a:4
e:5
d:1
f:11 

# cat sort.txt | sort 
a:4
b:3
c:2
d:1
e:5
f:11
#对输出内容直接排序时，默认按照每行的第一个字符进行排序

# cat sort.txt | sort -t ":" -k 2-n
d:1
c:2
b:3
a:4
e:5
f:11
```

### 使用uniq删除重复内容
如果文件（或标准输出）中有多行完全相同的内容，我们很自然希望能删除重复的行，同时还可以统计出完全相同的行出现的总次数，uniq命令就能帮助解决这个问题。
```sh
uniq [-ic]
# -i 忽略大小写
# -c 计算重复行数
```

案例：
```sh
# cat uniq.txt | uniq
abc
123
abc
123

# cat uniq.txt | sort | uniq
123
abc

#使用-c参数就会在每行前面打印出该行重复的次数
# cat uniq.txt | sort | uniq -c
2 123
2 abc
```


### 使用cut截取文本
顾名思义，cut就是截取的意思，它能处理的对象是“一行”文本，可从中选取出用户所需要的部分。在有特定的分隔符时，可以指定分隔符，然后打印出以分隔符隔开的具体某一列或某几列
```sh
cut -f指定行 -d'分隔符'
cut -c指定列的字符
```

案例
```sh
# 比如说我们需要打印出系统中的所有用户
cat /ect/passwd | cut -f1 -d':'

# 同时打印出用户和这个用户的家目录
cat /ect/passwd | cuf -f1,6 -d':'

# 同时打印出每位用户的登录shell
cat /ect/passwd | cuf -f1,6-7 -d':'

# 假设想要打印出每行第1～5个字符，以及第7～10个字符的内容
cat /ect/passwd | cuf -c1-5,7-10
```

### 使用tr做文本转换
tr命令比较简单，其主要作用在于文本转换或删除。这里假设要把文件/etc/passwd中的小写字母转换为大写字母，然后再尝试删除文本中的冒号
```sh
cat /etc/passwd | tr '[a-z]' '[A-Z]'

cat /etc/passwd | tr -d ':'
```

### 使用paste做文本合并
paste的作用在于将文件按照行进行合并，中间使用tab隔开。假设有两个文件分别为a.txt、b.txt，下面使用paste命令来合并文件
```sh
# cat a.txt
1
2
3

# cat b.txt
a
b
c

# past a.txt b.txt
1 a
2 b
3 c

# past -d: a.txt b.txt
1:a
2:b
3:c
```

### 使用split分割大文件
在Linux下使用split命令来实现文件的分割，支持按照行数分割和按照大小分割这两种模式。要说明的是，二进制文件因为没有“行”的概念，所以二进制文件无法使用行分割，而只能按照文件大小进行分割。
```sh
# -l参数：指定每500行为一个文件 
# 分割完成后，当前目录下会产生很多小文件
split -l 500 big_file.txt small_file_

# 如果是二进制文件， 只能按照文件大小分割
# 分割完成后，当前目录下会产生很多大小为6m的小文件
split -b 64m big_bin small_bin_
```

## 网络管理
### 网络接口
#### ifconfig检查和配置网卡
如果不使用任何参数，输入ifconfig命令时将会输出当前系统中所有处于活动状态的网络接口。
<img src="/img/linux4.jpeg" style="max-width:95%" />
图中的eth0表示的是以太网的第一块网卡。其中eth是Ethernet的前三个字母，代表以太网，0代表是第一块网卡，第二块以太网网卡则是eth1，以此类推。Linkencap是指封装方式为以太网；HWaddr是指网卡的硬件地址（MAC地址）；inetaddr是指该网卡当前的IP地址；Broadcast是广播地址（这部分是由系统根据IP和掩码算出来的，一般不需要手工设置）；Mask是指掩码；UP说明了该网卡目前处于活动状态；MTU代表最大存储单元，即此网卡一次所能传输的最大分包；RX和TX分别代表接收和发送的包；collision代表发生的冲突数，如果发现值不为0则很可能网络存在故障；txqueuelen代表传输缓冲区长度大小；第二个设备是lo，表示主机的环回地址，这个地址是用于本地通信的。

**手动设置etho的ip地址**
```sh
ifconfig etho 192.168.159.130 netmask 255.255.255.0

# 有时候需要手工断开/启用网卡，以eth0为例，使用方法如下：
ifconfig eth0 down #断开
ifconfig etho up #启用
```

#### 将ip配置信息写入配置文件
上一小节讲到的ifconfig命令可以直接配置网卡IP，但是这属于一种动态的配置，所配置的信息只是保存在当前运行的内核中。一旦系统重启，这些信息将丢失。为了能在重启后依然生效，可以在相关的配置文件中保存这些信息，这样，系统重启后将从这些配置文件中读取出来。RedHat和CentOS系统的网络配置文件所处的目录为/etc/sysconfig/network-scripts/，eth0的配置文件为ifcfg-eth0，如果有第二块物理网卡，则配置文件为ifcfg-eth1，以此类推
```sh
cat ifcfg-eth0

# Advanced Micro Devices [AMD] 79c970 [PCnet32 LANCE]
DEVICE=eth0 # DEVICE变量定义了设备的名称
BOOTPROTO=static # 系统在启用这块网卡时，IP将会通过dhcp的方式获得；还有个可选的值是static，表示静态设置的IP
ONBOOT=yes # ONBOOT变量定义了启动时是否激活使用该设备，yes表示激活，no表示不激活
IPADDR=192.168.159.129
NETMASK=255.255.255.0

# 生效操作
ifconfig eth0 down
ifconfig eth0 up
# 或者
service network restart
```

### 路由和网关
Linux主机之间是使用IP进行通信的，假设A主机和B主机同在一个网段内且网卡都处于激活状态，则A具备和B直接通信的能力（通过交换机或简易HUB）。但是如果A主机和B主机处于两个不同的网段，则A必须通过路由器才能和B通信。一般来说，路由器属于IT设备的基础设施，每一个网段都应该有至少一个网关。在Linux中可使用route命令添加默认网关。假设添加的网关是192.168.159.2，添加方式如下
```sh
# 添加
route add default gw 192.168.159.2

# 删除
route del default gw 192.168.159.2
```

如果只使用route命令添加网关，一旦系统重启，配置信息就不存在了，必须将这种配置信息写到相关的配置文件中才能永久保存。可以在网卡配置文件中使用GATEWAY变量来定义网关，只需要添加如下部分到ifcfg-eth0中即可，当然别忘了重启网络服务使配置生效
```sh
GATEWAY=192.168.159.2
```
另外，在配置文件/etc/sysconfig/network中添加这段配置也能达到同样的效果。

### DNS客户端配置
#### /etc/hosts
们使用hosts文件来记录主机名和IP的对应关系，这样访问对方的主机时，就不需要使用IP了，只需要使用主机名。这个文件在Linux下就是/etc/hosts，这种方式确实“可以工作”，但是当主机数量增长到一定数量级的时候仍然无法适用。为了彻底解决这个问题，人们发明了DNS系统。经过几十年的发展，虽然系统、网络技术都发生了翻天覆地的变化，但是这个文件还是被当作传统保留了下来。hosts文件的作用主要如下：
* 加快域名解析。当访问网站时，系统会首先查看hosts文件中是否有记录，如果记录存在则直接解析出对应的IP，这时则不需要请求DNS服务器
* 方便小型局域网用户使用的内部设备。很多单位的局域网中都存在着不少内部应用系统（比如办公自动化OA、公司论坛等），平时在工作中也都需要访问，但是由于这些局域网太小而不必为此专门设置DNS服务器，那么此时使用hosts文件则能简单地解决这个问题。

#### /etc/resolv.conf
从技术上来说，DNS就是全互联网上主机名及其IP地址对应关系的数据库。设置主机为DNS客户端的配置文件就是/etc/resolv.conf，其中包含nameserver、search、domain这3个关键字
```sh
# search关键字后紧跟的是一个域名。
# 每个主机严格来说都应该有一个FQDN（全限定域名），所以往往域名就很长，
# 如果这里写成search google.com，那么www就代表www.google. com了，
# 这个关键字后可以跟多个域名。
search bigo.local 
nameserver 172.24.8.15 # 172.24.8.15为DNS主机的IP地址
nameserver 172.24.8.16
```

### 网络测试工具
#### ping
ping程序的目的在于测试另一台主机是否可达，一般来说，如果ping不到某台主机，就说明对方主机已经出现了问题，但是不排除由于链路中防火墙的因素、ping包被丢弃等原因而造成ping不通的情况
```sh
ping 10.0.1.1.145
```
#### host
host命令是用来查询DNS记录的，如果使用域名作为host的参数，命令返回该域名的IP
```sh
# host baidu.com
baidu.com has address 220.181.38.148
baidu.com has address 220.181.38.251
baidu.com mail is handled by 10 mx.maillb.baidu.com.
baidu.com mail is handled by 20 mx1.baidu.com.
baidu.com mail is handled by 20 jpmx.baidu.com.
baidu.com mail is handled by 20 mx50.baidu.com.
baidu.com mail is handled by 20 usmx01.baidu.com.
baidu.com mail is handled by 15 mx.n.shifen.com.
```

#### 网络故障排查
1. 硬件故障又主要分为网卡物理损坏、链路故障等原因。
2. 软件主要表现为网卡驱动故障，也就是操作系统对网卡驱动的不兼容，这个问题往往需要通过安装对应的网卡设备驱动来解决
```sh
# 第一步:确认网卡本身是否能正常工作? 
ping 127.0.0.1

# 第二步确认网卡是否出现了物理或驱动故障，使用ping本机IP地址的方式，如果能ping通则说明本地设备和驱动都正常

# 第三步要确认是否能ping通同网段的其他主机

# 第四步要确认是否能ping通网关IP

# 第五步确认是否能ping通公网上的IP

# 第六步确认是否能ping通公网上的某个域名，如果能ping通则说明DNS部分设置正确
```
## 进程管理
### 什么是进程
进程表示程序的一次执行过程，它是应用程序的运行实例，是一个动态的过程。或者可以更简单地描述为：进程是操作系统当前运行的程序。当一个进程开始运行时，就是启动了这个过程。

所有的进程都可能存在3种状态：运行态、就绪态、阻塞态。运行态表示程序当前实际占用着CPU等资源；就绪态是指程序除CPU之外的一切运行资源都已经就绪，等待操作系统分配CPU资源，只要分配了CPU资源，即可立即运行；而阻塞态是指程序在运行的过程中由于需要请求外部资源（例如I/O资源、打印机等低速的或同一时刻只能独享的资源）而当前无法继续执行，从而主动放弃当前CPU资源转而等待所请求资源。

而进程同步指的是进程间通过某种通信机制实现信息交互。现代计算机使用信号量机制来实现进程间的互斥和同步，它的基本原理是：两个或者多个进程可以通过简单的信号进行合作，一个进程可以被迫在某一位置停止，直到它接收到一个特定的信号。
### 进程、线程、程序区别
#### 进程 VS 程序
进程是动态的，而程序是静态的，进程是程序以及数据在计算机上的一次执行，没有静态的程序也就没有动态的执行。程序是可以以某种形式保存在存储介质上的，而进程只能在运行时存在于计算机的内存中。
*如果说做一件事情需要经过若干既定的步骤，这些步骤可以被写成清单静态地列在纸上，那么它们就是广义上的“程序”，而只有真正开始将计划的步骤付诸实施的过程才是“进程”*


#### 进程 VS 线程
进程是资源分配的最小单位，线程是CPU调度的最小单位。做个简单的比喻：进程=火车，线程=车厢。
1. 线程在进程下行进（单纯的车厢无法运行）
2. 一个进程可以包含多个线程（一辆火车可以有多个车厢）
3. 同一进程下不同线程间数据很易共享（A车厢换到B车厢很容易）
4. 进程要比线程消耗更多的计算机资源（采用多列火车相比多个车厢更耗资源）

### 进程观察： ps、top
1. 命令ps输出的只是当前查询状态下进程瞬间的状态信息，如果要想及时动态地查看进程就需要使用top命令了。
2. top命令提供了实时的系统状态监控，可以按照CPU使用、内存使用、执行时间等指标对进程进行排序

```sh
ps [-Aawu]
# -A 列出所有进程
# -a 显示和本终端不相关的所有进程
# -w 显示加宽可以显示更多信息
# -u 显示有效使用者相关进程

top 
```

### 进程的终止： kill、killall
kill这些命令的原理都是向内核发送一个系统操作信号以及某个进程的标识号，使得内核对指定标识号的进程进行相应的操作。 典型用法是使用ps查出进程的PID，然后使用kill将其终止
```sh
# 第一步
ps -ef | grep dhcp # 2899
# 或者
pidof dhcp  # 2899

# 第二步
kill 2899
```

命令kill后可以跟的信号代码一共有64种，使用kill-l就可以看到具体有哪些，如图所示。但是常用的一般只有3个，即HUP（1）、KILL（9）、TERM（15），分别代表重启、强行杀掉、正常结束。
<img src="/img/linux5.jpeg" style="max-width:95%" />

使用kill-1重启进程的时候实际上是不会改变主进程的PID的，也就是说只是发生了原地重启，或者说“软重启”
```sh
# 软重启
kill -1 2899

# 如果要停止系统中所有的httpd进程，
# 那么只要按照以下方法操作
killall httpd
```

### 查询进程打开文件：lsof
Linux中一切皆文件，所以在系统中，被打开的文件可以是普通文件、目录、网络文件系统中的文件、字符设备、管道、socket等。那么如何知晓现在系统打开的是哪些文件呢，这时lsof命令就有用武之地了。
#### 查看进程
使用lsof还可以查找使用了某个端口的进程，比如说如果系统中运行了sshd进程（基本上都是默认运行的），则该进程默认会绑定22端口，让我们来确认一下
```sh
lsof -i:22
```

#### 查看什么进程使用该文件
```sh
lsof /var/log/messages
```

#### 恢复被删除的文件， 此文件必须正在被某个进程使用
*此文件必须正在被某个进程使用，也就是依然拥有打开文件的句柄能力。*

现假设文件/var/log/messages不小心被删除了，首先来确认一下当前是否有进程正在使用这个文件，如果有则可以继续，如果没有就无法使用该方法继续了。本例中看到有个PID为2449的进程正在使用该文件，那么接下来只要找到对应/proc目录下的文件就可以了
```sh
# lsof | grep message 
syslogd   2449      root    1w      REG      253,0   149423    4161767 /var/log/messages
# cat /proc/2449/fd/2 > /var/log/messages
# Service syslogd restart
```

### 进程优先级调整：nice、renice

## vi和vim编辑器
vim编辑器是vi的加强版，在简单的文本操作上与vi几乎完全一致，所以习惯使用vi的人可以完全无缝地切换使用vim编辑器。同时vim还增加了很多新功能，包括代码补全、错误跳转等，可方便编程

### vim编辑器使用
1. 搜索关键词
```sh
/keyword # 搜索keyword关键词
```
2. 替换关键词
```sh
:s/keyword/kw/g #将本行的keyword替换成kw

:1,$s/keyword/kw/g #将1行到最后一行所有的keyword替换成kw
```

## 正则表达式
### 正则基础
1. “ .”（一个点）符号
点符号用于匹配除换行符之外的任意一个字符。例如：r.t可以匹配rot、rut
```sh
grep 'r..t' /etc/passwd
```
2. “ \*”符号
"*"符号用于匹配前一个字符 0次 或 任意多次，比如ab*，可以匹配a、ab、abb等。“*”号经常和“.”符号加在一起使用
```sh
# 查找包含字母r，后面紧跟任意长度的字符，再跟一个字母t的行
grep 'r.*t' /etc/passwd
```

3. “ \\{n,m\\}”符号
```sh
# \{n\} 匹配前面的字符n次
grep 'ro\{2\}t' /etc/passwd

# \{n,\} 匹配前面的字符至少n次以上（含n次)
grep 'ro\{2,\}t' /etc/passwd

# \{n,m\} 匹配前面的字符n到m次
grep 'ro\{2,3\}t' /etc/passwd
```
4. “ ^”符号
```sh
# 这个符号用于匹配开头的字符。比如说“^root”匹配的是以字母root开始的行
grep '^root' /etc/passwd
```
5. “ $”符号
```sh
# “$”用于匹配尾部，比如说“abc$”代表的是以abc结尾的行。
# 如果是“^$”则代表该行为空，因为^和$间什么都没有
grep 'r.*t$' /etc/passwd
```
6. “ []”符号
这是一对方括号，用于匹配方括号内出现的任一字符
```sh
# 匹配abc任意字符
[abc]

# 比如说要匹配任意一个大写字母，就需要使用“-”号做范围限定，写成[A-Z]，
# 要匹配所有字母则写成[A-Za-z]。
# 一定要注意，这里“-”的作用不是充当一个字符。
[a-zA-Z]

# “^”符号出现在[]中，则代表取反
[^a-z] 

# 手机号码 （3移动8联调）
^1[38][0-9]\{9\} 
```

7. “ \”转义符号
```sh
"\\"  # 对\符号进行转义
"\.*" # 匹配任意长度的点号
```

8. “\<”符号和“\>”符号
界定单词的左边界和右边界
```sh
echo "hello" | grep '\<hello\>'  #匹配成功
echo "hellod" | grep '\<hello\>' # 没有输出， 匹配失败
```

9. “ \d”符号
```sh
echo 123 | grep [0-9] # 123

echo 123 | grep '\d' # 匹配不成功

# '\d'是一种perf兼容模式表达式， 要想使用这种模式得加上-P参数
echo 123 | grep -P '\d'
```

10. “ \b”符号
匹配单词的边界，比如“\bhello\b”可精确匹配“hello”单词
```sh
echo 'hello world' | grep '\bhello\b' # hello world
echo 'helloworld' | grep '\bhello\b' # 无匹配
```

11. “ \B”符号
匹配非单词的边界，比如hello\B可以匹配“helloworld”中的“hello”。

12. “ \w”符号
匹配字母、数字和下划线，等价于[A-Za-z0-9]。

13. “ \W”符号
匹配非字母、非数字、非下划线，等价于[^A-Za-z0-9]。

14. “ \s”符号
匹配任何空白字符

15. “ \S”符号
匹配任何非空白字符

16. “ \n”符号
匹配一个换行符（就是另起一新行，光标在新行的开头）。
*我们平时编写文件的回车符(即：回车键 )应该确切来说叫做回车式的换行符。*

17. “ \r”符号
匹配一个回车符（就是光标回到一旧行的开头，即光标目前所在的行为旧行）。

18. “ \t”符号
匹配一个制表符。

19. 扩展正则表达式（需要使用egrep命令）
```sh
# “?”符号用于匹配前一个字符0次或1次，所以“ro?t”仅能匹配rot或rt。
# “+”符号用于匹配前一个字符1次以上，所以“ro+t”就可以匹配rot、root等。
# “|”符号是“或”的意思，即多种可能的罗列，彼此间是一种分支关系
# “()”符号通常需要和“|”符号联合使用，用于枚举一系列可替换的字符
```
20. 通配符
```sh
# * 符号: 这里的“*”就是提到的第一个通配符，代表0个或多个字符。那么之前的*.doc就是指所有以.doc结尾的文件
ls -l *.doc 
ls -l A*.doc # 找doc文档是以字母A开头

# ？符号： 如果要列出以字母A开头、但是只有两个字母的文件名、以.doc结尾的文件，就需要使用“?”了
ls -l A?.doc

# {} 符号: “{}”可拥有匹配所有括号内包含的以逗号隔开的字符
ls -l {A, B, C}.doc  # 相等于 ls -l [A-C].doc


# ^和！符号： 这两个符号往往和“[]”一起使用，当出现在“[]”中的时候，代表取反。所以[^A]（或[!A]）代表不是A。

```

### 正则表达式示例
#### grep
grep的英文是Global search Regular Expression and print out the line，即全面搜索正则表达式并打印出匹配行
```sh
# 使用“^”匹配行首
grep '^good' regexp.txt # good morning teacher

# 使用“^$”组合，匹配空行
grep -c '^$' regexp.txt # 2

# 使用“.”号匹配任意字符
grep 'g..d' regexp.txt # good gold golden
grep '[Gg]..d' regexp.txt # good Good g12d

# 使用精确匹配
grep 'gold' regexp.txt # gold golden
grep '\<gold\>' regexp.txt # gold
grep '\bgold\b' regexp.txt # gold

# 使用“-”号
grep 'g[1-9]d' regexp.txt # g1d g2d

# 使用“*”号
grep 'go*d' regexp.txt # gd god good goood

# 使用“.*”号
grep 'g.*d' regexp.txt # gd gad good goood

# 使用“\”做字符转义, 下面.符号解析成正则任意字符
grep 'www.helloworld.com' regexp.txt 
grep 'www\.helloworld\.com' regexp.txt 
```

### 文本处理工具sed
#### sed介绍
sed（stream editor）是一种非交互式的流编辑器，通过多种转换修改流经它的文本。但是请注意，默认情况下，sed并不会改变原文件本身，而只是对流经sed命令的文本进行修改，并将修改后的结果打印到标准输出中（也就是屏幕）。所以本节讲的所有的sed操作都只是对“流”的操作，并不会改变原文件。sed处理文本时是以行为单位的，每处理完一行就立即打印出来，然后再处理下一行，直至全文处理结束。sed可做的编辑动作包括删除、查找替换、添加、插入、从其他文件中读入数据等。
**使用场景：常规编辑器编辑困难的文本；sed默认不更改源文件，如果想直接修改源文件本身则需要使用“-i”参数**
#### 删除
```sh
# 删除第一行然后输出到屏幕
sed '1d' sed.txt

# 删除第一行, 保存修改后的文件
sed '1d' sed.txt > save_file

# 删除第一行. 直接修改文件使用i参数；这里不会有任何输出， 而是直接修改了源文件
sed -i '1d' sed.txt

# 删除指定范围的行
sed '1, 3d' sed.txt
sed '1, $d' sed.txt # 删除1到最后一行

# 删除空行
sed '/^$/d' sed.txt

# 删除包含empty的行
sed '/empty/d' sed.txt
```

#### 查找替换
使用s命令可将查找到的匹配文本内容替换为新的文本
```sh
# 默认情况下每行只替换第一个line
sed 's/line/LINE/' sed.txt # this is LINE 1, this is First line, line

# 想要每行替换2个line
sed 's/line/LINE/2' sed.txt # this is LINE 1, this is First LINE, line

# 利用g选项， 完成所欲匹配值的替换
sed 's/line/LINE/g' sed.txt # this is LINE 1, this is First LINE, LINE

# 只替换开头的this为that
sed 's/^this/that/' sed.txt # that is line 1, this is First line, line
```

#### 字符转换
使用y命令可进行字符转换，其作用为将一系列字符逐个地变换为另外一系列字符
```sh
# 将file文件的O替换N, L替换E, D替换成W
sed 'y/OLD/NEW' file 

sed 'y/12345/ABCD' sed.txt
# this is line A, this is First line
# this is line N, this is Second line
# this is line C, this is Third line
```
#### 插入文本
使用i或a命令插入文本，其中i代表在匹配行之前插入，而a代表在匹配行之后插入
```sh
# 使用i在第二行前插入文本
sed '2 i Insert' sed.txt
# this is line 1, this is First line
# Insert
# this is line 2, this is Second line

# 使用a在第二行后插入文本
sed '2 a Insert' sed.txt
# this is line 1, this is First line
# this is line 2, this is Second line
# Insert


# 在匹配行的上一行插入文本
sed '/Second/i\Insert' sed.txt
# this is line 1, this is First line
# Insert
# this is line 2, this is Second line
```

#### 读入文本
使用r命令可从其他文件中读取文本，并插入匹配行之后
```sh
# 将/etc/passwd中的内容读出来放到sed.txt空行之后
sed '/^$/r /etc/passwd' sed.txt
```

#### 打印
使用p命令可进行打印，这里使用sed命令时一定要加-n参数，表示不打印没关系的行
```sh
# 使用p命令， 则只打印实际处理过的行，简化了输出（-n）
sed -n 's/the/THE/p' sed.txt
```
#### sed脚本
工作往往有一定“标准化”的操作，比如说先去除文件中所有的空行，然后再全部替换某些字符等，这种过程类似于生产线上程式化的流水作业；可以把这些动作静态化地写到某个文件中，然后调用sed命令并使用-f参数指定该文件，这样就可以将这一系列动作“装载”并应用于指定文件中，这无疑加快了工作效率，这种文件就是sed脚本。
```sh
# this替换THAT, 然后删除空行
# cat sed.rules
s/this/THAT/g
/^$/d

# 使用-f参数指定该脚本并应用到sed.txt
sed -f sed.rules sed.txt
```

### 文本处理工具awk
#### 介绍
sed其实是以行为单位的文本处理工具，而awk则是基于列的文本处理工具，它的工作方式是按行读取文本并视为一条记录，每条记录以字段分割成若干字段，然后输出各字段的值。

#### 打印指定域
awk使用$1、$2代表不同的域，则可以打印指定域
```sh
# 只打印部分
awk '{print $1, $4}' awk.txt
# zhao 021-1111111
# liu  021-2222222

# 打印全部
awk '{print $0}' awk.txt
# zhao male   30 021-1111111
# liu  female 31 021-2222222
# hu   female 31 021-3333333 shanghai
```

#### 指定打印分隔符
默认情况下awk是使用空白字符作为分隔符的，但是也可以通过-F参数指定分隔符，来区分不同的域（有点像之前学过的cut命令）
```sh
# 用.作为分隔符，这样每一行$1就是.之前的字符， $2就是.之后的字符
awk -F. '{print $1, $2}' awk.txt
```
#### 截取字符串
可以使用substr()函数对指定域截取字符串。substr(指定域,第一个开始字符的位置,第二个结束的位置)
#其中第二个结束的位置可以为空，这样默认输出到该域的最后一个字符。
```sh
cat awk.txt | awk '{print substr($1, 6)}'
```

#### 确定字符串的长度
```sh
cat awk.txt | awk '{print length}'
```
## shell编程概述和编程基础
### shell介绍
> shell是指一种命令行解释器，是为用户和操作系统之间通信提供的一种接口
> 脚本语言又被称作解释型语言，这种语言经过编写后不需要做任何编译就可以运行。 
> 计算机不能理解高级语言，只能理解机器语言，所以必须把高级语言翻译为机器码。而这种翻译的方式有两类，一类是编译，一类是解释，不同之处在于翻译的时间不同。编译型语言是运行前翻译，一般是使用编译工具将程序源码处理成机器认识的可执行文件（比如说Windows下的exe文件，Linux下的二进制可执行性文件），这种文件一旦产生，以后运行时将不需要再次翻译，所以一般来说，编译型语言的效率较高；而解释型语言是运行时翻译，执行一条语句就立即翻译一条，而且每次执行程序都需要进行解释，相对来说效率较低。但是也不能简单地认为编译型语言就一定比解释型效率高，随着解释器的发展，部分解释器能在运行程序时动态优化代码，因此这种效率差距也在一定程度上不断减小。


### shell脚本
#### 编写第一个shell脚本
一个Shell脚本永远是以“#!”开头的，这是一个脚本开始的标记，它是在告诉系统执行这个文件需要使用某个解释器，后面的/bin/bash就是指明了解释器的具体位置
```sh
#! /bin/bash
echo 'hello world'
```
#### 运行脚本
1. 第一种就是在该脚本所在的目录中直接bash这个脚本。实际上，如果使用这种方式来运行脚本，该脚本中的第一行“#!/bin/bash”就可以不需要了，因为直接bash一个文件就是指定了使用BashShell来解释脚本内容。
```sh
bash test.sh
```
2. 第二种方式是给该脚本加上可执行权限，然后使用“./”来运行，它代表运行的是当前目录下的HelloWorld.sh脚本，如果采用这种方式而脚本没有可执行权限则会报错。
```sh
chmod +x test.sh
./test.sh
```
3. 如果希望该脚本能成为默认的系统命令，简单地将该脚本复制到任一系统$PATH变量所包含的目录中，同时赋予可执行权限，下次运行的时候只需要直接输入该命令即可
```sh
chmod +x text.sh
mv test.sh /bin/
test.sh
```

#### shell脚本排错
为了更清晰地看到脚本运行的过程，还可以借助-x参数来观察脚本的运行情况。
```sh
bash -x test.sh
```
### shell内置命令
> 所谓Shell内建命令，就是由Bash自身提供的命令，而不是文件系统中的某个可执行文件。
通常来说，内建命令会比外部命令执行得更快，执行外部命令时不但会触发磁盘I/O，还需要fork出一个单独的进程来执行，执行完成后再退出。而执行内建命令相当于调用当前Shell进程的一个函数。

#### 如何确定内建命令：type
```sh
type cd 
# cd is shell builtin

type ifconfig
# ifconfig is /sbin/ifconfig
```

#### 执行程序：“.”（点号）
点号用于执行某个脚本，甚至脚本没有可执行权限也可以运行
```sh
./test.sh
# 提示无权限

. ./test.sh
# 执行成功

source test.sh
# 执行成功， 同时返回脚本中最后一个命令的返回状态； 如果没有返回值则返回0，代表执行成功； 如果未找到指定的脚本则返回false
```
#### 别名：alias
alias可用于创建命令的别名，若直接输入该命令且不带任何参数，则列出当前用户使用了别名的命令。现在你应该能理解类似ll这样的命令为什么与ls-l的效果是一样的吧
```sh
# 这样定义alias只能在当前Shell环境中有效，
# 换句话说，重新登录后这个别名就消失了。
# 为了确保永远生效，可以将该条目写到用户家目录中的.bashrc文件中
alias myShutdown='shutdown -h now'
```
#### 任务前后台切换：bg、fg、jobs
该命令用于将任务放置后台运行，一般会与Ctrl+z、fg、&符号联合使用。典型的使用场景是运行比较耗时的任务。比如打包某个占用较大空间的目录，若在前台执行，在任务完成前将会一直占用当前的终端，而导致无法执行其他任务，此时就应该将这类任务放置后台

## 测试和判断

## 循环

## 函数

## 重定向

## shell脚本范例