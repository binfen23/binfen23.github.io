---
title: 更新JsDelivr CDN缓存的方法
description: 
date: 2025-10-30 12:56:16 +0800
categories: []
tags: []
pin: false
math: false
mermaid: false
---
jsDelivr 缓存刷新方式#
对于 jsDelivr，缓存刷新的方式也很简单，只需将想刷新的链接的开头的
`https://cdn.jsdelivr.net/...`
替换成
`https://purge.jsdelivr.net/...`