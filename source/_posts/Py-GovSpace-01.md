---
title: 国家统计局统计用区划代码和城乡划分代码爬虫(二)
date: 2020-03-21 08:46:52
tags: ['码爬']
categories: 码爬
keywords: ['码爬','python']
top_img: https://goss1.cfp.cn/creative/vcg/veer/400/new/VCG41N680280554.jpg?x-oss-process=image/format,webp
cover : https://www.xmtzxm.com/uploads/allimg/181224/5-1Q22419514Q18.jpg
---
## 国家统计局统计用区划代码和城乡划分代码爬虫 (二)--实现

本文详细分析了国家统计局统计用区划代码和城乡划分代码爬虫的实现过程，这是第二篇，详细分析了爬取全过程。

## 一、总体思路说明

首先我定义了一个网页爬取函数，然后依次定义省级代码获取函数、市级代码获取函数、区级代码获取函数、街道代码获取函数、居委会代码获取函数，这些函数都会调用网页爬取函数。其中区级代码获取函数、街道代码获取函数、居委会代码获取函数这三个函数都是多线程实现爬取的。最后我将爬取得到的数据输出为 csv 格式文件。

### 1、库函数导入

```python
import requests
from lxml import etree
import csv
import time
import pandas as pd
from queue import Queue
from threading import Thread
from fake_useragent import UserAgent
```

### 2、网页爬取函数

```python
# 下面加入了num_retries这个参数，经过测试网络正常一般最多retry一次就能获得结果
def getUrl(url,num_retries = 5):
    ua = UserAgent()
    headers = {'User-Agent':ua.random}
    try:
        response = requests.get(url,headers = headers)
        response.encoding = response.apparent_encoding
        data = response.text
        return data
    except Exception as e:
        if num_retries > 0:
            time.sleep(10)
            print(url)
            print("requests fail, retry!")
            return getUrl(url,num_retries-1) #递归调用
        else:
            print("retry fail!")
            print("error: %s" % e + " " + url)
            return #返回空值，程序运行报错
```

### 3、获取省级代码函数

```python
def getProvince(url):
    province = []
    data = getUrl(url)
    selector = etree.HTML(data)
    provinceList = selector.xpath('//tr[@class="provincetr"]')
    for i in provinceList:
        provinceName = i.xpath('td/a/text()') #这里如果采用//a/text()路径会出现问题！！
        provinceLink = i.xpath('td/a/@href')
        for j in range(len(provinceLink)):
            provinceURL = url[:-10] + provinceLink[j] #根据获取到的每个省的链接进行补全，得到真实的URL。
            province.append({'name':provinceName[j],'link':provinceURL})
    return province
pro = getProvince("http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/index.html")
df_province = pd.DataFrame(pro)
df_province['link']
0     http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
1     http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
2     http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
3     http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
4     http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
5     http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
6     http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
7     http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
8     http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
9     http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
10    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
11    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
12    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
13    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
14    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
15    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
16    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
17    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
18    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
19    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
20    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
21    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
22    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
23    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
24    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
25    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
26    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
27    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
28    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
29    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
30    http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf...
Name: link, dtype: object
```

#### 信息写入 csv 文件

```python
df_province.to_csv('province.csv', sep=',', header=True, index=False)
```

### 4、获取市级代码函数

```python
def getCity(url_list):
    city_all = []
    for url in url_list:
        data = getUrl(url)
        selector = etree.HTML(data)
        cityList = selector.xpath('//tr[@class="citytr"]')
        #下面是抓取每一个城市的代码、URL
        city = []
        for i in cityList:
            cityCode = i.xpath('td[1]/a/text()')
            cityLink = i.xpath('td[1]/a/@href')
            cityName = i.xpath('td[2]/a/text()')
            for j in range(len(cityLink)):
                cityURL = url[:-7] + cityLink[j]
                city.append({'name':cityName[j],'code':cityCode[j],'link':cityURL})
        city_all.extend(city) #所有省的城市信息合并在一起
    return city_all
city = getCity(df_province['link'])
df_city = pd.DataFrame(city)
df_city
```

