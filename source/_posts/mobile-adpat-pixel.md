---
title: H5移动端适配
tags: 适配
categories: 移动端
cover: >-
  https://goss2.cfp.cn/creative/vcg/nowarter800/new/VCG211246652444.jpg?x-oss-process=image/format,webp
top_img: >-
  https://goss2.cfp.cn/creative/vcg/nowarter800/new/VCG211246652444.jpg?x-oss-process=image/format,webp
abbrlink: 11762
date: 2020-04-08 10:00:00
---
# @media H5移动端屏幕适配

## 添加配置

```
name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=no"
```

> - viewport ：用户网页的可视区域.
> - width：控制viewport的大小，可以指定一个值，如600，或者特殊的值，如device-width为设备的宽度（单位为缩放100%时的CSS的像素）。
> - height：和 width 相对应，指定高度。
> - initial-scale：初始缩放比例，也即是当页面第一次 load 的时候缩放比例。
> - maximum-scale：允许用户缩放到的最大比例。
> - user-scalable：用户是否可以手动缩放。
## 移动端手机尺寸

### 1. phone4/iphone5/SE(width是一样)

```scss
// 特殊场景，写法不一，比如：
1. @media screen and (max-width:320px ){}
2. @media screen and (max-height:480px ){}
3. @media screen and (max-width:320px ) and (max-height:480px ){}
```

### 2. iphone6/7/8

```scss
// 有时候我们只需要获取width 或者height 其中一个即可
1. @media screen and (max-width:375px ){}
2. @media screen and (max-height:667px ){}
```

### 3. iphone6/7/8 Plus

```scss
1. @media screen and (max-width:414px ){}
2. @media screen and (max-height:736px ){}
```

### 4. iPhone XS: 5.8 英寸

```scss
@media only screen and (device-width: 375px) and (device-height: 812px) and (-webkit-device-pixel-ratio: 3){}
```

### 5. iPhone XR: 6.1 英寸

```css
@media only screen and (device-width: 414px) and (device-height: 896px) and (-webkit-device-pixel-ratio: 2){}
```

### 6. iPhone XS Max: 6.5 英寸

```css
@media only screen and (device-width: 414px) and (device-height: 896px) and (-webkit-device-pixel-ratio: 3){}
```

## 获取屏幕尺寸

```dart
// 屏幕可以大小
screen.availWidth+"/"+screen.availHeight
// 屏幕分辨率
screen.width+"/"+screen.height
// 网页可见区域
document.body.clientWidth +"/"+ document.body.clientHeight
// 网页可见区域(包括边线的宽)
document.body.offsetWidth +"/"+ document.body.offsetHeight
```