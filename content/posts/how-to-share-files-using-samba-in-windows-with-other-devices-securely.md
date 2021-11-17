---
title: "如何安全地在 Windows 中开启 SMB 服务与其他设备进行文件共享"
date: 2021-03-27T14:51:30+08:00
isCJKlanguage: true
categories:
  - "2021年3月"
tags:
  - "SMB"
  - "SMB 1.0"
  - "文件共享"
keywords:
  - "安全"
description: 
draft: true
---

## 前言

自从 iOS 13 加入 SMB 支持之后，网上关于 Windows 设备与 iOS 设备进行文件共享的文章如雨后春笋般冒了出来，许多文章中都或多或少的提到了启用 Windows “SMB 1.0/CIFS 文件共享”这一功能，理由无非是“不开会连不上”、“显示账号密码错误看看这个功能有没有启用”。

殊不知这些问题与是否启用 SMB 1.0 没有关系，作为近 30 年的SMB 1.0，其本身有很大的[安全隐患](https://techcommunity.microsoft.com/t5/storage-at-microsoft/stop-using-smb1/ba-p/425858)，不应继续使用。

希望本文能让读者在自己的设备间，使用 SMB 服务稳定安全地进行文件共享。

## 