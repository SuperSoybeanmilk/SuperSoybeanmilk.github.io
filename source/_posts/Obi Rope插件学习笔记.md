---
toc: true
#toc_number: 1
title: Obi Rope插件学习笔记
date: 2024-06-17 15:07:00
cover: /img/cover/bq.png
tags:
    - unty
    - Obi Rope
    - 绳子
---

本篇文章参考自[Obi官方文档](http://obi.virtualmethodstudio.com/tutorials/index.html)

# 🎲Obi Rope Blueprint

## 🎮Rope Blueprint简介
Rope Blueprint根据官方的说法就是使用距离(Distance)和弯曲约束(Bend Constraints)通过连接粒子建立的。由于一般情况下粒子没有方向（只有距离），因此无法模拟扭转效果，并且绳索不能保持其静止的形状。绳索能扭转/分裂并且运行时长度可变，使用方法会在下方Obi Rope讲到。

另外在官方文档下还有一个Rod Blueprints这个就不多做介绍了简单的说一下就是使用伸缩(Stretch)和弯曲约束通(Bend Constraints)过连接有向粒子建立的，这个比绳索要更复杂，可以模拟弯曲静止的效果，所以叫杆，但是这个比起绳子(Rope Blueprint)来说不可以撕裂/拆分，长度也不能在运行时更改，所以一般用来做弹簧，天线等。  
![蓝色是杆子，红色是绳子](ropes.gif)

## 🎮Rope Blueprint属性
如果要创建一个基础的Rope Blueprint/Rod Blueprint，通过在Project窗口右击→Create→Obi→Rope blueprint/Rod blueprint.  
![1](1.jpg)

### __Thinckness（厚度）__  
生成绳子的粒子半径

### __Resolution（分辨率）__  
单位长度粒子的密度。_值为1_ _值为0.5_ 的话就会比较稀疏不会产生接触，_值小于0.5_ 的话会产生很大的间隔，这会对碰撞检测产生很大的影响，但是优点是性能很好。  
![2](2.jpg)

在内部，用于根据分辨率计算绳索中粒子数量的公式为 particleCount = ropeLength / ropeThickness * resolution.  
如果在运行时创建绳索，可以使用以下公式计算所需的分辨率，以获得绳索中特定数量的粒子 resolution = particleCount / (ropeLength / ropeThickness)

### __Pooled Particles（汇集粒子，仅限Rope Blueprint）__  
这个我个人的理解就是额外的粒子，比如在你做的一个吊机，他的绳子是可以伸缩的，当你增加绳子长度而产生额外粒子的时候就是从这个里面增加的，如果你只是设置了10那么等你去增加绳子长度的时候只能增加一点，加完10个粒子的绳子长度后绳子长度就不会改变了。所以如果绳子静止的情况下可以改为0就行，需要做缩放的绳子效果时就可以把这个值给高一点。

# 🎲Obi Solver

## 🎮Obi Solver简介
Obi Solver是Obi中最重要的组件，它执行模拟粒子物理，和执行约束的组件。关于这个有3点比较重要：  
他们可以添加到场景中的任何GameObject中，并且可以有多个Obi Solver 
 
每一个Actor必须在Obi Solver的层级下才可以执行，这些Actor物理模拟的具体行为和组件比如下面会提到的Obi Rope，或者布料(Obi Cloth)这些都是Obi Actor的派生类。  

不同的Obi Solver相互独立，所以他们的Actor不会相互碰撞，只有同一Obi Solver下的Actor才会对彼此作出反应

个人理解Obi Solver就像是一个房子里面住的人就是Actor一栋楼里面可以有多个房子，每个房子互不影响。

## 🎮Obi Solver属性
如果要创建一个Obi Solver可以在Hierarchy里右击→3D Object→Obi→Obi Solver.
![3](3.jpg)

### __Backend（后台）__  
这个算是一个物理效果实现的底层，不同的Backend有不同的平台兼容性，性能和功能。场景中的每个Obi Solver都可以选择不同的Backedn属性，但是大部分情况下还是保持一致最好。目前的话有两个一个是Oni一个是Burst.  
#### _Oni_   
是obi一个比较早的后台，用的是c++编写的本地库，所有大部分平台都支持，他是完全基于cpu的后台使用了多线程和单指令多数据的内嵌函数。  
#### _Burst_   
这个是Obi 5.5后的新有的后台，它是由睾贵的C#编写，用的是unity的Burst编译器和作业系统，它支持所有可以运行作业和Burst编译器可以编译的平台。与Oni一样，它完全基于CPU，并大量使用多线程和SIMD。使用Burst的时候大部分情况下新能要比Oni强，所以一般情况下Burst的优先级要比Oni高。  

当然使用Burst的话要导入一些包  
>Burst 1.3.3 或以上  
Collections 0.8.0-preview 5 或以上  
Mathematics 1.0.1 或以上  
Jobs 0.2.9-preview.15 或以上

上面的包 Burst应该是自带，Collectionsll在unity自带的包里面也有导入就行 Mathematics也是自带，最后Jobs因为是预览版本所以要用链接导入（unity 2021以及后的版本）__com.unity.jobs__  

![6](6.jpg)  
<font color='red'>
请注意，为了使用Burst后端编辑器时的正常性能，必须启用Burst编译并禁用安全检查，作业调试器和作业泄漏检测  
</font>
![4](4.jpg) ![5](5.jpg)  
<font color='red'>
另外，请记住Burst默认情况下在编辑器中使用异步编译。这意味着模拟的前几帧将明显较慢，因为Burst在场景运行时仍在编译作业。您可以在Jobs→Burst菜单中启用同步编译，这将强制Burst在进入播放模式之前编译所有作业。
</font>

### __Mode（模式）__ 
Solver可以在2D或者3D模式下模拟，2D的话模拟仅发生在2D场景中，只会发生在XY的平面中。2D模式通常与2D对撞机的使用相结合。但是，为了方便起见，你也3D 2D一起使用。

### __Interpolation（插值）__ 
用于渲染粒子的插值模式。None计算速度更快，但是Interpolate会在粒子的实际物理位置和渲染位置之间插入1帧的延迟。Interpolate在慢动作效果的时候会更好,因为他能在unity的Timescale<1时避免卡顿。

### __Gravity Space（重力空间）__  
是在Solver的局部空间(Self)还是在世界空间(World)中应用重力，如果设为Self，重力方向将和Solver一起旋转。

### __Gravity（重力）__  
Solver重力的方向和大小，以Slover的局部空间表示

### __Sleep Threshold（休眠阈值）__  
任速度能低于该值的粒子都将被冻结在当前位置。一般来说默认值就够了。

### __Damping（阻尼）__  
应用于粒子速度的速度阻尼。增加它以减少它们的动能。下图分别为0.3阻尼和1阻尼时的效果
![阻尼为0.3时](Damping0.3.gif)  ![阻尼为1时](Damping1.gif)

### __World linear Inertia Scale（世界线性惯性标度）__  
控制Solver变换的世界空间线性惯性应用于粒子的程度。值的范围从0（无）到1（100%）。
大概意思就是你对这个物体用的力有多少会继承到他这个Solver上，官方的说明不是很清楚这边上两组图片对比一下，前面是设为0时的状态后面是设为1的状态<font color='red'>（注意看左边垂直的绳子的效果）</font>  
![值为0时](WLIS0.gif)  
![值为1时](WLIS1.gif)  

### __World Angular Inertia Scale（世界线性惯性标度）__ 
跟前面的属性是一样的，只不过这个前面那个是线性惯性(translations)现在这个是角惯性(rotations)

### __Max Anisotropy（最大各向异性）__
这个一般用在流体中，Obi中的流体粒子可以是椭圆形的，而不是完全的球形。这用于更好地使其形状适应所代表对象的表面，从而实现更准确的碰撞检测和更平滑的渲染。最大各向异性可以确定椭球体半径之间的最大比率，值为1将强制所有粒子为球形，值大于1的时候就会更椭圆一点
![7](7.jpg) 

### __Simulate When Invisible（不可见模拟）__
当Solver在所有相机中不可见时是否应该保持模拟。保持启用你的模拟将会不停更新，禁用当没有任何相机看到的时候就会停止更新。

### __Continuous collision detection (CCD)__





