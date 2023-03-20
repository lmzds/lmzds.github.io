---
title: selenuim自动化调用Chrome的两种方法
date: 2023-02-23 23:33 +0800
categories: [Python,Crawler]
tags: [selenium,chrome]
img_path: /assets/images/SeleniumAndChrome.assets
---



## 安装

Python3 安装略。

1. Python3 安装 selenium

```shell
pip3 install selenium
```

2. 安装 Chrome

使用 edge 去访问搜索下载即可。

https://www.google.com/chrome/

![image-20230320121950211](image-20230320121950211.png)



## 方法一

1. 查看 Chrome 版本

地址框输入：chrome://settings/help，确定自己的程序版本。

![image-20230320122810345](image-20230320122810345.png)

可以确定版本为 111.0.5563

2. 下载驱动

访问网址： http://chromedriver.storage.googleapis.com/index.html

最后两位数字选择 和你版本最接近的。

![image-20230320123817699](image-20230320123817699.png)

下载 windows 版本

![image-20230320124141724](image-20230320124141724.png)

解压到脚本路径即可。

```python
from selenium import webdriver

url = "https://baidu.com"
path =  Service('./chromedriver.exe')
driver = webdriver.Chrome(service=path)
driver.get(url)
```





## 方法二

第二种属于远程调试的方法。

创建一个快捷方式

![image-20230320160119659](image-20230320160119659.png)

如果为用户安装，要修改自己的chrome.exe路径

```
目标
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9999 --user-data-dir="C:\Users\Administrator\AppData\Local\Google\Chrome\User Data"

起始位置
"C:\Program Files\Google\Chrome\Application"
```

或者是 bat + vbs

```
cd /d "C:\Program Files\Google\Chrome\Application\"

chrome.exe --remote-debugging-port=9999 --user-data-dir="D:\CODE\py_all\python\Crawler script\chrome"
pause
```

vbs 是为了不会出现窗口，修改 filename 即可

```
CreateObject("WScript.Shell").Run "filename.bat",0
```



参数解释

--remote-debugging-port=9999

是设置远程调式的端口

--user-data-dir="D:\CODE\py_all\python\Crawler script\chrome"

设置用户数据目录，可以自定义设置到其他路径，不会影响到自己默认的配置。

```python
from selenium import webdriver

chrome_options = webdriver.ChromeOptions()
chrome_options.add_experimental_option("debuggerAddress", "127.0.0.1:9999")
driver = webdriver.Chrome(options=chrome_options)
```

