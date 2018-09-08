---
title: "A,CNAME record在DNS中的区别"
date: 2018-09-08T00:00:47+08:00
draft: false
tags: ["network"]
author: "Peter Yang"
---
`A`,`CNAME`都可以将host name map到一个或多个IP地址，但是他们是有区别的.

首先`A` record 和 `CNAME` record都是标准的DNS record.

`A` record 是将一个host name map到一个具体的IP地址, 比如你要将 `example.com` 指向 `11.11.11.11` 那么你可以如下配置

	example.com		A		11.11.11.11

`CNAME` 是将一个域名指向另外一个域名,而不是IP。`CNAME` source 代表是目标域名的一个别名并且继承了全部的解析链

	blog.dnsimple.com      CNAME   aetrion.github.io
	aetrion.github.io      CNAME   github.map.fastly.net
	github.map.fastly.net  A       185.31.17.133

如上边的例子所示 `blog.dnsimple.com` 是 `aetrion.github.io` 的一个别名，而`aetrion.github.io`又是`github.map.fastly.net`的一个别名，`github.map.fastly.net`有一条 `A` record 指向了`185.31.17.133`,所以访问3个中的任意一个网址都会访问到185.31.17.133