|      | code         | link                                              | name                   |
| :--- | :----------- | :------------------------------------------------ | :--------------------- |
| 0    | 110100000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 市辖区                 |
| 1    | 120100000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 市辖区                 |
| 2    | 130100000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 石家庄市               |
| 3    | 130200000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 唐山市                 |
| 4    | 130300000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 秦皇岛市               |
| 5    | 130400000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 邯郸市                 |
| 6    | 130500000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 邢台市                 |
| 7    | 130600000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 保定市                 |
| 8    | 130700000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 张家口市               |
| 9    | 130800000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 承德市                 |
| 10   | 130900000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 沧州市                 |
| 11   | 131000000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 廊坊市                 |
| 12   | 131100000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 衡水市                 |
| 13   | 139000000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 省直辖县级行政区划     |
| 14   | 140100000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 太原市                 |
| 15   | 140200000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 大同市                 |
| 16   | 140300000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 阳泉市                 |
| 17   | 140400000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 长治市                 |
| 18   | 140500000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 晋城市                 |
| 19   | 140600000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 朔州市                 |
| 20   | 140700000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 晋中市                 |
| 21   | 140800000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 运城市                 |
| 22   | 140900000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 忻州市                 |
| 23   | 141000000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 临汾市                 |
| 24   | 141100000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 吕梁市                 |
| 25   | 150100000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 呼和浩特市             |
| 26   | 150200000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 包头市                 |
| 27   | 150300000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 乌海市                 |
| 28   | 150400000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 赤峰市                 |
| 29   | 150500000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 通辽市                 |
| ...  | ...          | ...                                               | ...                    |
| 314  | 622900000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 临夏回族自治州         |
| 315  | 623000000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 甘南藏族自治州         |
| 316  | 630100000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 西宁市                 |
| 317  | 630200000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 海东市                 |
| 318  | 632200000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 海北藏族自治州         |
| 319  | 632300000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 黄南藏族自治州         |
| 320  | 632500000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 海南藏族自治州         |
| 321  | 632600000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 果洛藏族自治州         |
| 322  | 632700000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 玉树藏族自治州         |
| 323  | 632800000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 海西蒙古族藏族自治州   |
| 324  | 640100000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 银川市                 |
| 325  | 640200000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 石嘴山市               |
| 326  | 640300000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 吴忠市                 |
| 327  | 640400000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 固原市                 |
| 328  | 640500000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 中卫市                 |
| 329  | 650100000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 乌鲁木齐市             |
| 330  | 650200000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 克拉玛依市             |
| 331  | 650400000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 吐鲁番市               |
| 332  | 650500000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 哈密市                 |
| 333  | 652300000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 昌吉回族自治州         |
| 334  | 652700000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 博尔塔拉蒙古自治州     |
| 335  | 652800000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 巴音郭楞蒙古自治州     |
| 336  | 652900000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 阿克苏地区             |
| 337  | 653000000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 克孜勒苏柯尔克孜自治州 |
| 338  | 653100000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 喀什地区               |
| 339  | 653200000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 和田地区               |
| 340  | 654000000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 伊犁哈萨克自治州       |
| 341  | 654200000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 塔城地区               |
| 342  | 654300000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 阿勒泰地区             |
| 343  | 659000000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 自治区直辖县级行政区划 |

#### 信息写入 csv 文件

```python
df_city.to_csv('city.csv', sep=',', header=True, index=False)
```

### 5、获取区级代码函数 — 多线程实现

