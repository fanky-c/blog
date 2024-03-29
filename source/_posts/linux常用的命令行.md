---
title: linux常用的命令行
date: 2021-07-09 18:17:51
tags:
    - linux命令行
    - linux操作
---

### 系统信息
1. df -h  容易阅读的方式显示磁盘分区使用情况
2. df /etc/dhcp  显示指定文件所在分区的磁盘使用情况

### 文件和目录
1. rm -rf dir 删除一个叫做 'dir' 的目录并同时删除其内容 
2. mv dir new_dir 重命名/移动 一个目录 
3. cp file1 file2 复制一个文件
4. cp /dir/* . 复制一个**目录下的所有文件**到当前工作目录 
5. cp /dir . 复制一个目录到当前目录
6. tree 显示文件和目录由根目录开始的树形结构
7. mkdir -p dir 创建目录
8. ls -al 显示文件(隐藏文件)和目录的详细资料 

### 文件搜索
1. find / -name file1 从 '/' 开始进入根文件系统搜索文件和目录
2. find /home/user1 -name \*.bin 在目录 '/home/user1' 中搜索带有'.bin' 结尾的文件
3. find /usr/bin -type f -mtime -10 搜索在10天内被创建或者修改过的文件 

### 文件的权限
1. ls -lh 显示权限
2. chown -R user directory 改变一个目录的所有人属性并同时改变改目录下所有文件的属性
3. chown user file 改变一个文件的所有人属性
4. chmod ugo+rwx directory 设置目录的所有人(u)、群组(g)以及其他人(o)以读（r ）、写(w)和执行(x)的权限 

### 查看文件内容
1. cat file 从第一个字节开始正向查看文件的内容 
2. tac file 从最后一行开始反向查看一个文件的内容
3. more file 查看一个长文件的内容 
4. tail -f /var/log/messages 实时查看被添加到一个文件中的内容 
5. tail -2 file 查看一个文件的最后两行 
6. head -2 file 查看一个文件的前两行

### 文本处理
1. grep Aug /var/log/messages 在文件 '/var/log/messages'中查找关键词"Aug"
2. grep ^Aug /var/log/messages 在文件 '/var/log/messages'中查找以"Aug"开始的词汇
3. sed 's/stringa1/stringa2/g' example.txt 将example.txt文件中的 "string1" 替换成 "string2"
4. sed -n '/stringa1/p' 查看只包含词汇 "string1"的行    

### 文件传输
1. curl https://baidu.com   curl命令是一个利用URL规则在shell终端命令行下工作的文件传输工具；它支持文件的上传和下载，所以是综合传输工具，但按传统，习惯称curl为下载工具。
2. tftp 218.28.188.288   ftp命令用于传输文件


### 打包压缩
1. zip file1.zip file1 创建一个zip格式的压缩包 
2. zip -r file1.zip file1 file2 dir1 将几个文件和目录同时压缩成一个zip格式的压缩包
3. unzip file1.zip 解压一个zip格式压缩包 

### 网络
1. ssh 202.102.240.88  安全连接客户端, 可以给予ssh加密协议实现安全的远程登录服务器，实现对服务器的远程管理。
2. ping baidu.com/202.102.240.88     测试主机间网络连通性
3. netstat -a          显示网络状态
4. ipconfig            显示或设置网络设备

### 进程
1. ps -a | grep helloworld 查看特定进程  
2. ps -ef  显示所有进程信息，连同命令行
3. kill -9 pid 杀掉进程
4. lsof -i:端口号  查看端口号被那个进程所占用


### 软链接 和 硬链接
1. Linux 链接分两种，一种被称为硬链接（Hard Link），另一种被称为符号链接（Symbolic Link）。默认情况下，ln 命令产生硬链接
2. 验证硬链接和软链接关系
```bash
touch f1          #创建一个测试文件f1    
ln f1 f2          #创建f1的一个硬连接文件f2
ln -s f1 f3       #创建f1的一个符号连接文件f3
ls -li            # -i参数显示文件的inode节点信息
total 0
9797648 -rw-r--r--  2 oracle oinstall 0 Apr 21 08:11 f1
9797648 -rw-r--r--  2 oracle oinstall 0 Apr 21 08:11 f2
9797649 lrwxrwxrwx  1 oracle oinstall 2 Apr 21 08:11 f3 -> f1

echo "I am f1 file" >>f1
cat f1
I am f1 file
cat f2
I am f1 file
cat f3
I am f1 file
rm -f f1
cat f2
I am f1 file
cat f3
cat: f3: No such file or directory
```
3. 结论：
  1. 当删除原始文件 f1 后，硬连接 f2 不受影响，但是符号连接 f3 文件无效
  2. 删除符号连接f3,对f1,f2无影响；
  3. 删除硬连接f2，对f1,f3也无影响；
  4. 删除原文件f1，对硬连接f2没有影响，导致符号连接f3失效；
  5. 同时删除原文件f1,硬连接f2，整个文件会真正的被删除。 



[资料来源于](https://www.cnblogs.com/fnlingnzb-learner/p/5831284.html)