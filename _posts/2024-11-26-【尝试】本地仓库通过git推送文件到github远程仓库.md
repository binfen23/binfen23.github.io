---
title: [尝试] 本地仓库通过git推送文件到github远程仓库
description: 0基础。记录在本地仓库从0开始使用git工具推送文件到github远程仓库
date: 2024-11-26 17:29:12 +0800
categories: []
tags: [尝试,github,git]
pin: false
math: false
mermaid: false
---
>最近修改日期：2025-10-07 18:26:49  
> 本次尝试在 华为云 服务器上进行。
> 环境：debian系统



### 创建github仓库 & ssh密钥 验证github身份


首先在 github 中创建了一个名为 hello\_world 的公开仓库，然后先别管。

`不知道什么原因，推送的时候直接输入帐号密码的方式无法推送 本次通过在 github中添加 ssh密钥 的方式进行推送`  


接着来到 云服务器中。
通过下面的命令生成ssh密钥，然后添加到github中完成身份的验证。
```
ssh-keygen -t rsa -b 4096 -C " bfkjz@outlook.com"
```  

回车后，会提示私钥文件的保存地址（保存后可以通过 `cat /root/.ssh/id_rsa` 命令读取私钥  ）

`Enter file in which to save the key (/root/.ssh/id_rsa):`  

此时可以默认回车，会默认创建 /root/.ssh/id_rsa 私钥文件直接回车

输入回车后会提示是否创建密码  
` Enter passphrase (empty for no passphrase):`  

为了避免接下来会出各种小问题，本次直接 回车2次 跳过，否则就输入强密码，然后会提示再确认一遍密码。 
会提示私钥和公钥的保存地址  
```
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
```

接下来需要获取ssh公钥，并将生成的ssh公钥添加到github内。
```
 cat /root/.ssh/id_rsa.pub
```  

复制全部内容（包括刚才的注释部分）

访问 [github 设置页面中的 SSH and GPG keys](https://github.com/settings/keys) 添加ssh key  
绿色按钮 <span style="color: #00a720">【New SSH key】 </span>   

title随便填写，这里填写 "用于github身份验证"  
key type 是 Authentication Key （身份验证密钥）
将刚才复制的公钥，填进去key框内  
点击 <span style="color: #00a720">【Add SSH key】 </span>   完成添加


### 创建本地仓库 & 推送至远程仓库

重新来到云服务器， 在home目录创建了同名 hello\_world 的文件夹 并且进入该文件夹。
```
mkdir ~/hello_world  

cd ~/hello_world
```  

然后使用 `git init` 将刚才创建的文件夹初始化为 git仓库（会创建名为 .git 的隐藏文件夹）  
```
git init
```  

为这个本地 git仓库 设置 提交者 和 邮箱信息（貌似是强制要求需要的，后续会在github界面中显示出来）  
```
git config user.name "zeb"

git config user.email "bfkjz@outlook.com"
```

添加一个readme文档  
```
echo "# hello_world" >> README.md
```

添加 README.md 到 暂存区 
```
git add README.md
```  

添加 提交信息  
```
git commit -m "first commit"
```  

将默认 master 分支改成 main分支  
```
git branch -M main
```

与github远程仓库关联  （注：需要使用ssh的链接，也就是git开头.git结尾的）
```
git remote add origin git@github.com:binfen23/hello_world.git
```  

将本地仓库推送到 github远程仓库的main分支  
```
git push -u origin main
```  


### All done！！
至此，从0本地推送至github远程仓库，完美撒花🎉🎉