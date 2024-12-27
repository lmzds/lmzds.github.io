---
title: 学校宥马运动题库爬取
date: 2024-12-27 11:25 +0800
categories: [Python,Crawler]
tags: [python]
---



### 题库报文

```
POST /school/school/exam/library/getExamLibraryQuestionList HTTP/2
Host: hwy.328ym.com
Systemtype: android
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip, deflate
User-Agent: okhttp/4.2.2

examLibraryId=xxxxxxxxxxxxxxxxxxxxxxxxxxxxx&token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

题库返回是 json 格式，可以直接读取转成word。

### 脚本

```python
import requests
import json
from docx import Document
from docx.shared import Pt, RGBColor, Inches
from docx.oxml.ns import qn
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)


class WMD:
    def __init__(self):
        global d
        d = Document()
        # 全局设置 字号 字体，针对 段落
        d.styles['Normal'].font.size = Pt(16)
        d.styles['Normal'].font.name = 'Times New Roman'
        d.styles['Normal']._element.rPr.rFonts.set(qn('w:eastAsia'), u'黑体')

    def title(self, title_text):
        run = d.add_heading("", level=1).add_run(title_text)
        run.font.size = Pt(26)
        # 设置西文字体
        run.font.name = u'宋体'
        # 设置中文字体
        run._element.rPr.rFonts.set(qn('w:eastAsia'), u'宋体')
        run.bold = True
        run.font.color.rgb = RGBColor(0, 0, 0)

    def title2(self, title_text):
        run = d.add_heading("", level=2).add_run(title_text)
        run.font.size = Pt(26)
        # 设置西文字体
        run.font.name = u'宋体'
        # 设置中文字体
        run._element.rPr.rFonts.set(qn('w:eastAsia'), u'宋体')
        run.bold = True
        run.font.color.rgb = RGBColor(0, 0, 0)
    def parag(self, p_text):
        d.add_paragraph(p_text)

    def endcl(self, filename):
        d.save(filename+".docx")

    def endclPath(self, loadpath ,filename):
        d.save(loadpath+"/"+filename+".docx")


def getData(id):
# type 1 单选 ABCD 0123
# type 2 判断 0错 1对
# type 3 多选 ABCD 0123
# type 4 简答 
# 1 3 ；2 ；4
    def xzt(jsondata):
        if jsondata['type'] == 1:
            wmd.title2("单选题")
        else:
            wmd.title2("多选题")
            
        for i in jsondata['questionList']:
            title = i['title']
            wmd.parag(title)

            options = i['optionList']
            for option in options:
                # print(option)
                wmd.parag(option)
            answer = i['answer'].replace('0','A').replace('1','B').replace('2','C').replace('3','D')
            # print(title,answer)
            wmd.parag("答案: "+answer+"\n\n")

    def pd(jsondata):
        wmd.title2("判断题")
        for i in jsondata['questionList']:
            title = i['title']
            wmd.parag(title)
            answer = i['answer'].replace('0','错误').replace('1','正确')
            # print(title,answer)
            wmd.parag("答案: "+answer+"\n\n")

    def jd(jsondata):
        wmd.title2("简答题")
        for i in jsondata['questionList']:
            title = i['title']
            wmd.parag(title)
            answer = i['answer']
            # print(title,answer)
            wmd.parag("答案: "+answer+"\n\n")


    url = "https://hwy.328ym.com/school/school/exam/library/getExamLibraryQuestionList"
    param2 = {
        "examLibraryId":f"{id}",
        "token":f"{token}"
        }
    
    examJson = requests.post(url=url,data=param2,verify=False).json()
    # print(examJson)

    for items in examJson['data']:
        if items['type'] == 1 or items['type'] == 3:
            xzt(items)
        elif items['type'] == 2:
            pd(items)
        else:
            jd(items)


def getID():
    url = "https://hwy.328ym.com/school/school/exam/library/getExamLibraryList"

    param1= {
        "type":1,
        "token":f"{token}"
        }

    Get_idJson = requests.post(url=url,data=param1,verify=False).json()
    # print(type(Get_examLibraryId))
    for items in Get_idJson['data']:
        # global wmd
        # wmd = WMD()       # 和下面一起，轮换

        id = items['examLibraryId']
        name = items['examLibraryName']
        # print(id,name)
        print(name)
        wmd.title(name)
        getData(id)
        # wmd.endclPath('./题库',name)	# 分文件保存
    wmd.endcl("理论题库")	# 报存文件


    # return 

if __name__ == "__main__":
    token = "--token--"

    header = {
        "Host": "hwy.328ym.com",
        "Systemtype": "android",
        "Content-Type": "application/x-www-form-urlencoded",
        "Content-Length": "85",
        "Accept-Encoding": "gzip, deflate",
        "User-Agent": "okhttp/3.12.12"
    }
    
    wmd = WMD()

    getID()
    # getData("--[题库id]--")
    # wmd.endcl("理论题库")
    
```

通过网盘分享的文件：宥马题库
链接: https://pan.baidu.com/s/1wtWJjE19fM1nRi1ltDR8fQ?pwd=ts4g 提取码: ts4g