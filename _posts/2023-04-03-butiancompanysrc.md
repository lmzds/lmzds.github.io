---
title: 补天公益src名单爬取
date: 2023-04-03 10:42 +0800
categories: [Python,Crawler]
tags: [xhs,python,butian]
img_path: /assets/images/butiancompanysrc.assets
---



```python
import requests
import json

def main(page):
    url = "https://www.butian.net/Reward/pub"
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36"
    }
    data = {"p":page}

    jsondata = json.loads(requests.post(url,data=data,headers=headers).content.decode("utf-8"))
    data = jsondata["data"]["list"]
    for i in data:
        company = i["company_name"]
        with open("butiancompany.txt","a+") as f:
            f.write(company + "\n")

for i in range(1,192+1):
    main(i)
```

![image-20230403104345136](image-20230403104345136.png)