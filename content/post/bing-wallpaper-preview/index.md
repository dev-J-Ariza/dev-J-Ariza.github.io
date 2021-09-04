+++
author = "J-Ariza"
title = "Bing壁纸预览及下载"
date = "2021-09-04"
description = "一个可以预览以及下载Bing壁纸的桌面程序"
categories = [
    "Python",
    "Python实践"
]
tags = [
    "Python",
    "Python实践"
]
image = "screenshot.png"
+++

# 简介

这是一个桌面端的程序，可以浏览Bing中文区官网上最近7天的壁纸，并且提供了下载高清版壁纸的功能。

项目地址： https://github.com/dev-J-Ariza/BingWallpaperPreview

# 实现

## 技术栈

使用Tkinter开发gui，使用requests进行网络请求。

## 工程结构

- main.py - 程序的入口，运行主函数即可。
- endpoint.py - 网络请求的工具类，主要负责请求Bing壁纸，并对返回结果进行解析。
- gui.py - 提供下载、前后查找功能，用户可以一张张翻看最近7天的壁纸，并且下载。

## 源码分析

首先我们先看负责网络请求的endpoint.py，
1. 去哪获取Bing壁纸

https://cn.bing.com/HPImageArchive.aspx?format=js&idx={cur}&n=1&mkt=zh-CN 这个就是官方提供的接口，用来获取要查找的壁纸的url。
也就是说，请求这个接口，得到的结果是一个指向壁纸的url。 其中cur=0表示请求今天的， cur=1表示请求前一天的，以此类推。

2. 如何解析出图片

当获取到壁纸url后，可以通过requests发出请求得到图片的数据，然后写到io管道里面。
```python
def get_single_image_stream(img_url):
    image_bytes = requests.get(img_url).content
    data_stream = io.BytesIO(image_bytes)
    return data_stream
```


接着我们再看用户图形界面部分
3. gui组成

我们使用Tkinter来编写gui部分， Window表示某个应用程序的窗口，是Tkinter里面用来承载一切组件的基础，所以最底层是Window。
在这之上我们就直接展示壁纸，使用Canvas组件负责展示，所以第二层是Canvas。
接着，需要为用户提供操作按钮，方便用户查看前一张/后一张/下载图片，所以需要用一个Frame来盛放各个按钮，这个frame在canvas之上。
最后，就是各个具体按钮。

Window -> Canvas -> Frame -> Button

4. 如何展示出壁纸

Canvas是画布的意思，可以用来在屏幕上的某个区域展示图形，这个图形可以是纯色背景，也可以是各种格式的文件。 这点很想Android中的Canvas，
Android.Canvas提供了各种drawXX()方法，用户还可以自己去画出各种形状。
```python
canvas.create_image(450, 300, image=get_image(image_url))
```
get_image返回的是一个图片控件，放在Canvas里，内容是image_url指向的壁纸。

5. 下载

下载的本质就是写文件的IO操作，需要指定保存路径，并且需要数据内容。 保存路径通常是用户自己决定的，所以提供一个对话框，用户可以指定保存
路径。这个Tkinter有现成的方法，`filedialog.asksaveasfile()` 可以参考文档使用。
数据内容也好说，就是接口请求url后，得到的放回数据。

真正保存图片的写操作python也提供了封装好的方法， `image.save(path)`.

最后，使用缓存提高效率

6. 缓存的使用

当用户前后查找图片时，假设当前图片是A，前一张是B，用户可能的操作路径是 A-B-A。 那么当用户反复横跳时，如果能将已经请求过的图片缓存下来，
就可以避免重复的网络请求，省时省流量。

这里的解决方案就是python自带的缓存框架，LRU_Cache. 我们以每张图片的url作为缓存的key， 壁纸图片作为value。这样当遇到相同的url时，
就可以直接从缓存中取出图片并展示了。

# 感受

写点python脚本还是挺有意思的，而且在日常挺有用的，比如这个Bing下载图片的脚本、之前写的公交实时查询。 之后还会再找点日常有用的功能，
写成脚本。