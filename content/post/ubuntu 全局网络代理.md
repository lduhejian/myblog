---
title: "ubuntu 全局网络代理"
date: 2021-02-20
draft: false
tags: [ubuntu, 网络代理, shadowsocks]
description: "ubuntu, 网络代理, shadowsocks export ALL_PROXY=socks5://"
---
在 ubuntu 中配置全局网络代理的方法:
```
sudo echo "export ALL_PROXY=socks5://ip:port" >> /etc/bash.bashrc
```
ip 是你 shadowsocks 客户端运行主机的 ip 地址  
port 是 shadowsocks 客户端本地 Socks5 监听端口  


怎么知道是不是代理成功？
```
curl cip.cc
```
如果返回 ip 地址等于代理服务器的 ip 地址则设置成功

参考：  
[Linux 让终端走代理的几种方法](https://zhuanlan.zhihu.com/p/46973701)