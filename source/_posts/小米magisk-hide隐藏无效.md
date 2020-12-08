---
title: 小米magisk_hide隐藏无效
toc: true
date: 2020-12-08 11:57:32
english_url: xiaomi_magisk_hide
tags: 
    - 小米
    - magisk
    - root
categories: 应用
---

小米开发版magisk hide隐藏无效是因为小米自带root文件su在作怪(检测root就是检测的小米root后留下的su文件)，用re或mt文件管理找到system/xbin/su删除即可，重新清除应用全部数据就ok了。