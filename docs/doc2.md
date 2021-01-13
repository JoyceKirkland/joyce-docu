---
id: doc2
title: 深度相机代码学习（持续更新
---
## 1.9

1、测距代码原理：小孔成像

2、摄像头帧率上限就30（……

3、Q：为什么要从深度图对齐到彩色图？  

A：因为彩色图看到的东西和深度图的像素不一样。深度图看到的东西更多更接近实际，因此要将深度图的像素对准到彩色图才是正确的数据处理。

![](https://image-up-1304421499.cos.ap-guangzhou.myqcloud.com/img/20210109153039.jpg)

---
## 1.12

（基于工程辅助对位的学习，将侧重点放在了与“扣图”有关的代码学习中）

1、删除背景（图像对齐的运用，即rs-align-advanced）初步框出了矿石，但是非常不稳定，显示颜色奇怪，而且背景杂质巨多。

但是好歹终于有点希望了呜呜呜呜呜呜

下一步就是在examples看看还有没有更合适的代码。

2、代码https://github.com/JoyceKirkland/rm/tree/master/Realsense

![](https://image-up-1304421499.cos.ap-guangzhou.myqcloud.com/img/20210112222607.png)

（落泪，我效率好低
