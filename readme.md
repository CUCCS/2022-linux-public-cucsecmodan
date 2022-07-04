# chap0x05

-----

## 实验环境
虚拟机：Ubuntu 20.04 宿主机：Microsoft Windows 11

------

## 实验要求
### 基本要求
* 在一台主机（虚拟机）上同时配置Nginx和VeryNginx
    * VeryNginx作为本次实验的Web App的反向代理服务器和WAF
    * PHP-FPM进程的反向代理配置在nginx服务器上，VeryNginx服务器不直接配置Web站点服务
* 使用Wordpress搭建的站点对外提供访问的地址为： http://wp.sec.cuc.edu.cn
* 使用Damn Vulnerable Web Application (DVWA)搭建的站点对外提供访问的地址为： http://dvwa.sec.cuc.edu.cn

### 安全加固要求
* 使用IP地址方式均无法访问上述任意站点，并向访客展示自定义的友好错误提示信息页面-1
* Damn Vulnerable Web Application (DVWA)只允许白名单上的访客来源IP，其他来源的IP访问均向访客展示自定义的友好错误提示信息页面-2
* 在不升级Wordpress版本的情况下，通过定制VeryNginx的访问控制策略规则，热修复WordPress < 4.7.1 - Username Enumeration
* 通过配置VeryNginx的Filter规则实现对Damn Vulnerable Web Application (DVWA)的SQL注入实验在低安全等级条件下进行防护

### VeryNginx配置
* VeryNginx的Web管理页面仅允许白名单上的访客来源IP，其他来源的IP访问均向访客展示自定义的友好错误提示信息页面-3
* 通过定制VeryNginx的访问控制策略规则实现：
    限制DVWA站点的单IP访问速率为每秒请求数 < 50
    限制Wordpress站点的单IP访问速率为每秒请求数 < 20
    超过访问频率限制的请求直接返回自定义错误提示信息页面-4
    禁止curl访问

------

## 试验过程
### 1.安装 `nginx`
安装 `nginx` ：
```bash
sudo apt update 
sudo apt install nginx
```
启动 `nginx` ：
```bash
sudo systemctl start nginx
sudo systemctl status nginx
```
![](imgs/start_nginx.png)
访问 `192.168.56.101:80` ：
![](imgs/访问nginx.png)
说明已经成功安装，还需要进一步配置。

### 2.安装 `Verynginx`
#### 准备工作
克隆VeryNginx仓库到本地:
```bash
mkdir verynginx
cd verynginx
git clone https://github.com/alexazhou/VeryNginx.git
```
![](imgs/git_clone_verynginx.png) 
安装 `verynginx` 依赖的工具：
```bash
cd Verynginx
sudo apt install python
sudo apt install libssl-dev
#pcre
sudo apt install libpcre3-dev
#gcc
sudo apt install gcc
#make
sudo apt install make
#zlib
sudo apt-get install zlib1g-dev
```
安装 `verynginx` :
```bash
sudo python install.py install
```
![](imgs/install_verynginx.png)

为了防止verynginx和nginx的端口号重复了，修改配置文件里的端口：
```bash
cd /opt/verynginx/openresty/nginx/conf
sudo vi nginx.conf
```
修改前：
![](imgs/verynginx_orinal_port.png)
修改后：
![](imgs/verynginx_reset_port.png)
修改配置文件里的用户名：
![](imgs/改用户名.png)
更改用户名以后会导致无法登陆，换一个方式，增加一个默认的 `nginx` 用户：
![](imgs/添加nginx用户.png)
启动 `verynginx` :
```bash
sudo /opt/verynginx/openresty/nginx/sbin/nginx
```
访问 `192.168.56.101:8100/verynginx/index.html` :
![](imgs/访问veryinginx.png)

使用官方文档里的默认用户名和密码登录：`verynginx` / `verynginx` 。
![](imgs/登录verynginx.png)

