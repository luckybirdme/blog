---
title: PHP-FPM
date: 2020-03-18 07:45:08
tags: PHP
---
> CGI, PHP-CGI, FastCGI, PHP-FPM

<!-- more -->


## 一. CGI，common gateway interface 
通用网关接口，定义Web服务器(nginx)与web应用程序(PHP)交互的协议，比如数据传递格式。

## 二. FastCGI
CGI协议的升级版，提供常驻进程的机制，以便快速处理请求，无需重复 fork-and-execute。

## 三. PHP-CGI
实现FastCGI接口的程序，用于接受NGINX转发的请求，按照CGI协议转换数据，并交给PHP脚本处理，然后把PHP脚本输出的数据再返回给NGINX。
当修改php.ini配置文件后，需要重启PHP-CGI才能生效，而杀掉PHP-CGI会导致PHP脚本无法继续执行。

![](http://qiniucdn.luckybird.me/blog/img/2020/PHP-CGI.png)


## 四. PHP-FPM，PHP FastCGI process master
实现FastCGI协议的进程管理器，master 是主进程，负责PHP公共环境初始化，并且创建和管理多个 PHP-FPM 进程。PHP-FPM进程直接处理NGINX转发过来的请求，它的数量可固定，也可以动态变化，以便适应不同场景的请求量。
当修改php.ini配置文件后，master 会在 PHP-FPM 进程处理完一个请求后，逐一重启，平滑过渡，不影响正常服务。

![](http://qiniucdn.luckybird.me/blog/img/2020/PHP-FPM.png)

![](http://qiniucdn.luckybird.me/blog/img/2020/PHP-FPM-two.png)

## 五. PHP-FPM 进程池
PHP-FPM 除了可动态设置进程数量外，还能设置多个进程池，既可以提高并发量，也能隔离不同请求。
通过 php-fpm.conf 设置进程池，以下是全局设置
```sh
;;;;;;;;;;;;;;;;;;
; Global Options ;
;;;;;;;;;;;;;;;;;;
[global]
; Pid file
; Default Value: none
pid = /run/php-fpm/php-fpm.pid

; Error log file
; Default Value: /var/log/php-fpm.log
error_log = /var/log/php-fpm/error.log

;;;;;;;;;;;;;;;;;;;;
; Pool Definitions ; 
;;;;;;;;;;;;;;;;;;;;

; See /etc/php-fpm.d/*.conf
```

在 /etc/php-fpm.d/ 目录下添加进程池配置文件
```sh
; www.conf 文件
; 进程池名称
[www]
; socket 名称
listen = /var/run/php-fpm/php-fpm-www.sock
; 进程用户
user = nginx
group = nginx
; 动态分配进程数量
pm = dynamic
pm.max_children = 50
pm.start_servers = 5


; http.conf 文件
; 进程池名称
[http]
; socket 名称
listen = /var/run/php-fpm/php-fpm-http.sock
; 进程用户
user = nginx
group = nginx
; 固定进程数量，取 max_children 的值
pm = static
pm.max_children = 50
pm.start_servers = 5

```


在 NGINX 配置文件 /etc/nginx/conf.d 添加 PHP-FPM 进程池的配置
```sh
# 隔离不同进程池
# vim /etc/nginx/conf.d/www.conf
location ~ \.php$         
{
     include fastcgi_params;
     fastcgi_pass unix:/var/run/php-fpm/php-fpm-www.sock;
     fastcgi_index index.php;
     fastcgi_param SCRIPT_FILENAME /data/www$fastcgi_script_name;
}

# 隔离不同进程池
# vim /etc/nginx/conf.d/http.conf
location ~ \.php$         
{
     include fastcgi_params;
     fastcgi_pass unix:/var/run/php-fpm/php-fpm-http.sock;
     fastcgi_index index.php;
     fastcgi_param SCRIPT_FILENAME /data/http$fastcgi_script_name;
}

# 进程池负载均衡
# vim /etc/nginx/conf.d/server.conf
upstream backend {
    server unix:/var/run/php-fpm/php-fpm-www.sock; weight=5;
    server unix:/var/run/php-fpm/php-fpm-http.sock;  weight=5;
}
```


