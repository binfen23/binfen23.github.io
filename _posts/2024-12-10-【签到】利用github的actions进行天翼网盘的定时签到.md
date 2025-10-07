---
title: 利用github的actions进行天翼网盘的定时签到
description: 利用github的actions和cloudflare定时触发签到并且用企业微信机器人进行通知
date: 2024-12-10 12:12:56 +0800
categories: []
tags: [github,网盘,actions,签到,消息推送]
pin: false
math: false
mermaid: false
---

## 说明
> 代码由网络收集、ai回答、整理后修改而成，出自哪位原作者已经无从得知    
> 于 2024.11.28 测试通过，可以正常 签到、推送 通知   
> 由于github的CRON表达式会存在延迟执行问题（延迟1小时甚至更久，非时区问题）  
> 所以建议使用第三方支持 CRON表达式 或 定时执行 的平台使用webhook方式触发工作流的运行。   
> 例如 `Cloudflare Workers`  `WPS AirScript`

## 特点 
* 支持多账号
* 支持 手动触发 & webhook触发
* 支持 企业微信机器人通知


## 使用方法  

首先Fork本仓库，然后再点击   
仓库的 `Settings` - `Secrets and variables` - `Actions` - `New repository secret` 绿色按钮  创建以下三个 secret密钥   

`USERNAME` `PASSWORD` `WEBHOOK` 

| 变量名 | 单账户 | 多账户 | 备注 |  
| :------: | :------------ | :-------------: | :-------------: |  
| USERNAME | 15000000000 | 15000000000,13000000000 | 多账号用 `，` 隔开 （*支持中文逗号） |  
| PASSWORD | abc123 | abc123，def456| 多账号用 `，` 隔开 （*支持中文逗号） |  
| WEBHOOK  | https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx |  / | 企业微信群聊机器人的webhook地址 |  



#### 账号  
Name*  填入 `USERNAME`    
Secret* 填入 账号   
* 单账号Secret值   
例如 手机号 `15000000000`  
* 多账号Secret 值  
例如 `15000000000,13000000000`  用 `，` 隔开 （*支持中文逗号） 

#### 密码  
Name*  填入 `PASSWORD`    
Secret* 填入 密码   
* 单账号Secret值   
例如 `abc123`  
* 多账号Secret值   
* 例如 `abc123,def456`  用 `，` 隔开 （*支持中文逗号）    


#### 企业微信机器人信息通知  
Name*  填入 `WEBHOOK` 
Secret* 填入 群聊机器人的webhook地址   
例如 `https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx`   

#### 此时已经可以使用手动触发的方式，运行actions的工作流了。   




## webhook触发工作流    

点击本仓库的 `Settings` - `Webhooks`    

#### Payload URL *   
填写 `https://api.github.com/repos/拥有者名/仓库名/dispatches`   
#### Content type *   
修改成 `application/json`   

默认 `Enable SSL verification` 不用管   

往下拉，在 *`Which events would you like to trigger this webhook?`* 中，选择 `Let me select individual events.`   
然后勾选 `Workflow runs`  其他不用勾选，点 `Update webhook`   

接下来创建用于 webhook 的 token   
`右上角头像` - `Settings` - `Developer settings` - `Personal access tokens` - `Tokens (classic)`   

点击右上角的 `Generate new token` 下拉框，选 `Generate new token (classic)`    

Note 随便填    
`Expiration` 选 `No expiration`   
然后往下拉 勾选 `workflow`  其它不用选择往下拉，按 `Generate token` 绿色按钮，会显示一段ghp_开头的 token（例如：ghp_xxxxxxxxxxxxxxxxxx）需要保存好，只显示一次   


最后可以使用post请求，webhook触发工作流   
```bash
curl -L \
  -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer ghp_xxxxxxxxxxxxxxxxxx" \
  https://api.github.com/repos/拥有者/仓库名/dispatches \
  -d '{"event_type":"sign-in"}'
```



## github actions的workflows文件   