#### 配置宿主机 `host`
在 `C:\Windows\System32\drivers\etc` 中找到 `host` 文件添加
```bash
# 测试网址
192.168.56.101 www.milkcandynginx.com www.milkcandyverynginx.com

# 目标搭建网址
192.168.56.101 wp.sec.cuc.edu.cn dvwa.sec.cuc.edu.cn
```

### 3.配置及搭建
#### 配置 `nginx`
创建服务器根目录 `milkcandynginx/php` :
```bash
mkdir milkcandynginx/php
```
创建 `index.html` 页面
```bash
cd milkcandynginx/php
vi index.html
# 以下内容在 index 中
<html>
<body>
    <h1>hello nginx</h1>
</body>
#
```
在 `/etc/nginx/conf.d` 目录下创建 `milkcandy.conf`:
```bash
sudo vi milkcandy.conf
```
配置 `milkcandy.conf` :


![](imgs/配置nginx_html.png)
在互联网上访问： `www.milkcandynginx.com/index.html` :
![](imgs/nginx_html.png)

配置 `php-fpm` 反向代理服务器：
下载 `php-fpm`：
```bash
sudo apt install php-fpm
```
更改端口号 `127.0.0.1:9000` ：
![](imgs/php_port_reset.png)
在 `~/milkcandy/php` 中创建 `index.php` :
```php
<?php
phpinfo();
?>
```
在 `/etc/nginx/conf.d/milkcandynginx.conf` 进行配置，配置和之前的一致:
![](imgs/配置nginx_html.png)


重启 `php-fpm` 和 `nginx`：
```bash
whereis php-fpm #查看php-fpm的具体位置以及版本
ps -ef | grep php #查看php进程是否在工作
sudo service php7.4-fpm restart #我的php-fpm版本是7.4


sudo systemctl restart nginx #重启nginx

```
根据配置连接 `www.milkcandyverynginx.com/index.php` or `milkcandnginx.com/index.php`都可以访问界面:
![](imgs/php_1.png)
![](imgs/php_2.png)

### 在 `nginx` 上搭建 `Wordpress 4.7 `:
用 `scp` 将 `wordpress4.7-zip` 传输至虚拟机并且解压缩
```bash
#宿主机
scp wordpress-6.0.zip cuc@192.168.56.101:~/milkcandynginx/wordpress

#虚拟机
unzip wordpress-4.7.zip
```
在 `nginx` 中配置虚拟主机：
```bash
cd /etc/nginx/conf.d/
sudo vi wordpress.conf

# in file
server{
        listen 192.168.56.101:8100;
        server_name wp.sec.cuc.edu.cn;

        root /home/cuc/mynginx/wordpress/;

        location / {
                index index.html;
        }

        location ~ \.php$ {
                root /home/cuc/mynginx/wordpress/;
                include /etc/nginx/fastcgi.conf;
                fastcgi_intercept_errors on;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
                fastcgi_pass 127.0.0.1:9000;
        }

}

# 重新载入配置
sudo nginx -t
sudo nginx -s reload
```

#### 安装php及相关组件
```bash
sudo apt install php-fpm php-mysql php-curl php-gd php-intl php-mbstring php-soap php-xml php-xmlrpc php-zip
```

#### 下载mysql
```bash
sudo apt-get update && sudo apt-get dist-upgrade
sudo apt-get install mysql-server
```

按照 `readme.html` 中的步骤进行安装，对数据库进行配置
```sql
sudo mysql

create database wordpress;

create user wordpress@localhost identified by 'wordpress';

grant all on wordpress.* to wordpress@localhost;

flush privileges;

exit
```
访问 `wp.sec.cuc.edu.cn/wordpress/index.php` :
![](imgs/wordpress_php.png)
登录：
![](imgs/wordpress登录.png)

