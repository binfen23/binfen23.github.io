---
title: 利用cloudflare的workers ai搭建的ai对话助手
description: 
date: 2025-10-24 12:57:12 +0800
categories: [cloudflare,大语言模型,api]
tags: [cloudflare,大语言模型,api]
pin: false
math: false
mermaid: false
---
https://api.cloudflare.com/client/v4/accounts/改成用户id值/ai/run/
"Authorization": "Bearer 改成token值"
@cf/qwen/qwen1.5-14b-chat-awq 是通义千问的模型可以在 cloudflare的 [workers ai models](https://developers.cloudflare.com/workers-ai/models/) 中看还有哪些模型

```python
import urllib.request
import json

API_BASE_URL = "https://api.cloudflare.com/client/v4/accounts/xxxxxxxxxx/ai/run/"
headers = {
    "Authorization": "Bearer xxxxxxxxxx",
    "Content-Type": "application/json"
}

def run(model, inputs):
    input_data = json.dumps({"messages": inputs}).encode('utf-8')
    request_url = f"{API_BASE_URL}{model}"
    req = urllib.request.Request(request_url, data=input_data, headers=headers, method='POST')
    
    with urllib.request.urlopen(req) as response:
        return json.loads(response.read().decode())

inputs = [
    { "role": "system", "content": "You are an AI assistant capable of answering all questions." },
    { "role": "user", "content": "你好" },
]

output = run("@cf/qwen/qwen1.5-14b-chat-awq", inputs)
print(output)

```