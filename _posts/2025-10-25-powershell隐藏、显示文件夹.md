---
title: powershell隐藏、显示文件夹
description: 
date: 2025-10-25 12:52:58 +0800
categories: []
tags: [windows,技巧]
pin: false
math: false
mermaid: false
---
当前文件夹内按住shift+右键  》 powershell运行

显示attrib -s -h /S /D
隐藏attrib +s +h /S /D

s是添加系统属性
h是添加隐藏属性

S是处理文件和文件夹
D是处理文件夹