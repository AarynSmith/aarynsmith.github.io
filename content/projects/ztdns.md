---
title: "ztDNS"
subtitle: ZeroTier DNS server
date: 2017-06-15T11:11:00-06:00
tags:  ["Go", "ZeroTier", "DNS"]
draft: false
comments: false
---

A DNS server for a ZeroTier network (or networks).
<!--more-->

ztDNS pulls device names from ZeroTier and makes them available by name using either IPv4 assigned addresses or IPv6 assigned addresses.

I created this project because I could not find a simple way to host a DNS server for my ZeroTier networks that would serve the names I had defined in the ZeroTier administration interface. I wanted to be able to address the devices on my network by the names I had defined.

The project is available on [GitHub](https://github.com/uxbh/ztdns).