```python
def getCounty(url_list):
    queue_county = Queue() #队列
    thread_num = 10 #进程数
    county = [] #记录区级信息的字典（全局）
    
    def produce_url(url_list):
        for url in url_list:
            queue_county.put(url) # 生成URL存入队列，等待其他线程提取
    
    def getData():
        while not queue_county.empty(): # 保证url遍历结束后能退出线程
            url = queue_county.get() # 从队列中获取URL
            data = getUrl(url)
            selector = etree.HTML(data)
            countyList = selector.xpath('//tr[@class="countytr"]')
            #下面是爬取每个区的代码、URL
            for i in countyList:
                countyCode = i.xpath('td[1]/a/text()')
                countyLink = i.xpath('td[1]/a/@href')
                countyName = i.xpath('td[2]/a/text()')
                #上面得到的是列表形式的，下面将其每一个用字典存储
                for j in range(len(countyLink)):
                    countyURL = url[:-9] + countyLink[j]
                    county.append({'code':countyCode[j],'link':countyURL,'name':countyName[j]})
                
    def run(url_list):
        produce_url(url_list)
    
        ths = []
        for _ in range(thread_num):
            th = Thread(target = getData)
            th.start()
            ths.append(th)
        for th in ths:
            th.join()
            
    run(url_list)
    return county
county = getCounty(df_city['link'])
df_county = pd.DataFrame(county)
df_county
```



|      | code         | link                                              | name                   |
| :--- | :----------- | :------------------------------------------------ | :--------------------- |
| 0    | 130702000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 桥东区                 |
| 1    | 130703000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 桥西区                 |
| 2    | 130705000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 宣化区                 |
| 3    | 130706000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 下花园区               |
| 4    | 130708000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 万全区                 |
| 5    | 130709000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 崇礼区                 |
| 6    | 130722000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 张北县                 |
| 7    | 130723000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 康保县                 |
| 8    | 130724000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 沽源县                 |
| 9    | 130725000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 尚义县                 |
| 10   | 130726000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 蔚县                   |
| 11   | 130727000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 阳原县                 |
| 12   | 130602000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 竞秀区                 |
| 13   | 130606000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 莲池区                 |
| 14   | 130607000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 满城区                 |
| 15   | 130608000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 清苑区                 |
| 16   | 130609000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 徐水区                 |
| 17   | 130623000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 涞水县                 |
| 18   | 130624000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 阜平县                 |
| 19   | 130626000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 定兴县                 |
| 20   | 130627000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 唐县                   |
| 21   | 130628000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 高阳县                 |
| 22   | 130629000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 容城县                 |
| 23   | 130630000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 涞源县                 |
| 24   | 130631000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 望都县                 |
| 25   | 130632000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 安新县                 |
| 26   | 130633000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 易县                   |
| 27   | 130634000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 曲阳县                 |
| 28   | 130635000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 蠡县                   |
| 29   | 130636000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 顺平县                 |
| ...  | ...          | ...                                               | ...                    |
| 2822 | 653128000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 岳普湖县               |
| 2823 | 653129000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 伽师县                 |
| 2824 | 654221000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 额敏县                 |
| 2825 | 652901000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 阿克苏市               |
| 2826 | 654223000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 沙湾县                 |
| 2827 | 652922000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 温宿县                 |
| 2828 | 653130000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 巴楚县                 |
| 2829 | 654224000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 托里县                 |
| 2830 | 652923000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 库车县                 |
| 2831 | 654225000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 裕民县                 |
| 2832 | 653131000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 塔什库尔干塔吉克自治县 |
| 2833 | 654226000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 和布克赛尔蒙古自治县   |
| 2834 | 652924000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 沙雅县                 |
| 2835 | 652925000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 新和县                 |
| 2836 | 652926000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 拜城县                 |
| 2837 | 652927000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 乌什县                 |
| 2838 | 652928000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 阿瓦提县               |
| 2839 | 652929000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 柯坪县                 |
| 2840 | 659001000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 石河子市               |
| 2841 | 659002000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 阿拉尔市               |
| 2842 | 659003000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 图木舒克市             |
| 2843 | 659004000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 五家渠市               |
| 2844 | 659006000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 铁门关市               |
| 2845 | 654301000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 阿勒泰市               |
| 2846 | 654321000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 布尔津县               |
| 2847 | 654322000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 富蕴县                 |
| 2848 | 654323000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 福海县                 |
| 2849 | 654324000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 哈巴河县               |
| 2850 | 654325000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 青河县                 |
| 2851 | 654326000000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 吉木乃县               |

