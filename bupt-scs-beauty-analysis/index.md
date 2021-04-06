# 2019北邮计算机学院新生颜值分析


<!--more-->

**声明：为保护个人隐私，本博文不提供任何学生的个人信息。**

偶得管理员账号🤐，分析2019计算机学院迎新页面找到了新生头像存放位置，正好赋闲，于是花了半天功夫，作了个颜值分析，却“意外”有了惊人的发现！

> Talk is cheap. Show me the code. - Linus Torvalds

## 头像爬取

爬虫基于request，大致思路是模拟登陆 -> 获取LT -> 进入迎新页面获取头像访问权 -> 按照学号逐个爬取头像

```py
#-*-coding=utf-8-*-
import os
import requests
import json
from http import cookiejar
from bs4 import  BeautifulSoup  as bs

s = requests.Session()
header =  {'User-Agent':'Mozilla/5.0 (Windows NT 10.0; WOW64; rv:46.0) Gecko/20100101 Firefox/46.0'}

def getLt(str):
# 获取LT，相当于免再次登录的流水号，不可或缺
lt = bs(str,'html.parser')
dic =  {}
for inp in lt.form.find_all('input'):
if(inp.get('name'))!=None:
dic[inp.get('name')]=inp.get('value')
return dic

def craw(num):
base_url =  "*************"  # 头像存储地址
img_url = base_url + str(num)
img_name = str(num)+'.jpg'
img_path = os.path.abspath(".\\2019scs_face")
img_save = img_path +  "\\"  + img_name
if  not os.path.isdir(img_path):
os.makedirs(img_path)

img_res = s.get(img_url, headers=header)
img = img_res.content
f = open(img_save,  "wb")
f.write(img)
f.close()
print("image of %d saved"%num)

def main():
r = s.get('https://auth.bupt.edu.cn/authserver/login?service=http%3A%2F%2Fmy.bupt.edu.cn%2Findex.portal',headers=header)
dic = getLt(r.text)
postdata =  {
'username':'*************',  #此处为管理员的账号，学生账号无法爬取头像
'password':'*************',  #密码
'lt':dic['lt'],
'execution':'e1s1',
'_eventId':'submit',
'rmShown':'1'
}
response_login = s.post('https://auth.bupt.edu.cn/authserver/login?service=http%3A%2F%2Fmy.bupt.edu.cn%2Findex.portal',data=postdata,headers=header)

response2 = s.post('*************', data=postdata,headers=header)  # 获得头像地址访问权
# 大类：
for num in range(**********,  ***********):  # 新手学号范围
craw(num)
# 智科：
for num in range(**********,  ***********):  # 新手学号范围
craw(num)

main()
```

一共爬取549个头像，按照学号保存。

![](../../_resources/ac60119ccfd349a094b980ee26287cc2.png)

## 性别划分 \+ 颜值评估

使用了百度AI的[人脸检测](https://ai.baidu.com/tech/face/detect)v3版本的API，在管理控制台中创建应用，获取AppID、API Key和Secret Key：

![](../../_resources/014706bce7754339a3116590165ba334.png)

首先安装 baidu-aip :
```sh
pip install baidu-aip
```
调用API分析下载的头像，分析后，按照性别分别保存，为了省事儿，直接将颜值得分加入文件名中，完整代码如下：
```py
#-*-coding=utf-8-*-
import time
import os
import base64

from PIL import  Image
from aip import  AipFace

# 定义常量
APP_ID =  "**********"
API_KEY =  "**********"
SECRET_KEY =  "**********"` 

# 初始化AipFace对象
aipFace =  AipFace(APP_ID, API_KEY, SECRET_KEY)` 

# 设置参数
options =  {
"face_field":  "age,beauty,gender",` 
"max_face_num":  1  ,
"face_type":  "CERT",
}

imageType =  "BASE64"

#将图片进行base64编码，再url编码
def get_file_content(filePath):` 
with open(filePath,  'rb')  as fp:` 
image = fp.read()` 
image_data = base64.b64encode(image)
return image_data.decode("utf-8")

def main():
img_file = os.walk(".\\2019scs_face")
for path,dir_list,file_list in img_file:
for file_name in file_list:
file_path = os.path.join(path, file_name)
print(file_name)

# 调用人脸属性检测接口` 
result = aipFace.detect(get_file_content(file_path),imageType,options)
face_info = result['result']['face_list'][0]
age = face_info['age']
beauty = face_info['beauty']
gender = face_info['gender']['type']
print(gender, age, beauty)

dir_ =  ".\\2019scs_face_done"
img =  Image.open(file_path)
if gender ==  'male':
img.save(dir_ +  "\\male\\"  + str(beauty)  +  '_'+ file_name)
else:
img.save(dir_ +  "\\female\\"  + str(beauty)  +  '_'+ file_name)
print()
time.sleep(0.5)  # 降低调用频率，防止QPS超限额
main()
```
分析完成后，目测发现性别划分有5个划分出错（都是将妹子划分成汉子了🤣）。 因绝大多数新生照片拍摄环境恶劣，都是“清水出芙蓉，天然去雕饰”，而且AI评分很玄乎、准确度不高，所以呢……

没多少人颜值分数及格。

* * *

按照评分的数据显示：

- 女生共120人，颜值最高分57.04，最低分9.37（？？？）
- 男生共429人，颜值最高分79.52，最低分30.86（！！！）

为了更直观的展现新生的颜值，我们使用 Matplotlib 画下颜值分布图。

## 颜值分布图

代码实现如下：
```py
import os
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

male =  []
female =  []

img_file = os.walk(".\\2019scs_face_done\\male")
for path,dir_list,file_list in img_file:
for file_name in file_list:
val = file_name.split('_')[0]
male.append(float(val))

img_file = os.walk(".\\2019scs_face_done\\female")
for path,dir_list,file_list in img_file:
for file_name in file_list:
val = file_name.split('_')[0]
female.append(float(val))

sns.kdeplot(male, bw=0.5, label="male")
sns.kdeplot(female, bw=0.5, label="female")
plt.title("Distribution of 2019SCS Face Score")
plt.xlabel("Face Score")
plt.ylabel("Probability")
plt.savefig("./Distribution.jpg")
plt.show()
```
展现出了下图：

![](../../_resources/99becbf9b4944728b5404817ce706b7a.jpg)

男生平均颜值53.15， 女生平均颜值36.91

## 结果分析

女生颜值得分较低的原因很简单，百度人脸检测模型的训练数据来自互联网，而美图工具等的使用大大提高了网络照片的“颜值”（尤其是女性），导致模型的“审美阈值”过高，看到素颜、不加修饰的脸庞，颜值分数也就这么骨感了…（机器评分仅供参考）

这便有了所谓的证件照的悲剧。

同理，影视剧等里的男神女神们，也提高了我们的“审美阈值”，为我们灌输了不健康的审美标准，这直接导致了近年来求偶动力、结婚率乃至全球范围内生育率（来自vczh）的走低😱，也极大了降低了人们的幸福水平，值得警醒和反思。（严肃脸）

**参考**：

- https://ai.baidu.com/docs#/Face-Detect-V3/top
    
- https://zhuanlan.zhihu.com/p/34425618