### 在 `nginx` 上搭建 `DVWA`
```bash
# 虚拟机
mkdir milkcandynginx/dvwa/
cd milkcandynginx/dvwa/

# 宿主机下载好 DVWA-master.zip
scp DVWA-master.zip cuc@192.168.56.101:~/milkcandynginx/dvwa

# 虚拟机
unzip DVWA-master.zip
```

配置 `DVWA`:
```bash
cd /etc/nginx/conf.d/
sudo vi dvwa.conf

# in file
server{
        listen 192.168.56.101:8100;
        server_name dvwa.sec.cuc.edu.cn;

        root /home/cuc/milkcandynginx/dvwa/DVWA-master;

        location / {
                index index.html;
        }

        location ~ \.php$ {
                root /home/cuc/milkcandynginx/dvwa/DVWA-master;
                include /etc/nginx/fastcgi.conf;
                fastcgi_intercept_errors on;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
                fastcgi_pass 127.0.0.1:9000;
        }
}

# 重新载入配置
sudo nginx -t
sudo nginx -s reload
```

重命名配置文件：
```bash
cd /milkcandynginx/dvwa/DVWA-master/config/
cp config.inc.php.dist config.inc.php
```
按照 `readme.md` 文件进行安装：
```sql
sudo mysql

create database dvwa;


create user dvwa@localhost identified by 'p@ssw0rd';

 grant all on dvwa.* to dvwa@localhost;
flush privileges;
```
访问:
![](imgs/访问dvwa1.png)
![](imgs/访问dvwa.png)

配置 `verynginx` 作为防火墙：
![](imgs/wp&dvwa.png)
![](imgs/wordpress&dvwa.png)


### 2.配置安全加固要求
1.使用IP地址方式均无法访问上述任意站点，并向访客展示自定义的友好错误提示信息页面-1
![](imgs/ip拦截.png)
![](imgs/ip拦截规则.png)
![](imgs/ip拦截相应.png)
![](imgs/ip拦截成功.png)

2.Damn Vulnerable Web Application (DVWA)只允许白名单上的访客来源IP，其他来源的IP访问均向访客展示自定义的友好错误提示信息页面-2
![](imgs/白名单.png)
![](imgs/白名单拦截规则.png)
![](imgs/白名单响应规则.png)
![](imgs/白名单测试.png)

3.在不升级Wordpress版本的情况下，通过定制VeryNginx的访问控制策略规则，热修复WordPress < 4.7.1 - Username Enumeration

4.通过配置VeryNginx的Filter规则实现对Damn Vulnerable Web Application (DVWA)的SQL注入实验在低安全等级条件下进行防护
![](imgs/attack_sql.png)
![](imgs/attack_sql拦截规则.png)
![](imgs/dvwa_low.png)


### 3.Verynginx配置要求
1.VeryNginx的Web管理页面仅允许白名单上的访客来源IP，其他来源的IP访问均向访客展示自定义的友好错误提示信息页面-3
![](imgs/vn_wl.png)
![](imgs/vn_wl拦截规则.png)
![](imgs/vn_wl响应规则.png)
![](imgs/vn_wl拦截成功.png)

2.超过访问频率限制的请求直接返回自定义错误提示信息页面-4
![](imgs/频率限制.png)
![](imgs/frequence响应规则.png)

3.禁止curl访问
![](imgs/curl.png)
![](imgs/curl拦截规则.png)
![](imgs/curl_响应规则.png)
![](imgs/curl测试成功.png)

-------------
## 参考文献
[nginx服务无法启动nginx: [emerg] getpwnam(“nginx”) failed](https://blog.csdn.net/weixin_42480196/article/details/100600274?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-100600274-blog-58601731.pc_relevant_multi_platform_whitelistv1&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-100600274-blog-58601731.pc_relevant_multi_platform_whitelistv1)
[邱纪霖同学的作业](https://github.com/CUCCS/2022-linux-public-807544076/tree/chap0x05/chap0x05)