#### 排序

由于多线程的关系，数据的顺序已经被打乱，所以这里按照区代码进行 “升序” 排序。

```python
df_county_sorted = df_county.sort_values(by = ['code']) #按1列进行升序排序
```

#### 信息写入 csv 文件

```python
df_county_sorted.to_csv('county.csv', sep=',', header=True, index=False)
```

### 6、获取街道代码函数 — 多线程实现

```python
def getTown(url_list):
    queue_town = Queue() #队列
    thread_num = 50 #进程数
    town = [] #记录街道信息的字典（全局）
    
    def produce_url(url_list):
        for url in url_list:
            queue_town.put(url) # 生成URL存入队列，等待其他线程提取
    
    def getData():
        while not queue_town.empty(): # 保证url遍历结束后能退出线程
            url = queue_town.get() # 从队列中获取URL
            data = getUrl(url)
            selector = etree.HTML(data)
            townList = selector.xpath('//tr[@class="towntr"]')
            #下面是爬取每个区的代码、URL
            for i in townList:
                townCode = i.xpath('td[1]/a/text()')
                townLink = i.xpath('td[1]/a/@href')
                townName = i.xpath('td[2]/a/text()')
                #上面得到的是列表形式的，下面将其每一个用字典存储
                for j in range(len(townLink)):
                    townURL = url[:-11] + townLink[j]
                    town.append({'code':townCode[j],'link':townURL,'name':townName[j]})
                
    def run(url_list):
        produce_url(url_list)
    
        ths = []
        for _ in range(thread_num):
            th = Thread(target = getData)
            th.start()
            ths.append(th)
        for th in ths:
            th.join()
            
    run(url_list)
    return town
town = getTown(df_county['link'])
df_town = pd.DataFrame(town)
df_town
```

|       | code         | link                                              | name                           |
| :---- | :----------- | :------------------------------------------------ | :----------------------------- |
| 0     | 130706001000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 城镇街道办事处                 |
| 1     | 130706002000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 煤矿街道办事处                 |
| 2     | 130706200000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 花园乡                         |
| 3     | 130706201000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 辛庄子乡                       |
| 4     | 130706202000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 定方水乡                       |
| 5     | 130706203000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 段家堡乡                       |
| 6     | 130702001000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 红旗楼街道办事处               |
| 7     | 130702002000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 胜利北路街道办事处             |
| 8     | 130702003000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 五一路街道办事处               |
| 9     | 130702004000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 花园街街道办事处               |
| 10    | 130702005000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 工业路街道办事处               |
| 11    | 130702101000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 姚家庄镇                       |
| 12    | 130623001000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 城区社区管理办公室街道办事处   |
| 13    | 130624100000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 阜平镇                         |
| 14    | 130624101000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 龙泉关镇                       |
| 15    | 130626100000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 定兴镇                         |
| 16    | 130623100000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 涞水镇                         |
| 17    | 130624102000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 平阳镇                         |
| 18    | 130624103000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 城南庄镇                       |
| 19    | 130624104000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 天生桥镇                       |
| 20    | 130624105000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 王林口镇                       |
| 21    | 130624202000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 台峪乡                         |
| 22    | 130624203000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 大台乡                         |
| 23    | 130624204000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 史家寨乡                       |
| 24    | 130624205000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 砂窝乡                         |
| 25    | 130724100000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 平定堡镇                       |
| 26    | 130724101000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 小厂镇                         |
| 27    | 130724102000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 黄盖淖镇                       |
| 28    | 130724103000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 九连城镇                       |
| 29    | 130724200000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 高山堡乡                       |
| ...   | ...          | ...                                               | ...                            |
| 42532 | 659002509000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 兵团十六团                     |
| 42533 | 659002511000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 兵团第一师水利水电工程处       |
| 42534 | 659002512000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 兵团第一师塔里木灌区水利管理处 |
| 42535 | 659002513000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 阿拉尔农场                     |
| 42536 | 659002514000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 兵团第一师幸福农场             |
| 42537 | 659002515000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 中心监狱                       |
| 42538 | 659002516000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 兵团一团                       |
| 42539 | 659002517000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 兵团农一师沙井子水利管理处     |
| 42540 | 659002518000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 西工业园区管理委员会           |
| 42541 | 659002519000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 兵团二团                       |
| 42542 | 659002520000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 兵团三团                       |
| 42543 | 522701001000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 广惠街道办事处                 |
| 42544 | 522701002000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 文峰街道办事处                 |
| 42545 | 522701004000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 小围寨街道办事处               |
| 42546 | 522701005000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 沙包堡街道办事处               |
| 42547 | 522701006000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 绿茵湖街道办事处               |
| 42548 | 522701106000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 墨冲镇                         |
| 42549 | 522701107000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 平浪镇                         |
| 42550 | 522701110000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 毛尖镇                         |
| 42551 | 522701111000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 匀东镇                         |
| 42552 | 522701208000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 归兰水族乡                     |
| 42553 | 652928100000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 阿瓦提镇                       |
| 42554 | 652928101000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 乌鲁却勒镇                     |
| 42555 | 652928102000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 拜什艾日克镇                   |
| 42556 | 652928200000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 阿依巴格乡                     |
| 42557 | 652928201000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 塔木托格拉克乡                 |
| 42558 | 652928202000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 英艾日克乡                     |
| 42559 | 652928203000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 多浪乡                         |
| 42560 | 652928204000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 巴格托格拉克乡                 |
| 42561 | 652928405000 | http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhf... | 阿克苏监狱                     |

