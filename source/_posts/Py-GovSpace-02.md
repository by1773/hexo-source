---
title: 国家统计局统计用区划代码和城乡划分代码爬虫(一)
date: 2020-03-21 08:46:52
tags: ['码爬']
categories: 码爬
keywords: ['码爬','python']
top_img: https://goss.cfp.cn/creative/vcg/nowarter800/version2/gic11136091.jpg?x-oss-process=image/format,webp
cover : https://goss.cfp.cn/creative/vcg/nowarter800/version2/gic11136091.jpg?x-oss-process=image/format,webp
---

## 国家统计局统计用区划代码和城乡划分代码爬虫 (一)--分析



这里我就拿 2016 年的页面做下分析：[2016 年统计用区划代码和城乡划分代码 (截止 2016 年 07 月 31 日)](http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/index.html)。

## 一、省级页面分析

### 1、省级信息提取

我们进入到 [2016 年统计用区划代码和城乡划分代码 (截止 2016 年 07 月 31 日)](http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/index.html) 这个页面，然后用 chrome 的 “检查” 工具看下我们要找的信息在哪。

这里我们需要爬取省级名称、省内市级信息的子链接这两个参数。

我们从图中可以发现，左边页面每一行对应的 XPath 路径为：

```python
//tr[@class="provincetr"]
```

然后一行中每个省的信息在下一级的 td 标签内：

```python
td/a/text()
td/a/@href
```

![省级页面分析](https://tding.top/archives/a4d70246/%E7%9C%81%E7%BA%A7%E9%A1%B5%E9%9D%A2%E5%88%86%E6%9E%90.png)

### 2、下级链接获取

省级页面的 URL：

```python
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/index.html
```

下级页面的 URL（我这里以`浙江省`为例）：

```python
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/33.html
```

页面中提取到的信息（我这里以`浙江省`为例）：

```python
33.html
```

所以我们可以通过如下方式获取真实的 URL 保存到一个列表中：

```python
url = "http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/index.html"
# provinceLink = "33.html"
provinceURL = url[:-10] + provinceLink
```

## 二、市级页面分析

### 1、市级信息提取

我们进入到[浙江省](http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/33.html)中。具体的分析跟上面的省级页面分析类似，不再赘述。下面是市级页面分析图：

![市级页面分析](https://tding.top/archives/a4d70246/%E5%B8%82%E7%BA%A7%E9%A1%B5%E9%9D%A2%E5%88%86%E6%9E%90.png)

### 2、下级链接获取

市级页面的 URL：

```python
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/33.html
```

下级页面的 URL（我这里以`杭州市`为例）：

```python
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/33/3301.html
```

页面中提取到的信息（我这里以`杭州市`为例）：

```python
33/3301.html
```

## 三、区级页面分析

### 1、区级信息提取

我们进入到[杭州市](http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/33/3301.html)中。具体的分析跟上面的省级页面分析类似，不再赘述。下面是区级页面分析图：

![区级页面分析](https://tding.top/archives/a4d70246/%E5%8C%BA%E7%BA%A7%E9%A1%B5%E9%9D%A2%E5%88%86%E6%9E%90.png)

### 2、下级链接获取

区级页面的 URL：

```python
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/33/3301.html
```

下级页面的 URL（我这里以`上城区`为例）：

```python
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/33/01/330102.html
```

页面中提取到的信息（我这里以`上城区`为例）：

```python
01/330102.html
```

## 四、街道页面分析

### 1、街道信息提取

我们进入到[上城区](http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/33/01/330102.html)中。具体的分析跟上面的省级页面分析类似，不再赘述。下面是街道页面分析图：

![街道页面分析](https://tding.top/archives/a4d70246/%E8%A1%97%E9%81%93%E9%A1%B5%E9%9D%A2%E5%88%86%E6%9E%90.png)

### 2、下级链接获取

街道页面的 URL：

```python
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/33/01/330102.html
```

街道页面的 URL（我这里以`湖滨街道`为例）：

```python
http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/33/01/02/330102003.html
```

页面中提取到的信息（我这里以`湖滨街道`为例）：

```python
02/330102003.html
```

## 五、居委会页面分析

### 1、居委会信息提取

我们进入到[湖滨街道](http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2016/33/01/02/330102003.html)中。具体的分析跟上面的省级页面分析类似，不再赘述。下面是居委会页面分析图：

![居委会页面分析](https://tding.top/archives/a4d70246/%E5%B1%85%E5%A7%94%E4%BC%9A%E9%A1%B5%E9%9D%A2%E5%88%86%E6%9E%90.png)

这里已经到了最底层，没有下级链接了。