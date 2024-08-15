---
toc: true
#toc_number: 1
title: unity杂项研究
date: 2024-08-14 13:05:00
cover: /img/cover/bq.png
tags:
    - unty
---

# 🎲获取两个物体之间的最短距离
ClosestPoint 简介：  
1.获取两个物体表面最短距离可以使用Unity自带的ClosestPoint API
应用场景：  
比如做一个潜水器的寻缆功能，需要检测潜水器和线缆之间的最短距离，而线缆是弯弯曲曲的很长的一条不太好直接计算，就需要用到这个  
效果展示：  
![1](1.gif)  
代码：
```
    public GameObject A1;
    public GameObject A2;

    // Update is called once per frame
    void Update()
    {
        
        Debug.DrawLine(
            //返回A1点到A2collider表面最近的一个点
            A2.GetComponent<Collider>().bounds.ClosestPoint(A1.transform.position), 
            A1.GetComponent<Collider>().bounds.ClosestPoint(A2.GetComponent<Collider>().bounds.ClosestPoint(A1.transform.position)), 
            Color.red
            );
    }
```

# 🎲获取两个物体之间的最短距离