`sign_in.yml`
```yml  
name: tycloud_sign_in

on:
  repository_dispatch:
      types: [sign-in]
  workflow_dispatch:


permissions:
  contents: read

jobs:
  build:

    runs-on: ubuntu-latest
    env:
          USERNAME: ${{ secrets.USERNAME }}
          PASSWORD: ${{ secrets.PASSWORD }}
          WEBHOOK: ${{ secrets.WEBHOOK }}
    steps:
    - uses: actions/checkout@v4
    - name: Set up Python 3.10
      uses: actions/setup-python@v3
      with:
        python-version: "3.10"
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install requests rsa
    - name: Run
      run: |
        python main.py
```
## python签到脚本文件  
`main.py`  
```python  
import time
import random
import json
import base64
import rsa
import requests
import re
import os
# 企业微信webhook推送信息 启用1 禁用0
qywx_push = 1
msg_c = "☁️天翼云盘签到☁️"
BI_RM = list("0123456789abcdefghijklmnopqrstuvwxyz")
B64MAP = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
usernames = os.getenv('USERNAME')
passwords = os.getenv('PASSWORD')
usernames = usernames.replace('，', ',').strip()
passwords = passwords.replace('，', ',').strip()
ty_usernames = usernames.split(',')
ty_passwords = passwords.split(',')
# 将用户名和密码组合成一个列表
accounts = [{"username": u, "password": p} for u, p in zip(ty_usernames, ty_passwords)]
def int2char(a):
    return BI_RM[a]
def b64tohex(a):
    d = ""
    e = 0
    c = 0
    for i in range(len(a)):
        if a[i] != "=":
            v = B64MAP.index(a[i])
            if e == 0:
                e = 1
                d += int2char(v >> 2)
                c = 3 & v
            elif e == 1:
                e = 2
                d += int2char(c << 2 | v >> 4)
                c = 15 & v
            elif e == 2:
                e = 3
                d += int2char(c)
                d += int2char(v >> 2)
                c = 3 & v
            else:
                e = 0
                d += int2char(c << 2 | v >> 4)
                d += int2char(15 & v)
    if e == 1:
        d += int2char(c << 2)
    return d
def rsa_encode(j_rsakey, string):
    rsa_key = f"-----BEGIN PUBLIC KEY-----\n{j_rsakey}\n-----END PUBLIC KEY-----"
    pubkey = rsa.PublicKey.load_pkcs1_openssl_pem(rsa_key.encode())
    result = b64tohex((base64.b64encode(rsa.encrypt(f'{string}'.encode(), pubkey))).decode())
    return result
def login(username, password):
    global msg_c
    urlToken = "https://m.cloud.189.cn/udb/udb_login.jsp?pageId=1&pageKey=default&clientType=wap&redirectURL=https://m.cloud.189.cn/zhuanti/2021/shakeLottery/index.html"
    s = requests.Session()
    r = s.get(urlToken)
    pattern = r"https?://[^\s'\"]+"  # 匹配以http或https开头的url
    match = re.search(pattern, r.text)  # 在文本中搜索匹配
    if match:  # 如果找到匹配
        url = match.group()  # 获取匹配的字符串
    else:  # 如果没有找到匹配
        print("没有找到url")
        return None
    r = s.get(url)
    pattern = r"<a id=\"j-tab-login-link\"[^>]*href=\"([^\"]+)\""  # 匹配id为j-tab-login-link的a标签，并捕获href引号内的内容
    match = re.search(pattern, r.text)  # 在文本中搜索匹配
    if match:  # 如果找到匹配
        href = match.group(1)  # 获取捕获的内容
    else:  # 如果没有找到匹配
        print("没有找到href链接")
        return None
    r = s.get(href)
    captchaToken = re.findall(r"captchaToken' value='(.+?)'", r.text)[0]
    lt = re.findall(r'lt = "(.+?)"', r.text)[0]
    returnUrl = re.findall(r"returnUrl= '(.+?)'", r.text)[0]
    paramId = re.findall(r'paramId = "(.+?)"', r.text)[0]
    j_rsakey = re.findall(r'j_rsaKey" value="(\S+)"', r.text, re.M)[0]
    s.headers.update({"lt": lt})
    username = rsa_encode(j_rsakey, username)
    password = rsa_encode(j_rsakey, password)
    url = "https://open.e.189.cn/api/logbox/oauth2/loginSubmit.do"
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:74.0) Gecko/20100101 Firefox/76.0',
        'Referer': 'https://open.e.189.cn/',
    }
    data = {
        "appKey": "cloud",
        "accountType": '01',
        "userName": f"{{RSA}}{username}",
        "password": f"{{RSA}}{password}",
        "validateCode": "",
        "captchaToken": captchaToken,
        "returnUrl": returnUrl,
        "mailSuffix": "@189.cn",
        "paramId": paramId
    }
    r = s.post(url, data=data, headers=headers, timeout=5)
    if r.json().get('result', None) == 0:
        print(f"🥳{r.json()['msg']}🥳")
        msg_c += f"\n🥳{r.json()['msg']}🥳"
    else:
        print(f"😭{r.json()['msg']}😭")
        msg_c += f"\n🥳{r.json()['msg']}🥳"
    redirect_url = r.json()['toUrl']
    r = s.get(redirect_url)
    return s
def main():
    global msg_c
    num = 1
    for account in accounts:
        username = account["username"]
        password = account["password"]
        session = login(username, password)
        if session is not None:
            rand = str(round(time.time() * 1000))
            surl = f'https://api.cloud.189.cn/mkt/userSign.action?rand={rand}&clientType=TELEANDROID&version=8.6.3&model=Pixel 2'
            url = f'https://m.cloud.189.cn/v2/drawPrizeMarketDetails.action?taskId=TASK_SIGNIN&activityId=ACT_SIGNIN'
            url2 = f'https://m.cloud.189.cn/v2/drawPrizeMarketDetails.action?taskId=TASK_SIGNIN_PHOTOS&activityId=ACT_SIGNIN'
            url3 = f'https://m.cloud.189.cn/v2/drawPrizeMarketDetails.action?taskId=TASK_2022_FLDFS_KJ&activityId=ACT_SIGNIN'
            headers = {
                'User-Agent': 'Mozilla/5.0 (Linux; Android 8.0; Pixel 2 Build/OPD3.170816.012) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.5993.118 Mobile Safari/537.36 Ecloud/8.6.3 Android/26 clientId/355325117317828 clientModel/Pixel 2 imsi/460071114317824 clientChannelId/qq proVersion/1.0.6',
                "Referer": "https://m.cloud.189.cn/zhuanti/2016/sign/index.jsp?albumBackupOpened=1",
                "Host": "m.cloud.189.cn",
                "Accept-Encoding": "gzip, deflate",
            }
            response = session.get(surl, headers=headers)
            netdiskBonus = response.json()['netdiskBonus']
            if response.json()['isSign'] == "false":
                print(f"🆔账号{num} 🎁签到获得{netdiskBonus}M空间🎉🎉")
                msg_c += f"\n🆔账号{num} 🎁签到获得{netdiskBonus}M空间🎉🎉"
            else:
                print(f"🆔账号{num} 已经签到过了")
                msg_c += f"\n🆔账号{num} 已经签到过了"
            response = session.get(url, headers=headers)
            if "errorCode" in response.text:
                if "User_Not_Chance" in response.text:
                    print(f"🆔账号{num} 🥰抽奖1：已经抽奖过了🥰")
                    msg_c += f"\n🆔账号{num} 🥰抽奖1：已经抽奖过了🥰"
                else:
                    print(f"🆔账号{num} {response.text}")
                    msg_c += f"\n🆔账号{num} {response.text}"
            else:
                description = response.json()['description']
                print(f"🆔账号{num} 🎁抽奖1：获得{description}🎉🎉")
                msg_c += f"\n🆔账号{num} 🎁抽奖1：获得{description}🎉🎉"
            time.sleep(random.randint(5, 10))  
            response = session.get(url2, headers=headers)
            if "errorCode" in response.text:
                if "User_Not_Chance" in response.text:
                    print(f"🆔账号{num} 🥰抽奖2：已经抽奖过了🥰")
                    msg_c += f"\n🆔账号{num} 🥰抽奖2：已经抽奖过了🥰"
                else:
                    print(f"🆔账号{num} {response.text}")
                    msg_c += f"\n🆔账号{num} {response.text}"
            else:
                description = response.json()['prizeName']
                print(f"🆔账号{num} 🎁抽奖2：获得{description}🎉🎉")
                msg_c += f"\n🆔账号{num} 🎁抽奖2：获得{description}🎉🎉"
            time.sleep(random.randint(5, 10))      
            response = session.get(url3, headers=headers)
            if "errorCode" in response.text:
                if "User_Not_Chance" in response.text:
                    print(f"🆔账号{num} 🥰抽奖3：已经抽奖过了🥰")
                    msg_c += f"\n🆔账号{num} 🥰抽奖3：已经抽奖过了🥰"
                else:
                    print(f"🆔账号{num} 抽奖3：{response.text}")
                    msg_c += f"\n🆔账号{num} 抽奖3：{response.text}"
            else:
                description = response.json()['prizeName']
                print(f"🆔账号{num} 🎁抽奖3：获得{description}🎉🎉")
                msg_c += f"\n🆔账号{num} 🎁抽奖3：获得{description}🎉🎉"
            num += 1
def send_msg():
    
    if qywx_push == 1:
        webhook_url = os.getenv('WEBHOOK')
        if webhook_url is not None:
            data = {
                "msgtype": "text",
                "text": {
                    "content": msg_c
                }
            }
            response = requests.post(webhook_url, data=json.dumps(data), headers={'Content-Type': 'application/json'})
            if response.status_code == 200:
                print("发送成功")
                print(response.json())
            else:
                print(f"请求失败，状态码: {response.status_code}")
                print(response.text)
        else:
            print("环境变量 WEBHOOK 不存在\n请在github - settings - Secrets and variables - Actions - New repository secret\n请以上路径手动添加名称为WEBHOOK，值为企业微信webhook地址的密钥键值")
if __name__ == "__main__":
    main()
    send_msg()
```
  
 ## cloudflare的workers文件   
