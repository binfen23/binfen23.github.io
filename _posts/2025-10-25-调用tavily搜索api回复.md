---
title: 调用tavily搜索api回复
description: 
date: 2025-10-25 12:44:39 +0800
categories: []
tags: [api,大语言模型]
pin: false
math: false
mermaid: false
---
```python
import urllib.request
import urllib.parse
import json

# API端点URL
url = "https://api.tavily.com/search"

# 请求头信息
headers = {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer xxxxxxxxxx'
}

# 请求数据
data = {
    "query": "提取文件“冲浪惊魂.2025.BD.1080P.中英双字.mp4”中的电影名，并且搜索豆瓣地址，直接回复电影名和豆瓣地址",
    "include_answer": "advanced",
    "search_depth": "advanced",
    "max_results": 20,
    "include_raw_content": "text",
    "include_domains": [
        "douban.com"
    ]
}

# 将数据转换为JSON字符串并编码为bytes
json_data = json.dumps(data).encode('utf-8')

# 创建请求对象
req = urllib.request.Request(url, data=json_data, headers=headers, method='POST')

try:
    # 发送请求并获取响应
    with urllib.request.urlopen(req) as response:
        # 读取响应内容并解码
        response_data = response.read().decode('utf-8')
        # 解析JSON响应
        result = json.loads(response_data)
        # 打印结果
        print(json.dumps(result, indent=2, ensure_ascii=False))
except urllib.error.URLError as e:
    print(f"请求错误: {e.reason}")
except urllib.error.HTTPError as e:
    print(f"HTTP错误: {e.code}")
    print(f"响应内容: {e.read().decode('utf-8')}")
except json.JSONDecodeError:
    print("解析响应JSON失败")
except Exception as e:
    print(f"发生错误: {str(e)}")

```