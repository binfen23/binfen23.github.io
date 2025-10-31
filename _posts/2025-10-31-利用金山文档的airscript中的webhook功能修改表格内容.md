---
title: 利用金山文档的airscript中的webhook功能修改表格内容
description: 
date: 2025-10-31 11:16:26 +0800
categories: []
tags: []
pin: false
math: false
mermaid: false
---
这是airscript代码，这是将a1的内容改成webhook传输的内容，其中Context.argv.neirong的.neirong就是webhook中传输的对象名

```javascript
Application.Range('A1').Value = Context.argv.neirong
return {
  name: '执行成功'
}
```


这是python代码，客户端，url可以通过脚本的三个点，复制脚本webhook。token是脚本令牌，有180天的时间限制，可以无线延续


```python
import urllib.request
import json

url = "https://www.kdocs.cn/api/v3/ide/file/xxxxxxxxxxxx/script/V2-xxxxxxxxxxxx/sync_task"
payload = json.dumps({"Context":{"argv":{}}}).encode('utf-8')
headers = {
    'Content-Type': "application/json",
    'AirScript-Token': "xxxxxxxxxxxxx"
}

req = urllib.request.Request(url, data=payload, headers=headers, method='POST')
with urllib.request.urlopen(req) as response:
    data = response.read().decode('utf-8')
    print(data)

```