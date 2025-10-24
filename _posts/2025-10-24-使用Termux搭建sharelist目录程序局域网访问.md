---
title: 使用Termux搭建sharelist目录程序局域网访问
description: 
date: 2025-10-24 12:52:34 +0800
categories: [termux]
tags: [termux]
pin: false
math: false
mermaid: false
---
一、Termux部署
下载源码安装
git clone https://github.com/reruin/sharelist.git
cd sharelist && bash install.sh
这个项目的部署依赖NodeJS运行环境（>=8.0）,在install.sh脚本中有安装nodejs的命令，可以自动安装（同时pm2管理器也会自动安装，用于运行管理程序），在安装过程中可能会需要梯子，视情况是否准备。
当然nodejs和pm2管理器也可以提前安装。

pkg install nodejs
npm install
npm install pm2 -g
但是在某些情况下可能会出现npm或pm2 not found，这时可以将install.sh里第三行的PATH注释掉（#号注释）。
接着就可以使用pm2启动程序了

pm2 start app.js --name sharelist --env prod
pm2 save
pm2 startup
安装成功后就可以使用http://127.0.0.1:33001或http://localhost:33001访问，使用http://127.0.0.1:33001/webdav即可挂载webdav

运行updata.sh即可更新程序

bash update.sh