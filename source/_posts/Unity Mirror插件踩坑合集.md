---
toc: true
#toc_number: 1
title: Unity Mirror插件踩坑合集
date: 2024-06-12 15:10:00
tags:
    - hexo
    - butterfly
---

# 🎲报错

## 🎮Mirror客户端找不到服务端并且报找不到物体的错
![1](1.png)
先在cmd里面用ipconfig找到自己ip然后把NetworkManager里面的NetworkAddress改成自己的IP，如果用自带的localhost就会报这个错并且找不到服务端（这个问题卡了我半天，最后灵机一动试了一下就成功了）。
![2](2.png)