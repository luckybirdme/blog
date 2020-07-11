---
title: PHP 获取客户端真实 IP
date: 2020-07-09 08:28:11
tags: PHP
---

> REMOTE_ADDR

<!-- more -->


首先我们来了解相关变量的含义：

```php
// 浏览当前页面的用户计算机的ip地址
$_SERVER['REMOTE_ADDR']
// 客户端的ip
$_SERVER['HTTP_CLIENT_IP']
// 浏览当前页面的用户计算机的网关
// proxy_set_header X-Forwarded-For $remote_addr;
$_SERVER['HTTP_X_FORWARDED_FOR']
// nginx 代理模式下，获取客户端真实IP
$_SERVER['HTTP_X_REAL_IP']

```

下面我们通过简单的代码示例来介绍PHP获取客户端IP地址的方法。

```php
/**
 *  获取客户端IP地址
 */
function real_ip()
{

    $ip = $_SERVER['REMOTE_ADDR'];
    if (isset($_SERVER['HTTP_X_FORWARDED_FOR']) && preg_match_all('#\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}#s', $_SERVER['HTTP_X_FORWARDED_FOR'], $matches)) {
        foreach ($matches[0] AS $xip) {
            if (!preg_match('#^(10|172\.16|192\.168)\.#', $xip)) {
                $ip = $xip;
                break;
            }
        }
    } elseif (isset($_SERVER['HTTP_CLIENT_IP']) && preg_match('/^([0-9]{1,3}\.){3}[0-9]{1,3}$/', $_SERVER['HTTP_CLIENT_IP'])) {
        $ip = $_SERVER['HTTP_CLIENT_IP'];
    } elseif (isset($_SERVER['HTTP_CF_CONNECTING_IP']) && preg_match('/^([0-9]{1,3}\.){3}[0-9]{1,3}$/', $_SERVER['HTTP_CF_CONNECTING_IP'])) {
        $ip = $_SERVER['HTTP_CF_CONNECTING_IP'];
    } elseif (isset($_SERVER['HTTP_X_REAL_IP']) && preg_match('/^([0-9]{1,3}\.){3}[0-9]{1,3}$/', $_SERVER['HTTP_X_REAL_IP'])) {
        $ip = $_SERVER['HTTP_X_REAL_IP'];
    }
    return $ip;
}
```