#### 排序

由于多线程的关系，数据的顺序已经被打乱，所以这里按照街道代码进行 “升序” 排序。

```python
df_town_sorted = df_town.sort_values(by = ['code']) #按1列进行升序排序
```

#### 信息写入 csv 文件

```python
df_town_sorted.to_csv('town.csv', sep=',', header=True, index=False)
```

### 7、获取居委会代码函数 — 多线程实现

```python
def getVillage(url_list):
    queue_village = Queue() #队列
    thread_num = 200 #进程数
    town = [] #记录街道信息的字典（全局）
    
    def produce_url(url_list):
        for url in url_list:
            queue_village.put(url) # 生成URL存入队列，等待其他线程提取
    
    def getData():
        while not queue_village.empty(): # 保证url遍历结束后能退出线程
            url = queue_village.get() # 从队列中获取URL
            data = getUrl(url)
            selector = etree.HTML(data)
            villageList = selector.xpath('//tr[@class="villagetr"]')
            #下面是爬取每个区的代码、URL
            for i in villageList:
                villageCode = i.xpath('td[1]/text()')
                UrbanRuralCode = i.xpath('td[2]/text()')
                villageName = i.xpath('td[3]/text()')
                #上面得到的是列表形式的，下面将其每一个用字典存储
                for j in range(len(villageCode)):
                    town.append({'code':villageCode[j],'UrbanRuralCode':UrbanRuralCode[j],'name':villageName[j]})
                
    def run(url_list):
        produce_url(url_list)
    
        ths = []
        for _ in range(thread_num):
            th = Thread(target = getData)
            th.start()
            ths.append(th)
        for th in ths:
            th.join()
            
    run(url_list)
    return town
village = getVillage(df_town['link'])
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/14/07/24/140724204.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/14/07/27/140727400.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/14/10/29/141029204.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/15/01/04/150104008.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/14/09/81/140981102.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/15/01/02/150102001.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/14/09/81/140981210.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/15/04/21/150421202.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/15/04/25/150425100.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/15/04/22/150422401.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/15/04/02/150402402.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/15/04/30/150430207.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/15/01/21/150121105.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/15/07/22/150722105.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/15/25/26/152526103.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/21/04/21/210421209.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/21/04/22/210422108.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/21/05/02/210502002.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/21/06/03/210603007.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/21/05/02/210502010.html
requests fail, retry!
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/21/05/03/210503005.html
requests fail, retry!
```

由于数据量很大，所以这里我没有爬取完毕。