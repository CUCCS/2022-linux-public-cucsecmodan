# linux实验作业chap0x03

-----

作者：cucsecmodan

----
## 实验要求
1.完成 Systemd 入门教程练习并将完整操作过程上传 asciinema， 文档上传 GitHub
2.自查清单
* 如何添加一个用户并使其具备sudo执行程序的权限？
* 如何将一个用户添加到一个用户组？
* 如何查看当前系统的分区表和文件系统详细信息？
* 如何实现开机自动挂载Virtualbox的共享目录分区？
* 基于LVM（逻辑分卷管理）的分区如何实现动态扩容和缩减容量？
* 如何通过systemd设置实现在网络连通时运行一个指定脚本，在网络断开时运行另一个脚本？
* 如何通过systemd设置实现一个脚本在任何情况下被杀死之后会立即重新启动？实现杀不死？ 

----
## 实验环境
宿主机：win 11
虚拟机：unbuntu20.04

------

## 一.完成 Systemd 入门教程练习并将完整操作过程上传 asciinema， 文档上传 GitHub
1.查看 `Systemd` 的版本:
```bash
$ systemctl --version
```

![](img\systemd--version.png)


2.`systemd-analyze` 命令查看启动耗时。
```bash
# 查看启动耗时
$ systemd-analyze                                                                                       

# 查看每个服务的启动耗时
$ systemd-analyze blame

# 显示瀑布状的启动过程流
$ systemd-analyze critical-chain

# 显示指定服务的启动流
$ systemd-analyze critical-chain atd.service
```
录屏地址：
[![](img\systemd-analyze.png)](https://asciinema.org/a/485762)
3.`hostnamectl`命令用于查看当前主机的信息:
```bash
# 显示当前主机的信息
$ hostnamectl

# 设置主机名。
$ sudo hostnamectl set-hostname rhel7
```
录屏地址：
[![](img\hostnamectl_asciinema.png)](https://asciinema.org/a/485765)
4.`localectl`命令用于查看本地化设置:
```bash

# 查看本地化设置
$ localectl

# 设置本地化参数。
$ sudo localectl set-locale LANG=en_GB.utf8
$ sudo localectl set-keymap en_GB
```
录屏地址：
[![](img\localectl_asciinema.png)](https://asciinema.org/a/485769)

5.`timedatectl` 命令用于查看当前时区设置。
```bash

# 查看当前时区设置
$ timedatectl

# 显示所有可用的时区
$ timedatectl list-timezones                                                                                   

# 设置当前时区
$ sudo timedatectl set-timezone America/New_York
$ sudo timedatectl set-time YYYY-MM-DD
$ sudo timedatectl set-time HH:MM:SS
```
录屏地址：
[![](img\timedate_asciinema.png)](https://asciinema.org/a/485770)
6.`loginctl` 命令用于查看当前登录的用户。
```bash
# 列出当前session
$ loginctl list-sessions

# 列出当前登录用户
$ loginctl list-users

# 列出显示指定用户的信息
$ loginctl show-user ruanyf
```
录屏地址：
[![](img\loginctl_asciinema.png)](https://asciinema.org/a/485773)
7.`systemctl list-units` 命令可以查看当前系统的所有 Unit
```bash

# 列出正在运行的 Unit
$ systemctl list-units

# 列出所有Unit，包括没有找到配置文件的或者启动失败的
$ systemctl list-units --all

# 列出所有没有运行的 Unit
$ systemctl list-units --all --state=inactive

# 列出所有加载失败的 Unit
$ systemctl list-units --failed

# 列出所有正在运行的、类型为 service 的 Unit
$ systemctl list-units --type=service
```
录屏地址：
[![](img\systemctllist-units_asciinema.png)](https://asciinema.org/a/485775)
8.`systemctl status` 命令用于查看系统状态和单个 Unit 的状态:
```bash
# 显示系统状态
$ systemctl status

# 显示单个 Unit 的状态
$ sysystemctl status bluetooth.service

#以下unit选accounts-daemon.service为例
# 显示某个 Unit 是否正在运行
$ systemctl is-active accounts-daemon.service

# 显示某个 Unit 是否处于启动失败状态
$ systemctl is-failed accounts-daemon.service

# 显示某个 Unit 服务是否建立了启动链接
$ systemctl is-enabled accounts-daemon.service
```
[![](img\systemctlstatus_asciinema.png)](https://asciinema.org/a/485785)
9.`systemctl list-dependencies` 命令列出一个 Unit 的所有依赖
```bash

$ systemctl list-dependencies nginx.service
#展开 Target
$ systemctl list-dependencies --all nginx.service
```
录屏地址：
[![](img\systemctllist-dependencies_asciinema.png)](https://asciinema.org/a/485789)
10.`systemctl list-unit-files` 命令用于列出所有配置文件:
```bash
# 列出所有配置文件
$ systemctl list-unit-files

# 列出指定类型的配置文件
$ systemctl list-unit-files --type=service
```
录屏地址：
[![](img\systemctllist-unit-files_asciinema.png)](https://asciinema.org/a/485790)

11.target管理：
```bash

# 查看当前系统的所有 Target
$ systemctl list-unit-files --type=target

# 查看一个 Target 包含的所有 Unit
$ systemctl list-dependencies multi-user.target

# 查看启动时的默认 Target
$ systemctl get-default

# 设置启动时的默认 Target
$ sudo systemctl set-default multi-user.target

# 切换 Target 时，默认不关闭前一个 Target 启动的进程，
# systemctl isolate 命令改变这种行为，
# 关闭前一个 Target 里面所有不属于后一个 Target 的进程
$ sudo systemctl isolate multi-user.target
```
录屏地址：
[![](img\target_asciinema.png)](https://asciinema.org/a/485792)

12.日志管理
```bash
#查看所有日志（默认情况下 ，只保存本次启动的日志）
$ sudo journalctl

# 查看内核日志（不显示应用日志）
$ sudo journalctl -k

# 查看系统本次启动的日志
$ sudo journalctl -b
$ sudo journalctl -b -0

# 查看上一次启动的日志（需更改设置）
$ sudo journalctl -b -1

# 查看指定时间的日志
$ sudo journalctl --since="2012-10-30 18:17:16"
$ sudo journalctl --since "20 min ago"
$ sudo journalctl --since yesterday
$ sudo journalctl --since "2015-01-10" --until "2015-01-11 03:00"
$ sudo journalctl --since 09:00 --until "1 hour ago"

# 显示尾部的最新10行日志
$ sudo journalctl -n

# 显示尾部指定行数的日志
$ sudo journalctl -n 20

# 实时滚动显示最新日志
$ sudo journalctl -f

# 查看指定服务的日志
$ sudo journalctl /usr/lib/systemd/systemd

# 查看指定进程的日志
$ sudo journalctl _PID=1

# 查看某个路径的脚本的日志
$ sudo journalctl /usr/bin/bash

# 查看指定用户的日志
$ sudo journalctl _UID=33 --since today

# 查看某个 Unit 的日志
$ sudo journalctl -u nginx.service
$ sudo journalctl -u nginx.service --since today

# 实时滚动显示某个 Unit 的最新日志
$ sudo journalctl -u nginx.service -f

# 合并显示多个 Unit 的日志
$ journalctl -u nginx.service -u php-fpm.service --since today

# 查看指定优先级（及其以上级别）的日志，共有8级
# 0: emerg
# 1: alert
# 2: crit
# 3: err
# 4: warning
# 5: notice
# 6: info
# 7: debug
$ sudo journalctl -p err -b

# 日志默认分页输出，--no-pager 改为正常的标准输出
$ sudo journalctl --no-pager

# 以 JSON 格式（单行）输出
$ sudo journalctl -b -u nginx.service -o json

# 以 JSON 格式（多行）输出，可读性更好
$ sudo journalctl -b -u nginx.service -o json-pretty

# 显示日志占据的硬盘空间
$ sudo journalctl --disk-usage

# 指定日志文件占据的最大空间
$ sudo journalctl --vacuum-size=1G

# 指定日志文件保存多久
$ sudo journalctl --vacuum-time=1years
```
录屏地址：
[![](img\journalctl_asciinema.png)](https://asciinema.org/a/485883)

12.开机启动
```bash
$ sudo systemctl enable networktest.service
```
录屏地址：
[![](img\systemctl_enable_asciinema.png)](https://asciinema.org/a/485884)

13.启动服务和停止服务
```bash
$ sudo systemctl start networktest.service #启动服务
$ sudo systemctl stop networktest.service #停止服务
$ sudo systemctl kill networktest.service #如果停止服务不成功，可以杀进程

$ sudo systemctl restart httpd.service #重启服务
```
13.默认启动
```bash
$ systemctl get-default # 启动 Target 是graphical.target。在这个组里的所有服务，都将开机启动

$ systemctl list-dependencies multi-user.target # 查看 multi-user.target 包含的所有服务

$ sudo systemctl isolate shutdown.target # shutdown.target 就是关机状态
```
![](img\systemgetdefault.png)

![](img\systemctllistdependence.png)


14.修改配置文件后重启
```bash
# 重新加载配置文件
$ sudo systemctl daemon-reload

# 重启相关服务
$ sudo systemctl restart networktest.service
```

-----
## 二.自查清单
---
### 1.如何添加一个用户并使其具备sudo执行程序的权限？
新建用户 `lily`
```bash
~$ sudo adduser lily
```
给用户增加 `sudo` 权限
```bash
~$ sudo usermod -a -G adm lily
~$ sudo usermod -a -G sudo lily
```
录屏地址：
[![](img\adduser&sudo_ubuntu.png)](https://asciinema.org/a/482316)
### 2.如何将一个用户添加到一个用户组？
查看用户组
```bash
~$ cat /ect/group
```
创建新的用户组：
```bash
~$ sudo adddroup mygroup
```
把用户添加到用户组：
```bash
~$ sudo gpasswd -a lily mygroup
```
再次查看用户组：
```bash
~$ cat /ect/group
```
删除用户组：
```bash
~$ sudo groupdel mygroup
```
录屏地址：
[![](img\addgroup_ubuntu.png)](https://asciinema.org/a/482589)

---
### 3.如何查看当前系统的分区表和文件系统详细信息？
查看所有系统分区表：
```bash
~$ sudo fdisk -l
```
![](img\fdisk.png)
也可以进入到某个分区(/dev/sda)进行查看：
```bash
~$ lsblk #查看块区
~$ sudo fdisk /dev/sda #进入sda盘分区
m #查看帮助
p #查看分区表和详细信息
```
![](img\fdisk_sda.png)

查看磁盘管理信息：
```bash
~$ df -h
```
![](img\df.png)

### 4.如何实现开机自动挂载Virtualbox的共享目录分区？
在虚拟机中的共享文件夹设置中设置好固定分配目录：
![](img\set_fixedshareplace.png)
在虚拟机上创建共享文件目录：
```bash
~$ sudo mkdir /mnt/share
```
实现挂载：
```bash
~$ mount -t vboxsf Share_vbox /mnt/share
~$ cd /mnt/share
~$ ls
```
![](img\share_file.png)
关机重新连接，发现共享文件夹里的文件没有出现：
![](img\afterpoweroff.png)

实现开机自动挂载：
```bash
切换到root用户
~$ sudo su
#更改配置文件
~$ vi /etc/fstab
#Share_vbox是Windows上的共享文件夹，/mnt/share是Ubuntu上的共享文件夹
~$ Share_vbox /mnt/share vboxsf rw,gid=cuc,uid=cuc,auto 0 0 
```
![](img\auto_mountshare.png)
重启虚拟机：
```bash
poweroff
```
再次查看：
```bash
~$ cd /mnt/share
~$ ls
```
![](img\auto_mountshare_file.png)

### 5.基于LVM（逻辑分卷管理）的分区如何实现动态扩容和缩减容量？
![](img\LVM_struction.png)
初始化硬盘：先关闭正在运行的虚拟机，在硬盘设置SATA下新增硬盘
普通磁盘管理：
```bash
~$ lsblk #查看硬盘信息
~$ sudo fdisk /dev/sdb #创建硬盘分区
m #查看帮助
p #查看分区表和详细信息
n #创建新的分区
w #保存和退出
~$ sudo mkfs.ext4 /dev/sdb1 #创建文件系统 mkfs -t ext4 /dev/sdb1
~$ mkdir -p /mnt/sdc1 #挂载磁盘
~$ sudo mount /dev/sdc1 /mnt/sdc1
```
* LVM磁盘管理：
创建PV分区：
```bash
pvcreate /dev/sdb{1,2,3} #创建PV分区
pvs #查看PV分区信息
pvscan #查看PV分区信息
```
![](img\pvcreate.png)
![](img\vgcreate_sdc.png)
创建VG分区：
```bash
vgcreate test-vg /dev/sdb{1,2,3} #创建VG分区
vgs #查看VG分区
```
![](img\vgcreate.png)
扩展vg分区：
```bash
vgextend test-vg /dev/sdc{1,2}
```
![](img\vgextend.png)
创建LV分区：
```bash lvcreate -L 10G -n test-lv-1 test-vg #创建LV分区
lvdisplay #查看LV信息
lvs
vgs
lvcreate -l 100%FREE -n test-lv-2 test-vg #将剩余空间全部用于创建LV2
```
为LV分区创建文件系统：
```bash
mkfs.ext4 /dev/test-vg/test-lv-1
```
挂载LV分区：
```bash
mkdir /mnt/test-lv-1
mount /dev/test-vg/test-lv-1 /mnt/test-lv-1
```
![](img\mount_lv.png)
动态扩容和缩减容量:
```bash
lvresize --size +2G --resizefs /dev/test-vg/test-lv-3 #扩容
lvresize --size -2G --resizefs /dev/test-vg/test-lv-3 #减容
```
录屏地址：
[![](img\lvresize_asciinema.png)](https://asciinema.org/a/484868)
### 6.如何通过systemd设置实现在网络连通时运行一个指定脚本，在网络断开时运行另一个脚本？
创建脚本：
```bash
~$ cd /etc/systemd/system
~$ sudo vi networktest.service
```
写脚本：
```bash
[Unit]
Description=networktest
#网络检测依赖于 network-online.target
After=network-online.target

[Service]
ExecStart=/bin/echo network-ok# 联网时输出ok
ExecStop=/bin/echo network-error# 断网时输出error
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target                         
```
让Systemd 重新读取所有的 Unit 文件：
```bash
~$ sudo systemctl daemon-reload
```
建立服务的符号链接：
```bash
~$ sudo systemctl enable networktest.service
```
使用 `journalctl` 查看新创建的service的日志：
```bash
~$ sudo systemctl stop networktest.service # 关闭网络服务
~$ sudo journalctl -u networktest.service
```
```bash
~$ sudo systemctl stop networktest.service # 开启网络服务
~$ sudo journalctl -u networktest.service
```
录屏地址：
[![](img\system_unit.png)](https://asciinema.org/a/485481)

### 7.如何通过systemd设置实现一个脚本在任何情况下被杀死之后会立即重新启动？实现杀不死？ 
创建一个不可杀死的脚本：
```bash
~$ cd /etc/systemd/system
~$ sudo vi stayalive.service
```
写脚本：
```bash
[Unit]

[Service]
ExecStart=/bin/echo Alive 
ExecStop=/bin/echo Dead
Restart=always #定义何种情况 Systemd 会自动重启当前服务，可能的值包括 always（总是重启）、on-success、on-failure、on-abnormal、on-abort、on-watchdog
RestartSec=3 #自动重启当前服务间隔的秒数

[Install]
WantedBy=multi-user.target                         
```
查看日志：
```bash
sudo systemctl status stayalive.service #显示单个unit的状态
sudo systemctl start stayalive.service #立即启动一个服务
sudo journalctl -u stayalive.service 
```

-----

## 参考文献
[Ubuntu20.04增加、删除、查看用户及给用户root权限，及防火墙常用操作](https://blog.csdn.net/UCB001/article/details/116277337?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_ecpm_v1~rank_v31_ecpm-1-116277337.pc_agg_new_rank&utm_term=ubuntu20%E6%9F%A5%E7%9C%8B%E7%94%A8%E6%88%B7&spm=1000.2123.3001.4430) 
[ubuntu20.04 创建新的用户,并添加到用户组](https://blog.csdn.net/qq_41166909/article/details/121735361)
[Virtualbox虚拟机Ubuntu系统设置共享文件夹及自动挂载](https://blog.csdn.net/dahuzix/article/details/80020934?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_ecpm_v1~rank_v31_ecpm-14-80020934.pc_agg_new_rank&utm_term=virtualbox%E6%8C%82%E8%BD%BDubuntu&spm=1000.2123.3001.4430)
[可能是史上最全面易懂的 Systemd 服务管理教程！( 强烈建议收藏 )](https://cloud.tencent.com/developer/article/1516125)