添加完以下的workers文件后，需要在对应的workers里设置触发条件  
`workers设置` -  `触发事件` -  `添加`  - `Cron 触发器` - `Cron 表达式` - `填写 0 2 * * *  ` 
以上是设置每天的早上10点触发，由于cloudflare的cron使用的是UTC时区，所以+8的话就是对应的中国的早上10点
```javascript  
// 定义 fetch 事件处理器，用于处理常规的HTTP请求
addEventListener('fetch', event => {
  // 对于非Cron触发的请求
  event.respondWith(handleOtherTrigger(event.request));
});
// 定义 scheduled 事件处理器，用于处理Cron触发
addEventListener('scheduled', event => {
  event.waitUntil(handleCron());
});
async function handleCron() {
  const response = await handleRequest();
  return new Response('Cron job executed and notification sent', { status: 200 });
}
async function handleOtherTrigger(request) {
  return new Response(`为避免频繁触发签到，暂不支持HTTP请求，请在设置页面添加Cron表达式进行定时触发`, { status: 200 });
}
async function handleRequest() {
  const url = 'https://api.github.com/repos/拥有者名/仓库名/dispatches';  //这里记得修改 拥有者 和 仓库名
  const token = 'ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxx';  //这里填写 github 的 access token
  const body = JSON.stringify({ "event_type": "sign-in" });
  const init = {
    method: 'POST',
    headers: {
      'Accept': 'application/vnd.github+json',
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json',
      'User-Agent': 'Mozilla/5.0 (Linux; Android 8.0; Pixel 2 Build/OPD3.170816.012) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/102.0.0.0 Mobile Safari/537.36'
    },
    body: body
  };
  try {
    const response = await fetch(url, init);
    if (response.ok) {
      return new Response('签到成功！', { status: 200 });
    } else {
      const errorMessage = await response.text();
      throw new Error(`触发签到失败: ${errorMessage}`);
    }
  } catch (error) {
    throw new Error(`请求过程中发生错误: ${error.message}`);
  }
}
```
  
## All Done!!

