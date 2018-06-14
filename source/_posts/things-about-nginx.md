---
title: 用Nginx代理解决一些问题总结
date: 2018-04-09 18:17:53
tags:
  - nginx
  - 运维
categories:
  - 开发
---

总结一下用Nginx代理解决的一些问题的思路。

小程序校验domain

docker暴露端口不够

WFG墙翻（反），外网不可以


websockt：ws变wss

## IPV6不支持的情况
阿里云、腾讯云都不支持IPV6，如果代理时返回的IPV6将会报错：
```
Address family not supported by protocol
```
解决办法是在DNS后指定不返回IPV6地址
```
resolver 8.8.8.8 ipv6=off;
```


