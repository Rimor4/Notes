---
url: https://zhuanlan.zhihu.com/p/552056254
title: 《刺客信条》基于预测的 FootIK 方案分析与 UE4 实现
author: 左加右
aliases:
  - 
date: 2026-02-04 12:47:38
tags:
banner: https://pic4.zhimg.com/v2-ce18b0f2449a8204d6e148885fefa0a1_r.jpg
banner_icon: 🔖
---
_2016 年的 [GDC](https://zhida.zhihu.com/search?content_id=210812013&content_type=Article&match_order=1&q=GDC&zhida_source=entity)，育碧的大佬分享了一项在刺客信条上应用的 [FootIK 方案](https://zhida.zhihu.com/search?content_id=210812013&content_type=Article&match_order=1&q=FootIK%E6%96%B9%E6%A1%88&zhida_source=entity)，也就是基于预测的 FootIK。_

## 前言

在分析基于预测的 FootIK 之前，先简单了解一下 FootIK 的概念。

假设我们有一个动画片段是令角色行走在地面上。在 Maya 中，该片段看起来非常好；但在游戏场景中，由于地面不是完全平坦的，导致有时候角色的脚会飘在空中，有时候又会与地面交叉。 在这种情况下，我们希望可以调整骨骼的最终姿势，令角色的脚能完全地面贴齐。名为[逆运动学](https://zhida.zhihu.com/search?content_id=210812013&content_type=Article&match_order=1&q=%E9%80%86%E8%BF%90%E5%8A%A8%E5%AD%A6&zhida_source=entity)（inverse kinematics，IK）的技术可以达成此事。

IK 的话，我们可以这么理解。正常情况下，骨骼动画中一个关节的位置是由其父关节的变换信息和其局部坐标来确定的。这也是[正向运动学](https://zhida.zhihu.com/search?content_id=210812013&content_type=Article&match_order=1&q=%E6%AD%A3%E5%90%91%E8%BF%90%E5%8A%A8%E5%AD%A6&zhida_source=entity)（forward kinematics，FK）的概念。但是 IK 与其相反，是先确定子关节的变换信息，再来逆推出父关节的变换信息。

例如 FootIK，一般是先根据地形信息确定 [Pelvis 关节](https://zhida.zhihu.com/search?content_id=210812013&content_type=Article&match_order=1&q=Pelvis%E5%85%B3%E8%8A%82&zhida_source=entity)与 Foot 关节变换信息。再来计算其他受影响的关节变换信息。

## 传统 FootIK 介绍

传统的 FootIK 思路比较简单。主要是以下 5 个阶段。首先初始化一些数据，例如 Foot 脚的原始高度，盆骨高度。第二步就是两只脚都打射线。取交点最低的 Foot 作为 Pelvis 下移的参考脚。第三步就是 Pelvis 关节下移。第四步就是算出 FootIK 的偏移信息和 Foot 旋转。最后就是 IK 的解算。

![](https://pic4.zhimg.com/v2-ce18b0f2449a8204d6e148885fefa0a1_r.jpg)

下面虚幻商城中的两个经典 Demo 使用的就是这套方案。

![](https://pic4.zhimg.com/v2-5ef2d242c73964cd0ed562a45dfd1ed1_r.jpg)

这套方案的缺陷就是对于地形变化的敏感度比较高。斜坡或低台阶类型的地形，这套方案跑的没问题。但是碰到高低突变的地形，表现就会比较差。先看下下面这个 Echo 的视频。

可以发现角色会发生抖动。原因是因为传统 FootIK 是一种即时响应式的方案，当前帧检测地形，当前帧调整 Pelvis 高度。这样的话，碰到上面视频中的地形，就会出现抖动情况。这种情况就算是加了插值也没办法解决。

## 基于预测的 FootIK 介绍

2016 年的 GDC，育碧的大佬分享了一项在刺客信条上应用的 FootIK 方案，也就是基于预测的 FootIK。下面是育碧大佬在介绍该方案时的几个说明。

1.  效果比传统响应式 IK 好。
2.  更还原动画的原始表现。
3.  从生物力学得到的灵感。更符合双足生物行走的习惯。

[Fitting-the-World-A-Biomechanical](https://link.zhihu.com/?target=https%3A//www.gdcvault.com/play/1023009/Fitting-the-World-A-Biomechanical)

![](https://pica.zhimg.com/v2-75d6ef39ca9d28bf8dce8f6b21141980_r.jpg)

更好的效果当然会匹配更复杂的逻辑。基于预测的 FootIK 在实现上相较于传统 FootIK 思路完全不一样。而且过程也较复杂。下面就我对这个方案的理解做一个简单的步骤拆分。首先是动画数据预处理，在这一步骤需要生成一些预处理数据以供后面的步骤使用，例如 Toe 关节的位置和速度。

![](https://picx.zhimg.com/v2-1c64d69dcacad700237d2e315cf5eb2f_r.jpg)

然后，根据预处理数据预测脚步落点。生成移动曲线。

![](https://pic3.zhimg.com/v2-0023ff29bbb72b89f98247645b39cdca_r.jpg)

再然后，根据移动曲线的高度值来校正 Pelvis 关节。

![](https://pic1.zhimg.com/v2-3069d3cf2940daed0b19c40a3cba0ec0_r.jpg)

接着，根据移动路径进行地形检测，生成地形曲线以校正 Foot 关节。

![](https://pica.zhimg.com/v2-6ac397becf85d5610e717a8515b45226_r.jpg)

当然，在该 GDC 中还讲了很多细节处理，我这里就不一一细数了。我个人感觉，上述步骤是该方案的最小合集。在实现上述功能后可以不断完善细节和表现。而且也不必完全复刻育碧的方案，核心还是在于这种方案的思路。下面这个视频就是 GDC 的效果演示视频。

## 在 UE 中的预测 IK 实现

接下来介绍一下我在 UE 中是如何实现预测 IK 的，在 [Echo 项目](https://zhida.zhihu.com/search?content_id=210812013&content_type=Article&match_order=1&q=Echo%E9%A1%B9%E7%9B%AE&zhida_source=entity)的基础上进行修改。在介绍前，先感谢工作室的某大佬在研究过程种对于我的的支持。同时在这里也分享一篇知乎文章，同样也是预测 IK 在 UE 中的实现（我的实现的思路很大一部分是受益于这篇文章）。大家有兴趣可以看看。

[sincezxl：UE4 实现基于预测的 FootIK](https://zhuanlan.zhihu.com/p/380222928)

### 动画数据预处理

动画数据的预处理，我没有使用 GDC 中介绍的那种方案。使用的是油管的一个视频介绍的方案。

[UE4 Character's Feet Prediction (Using Anim Curves) UE4Tuts For You#UE4tuts#UE4 - YouTube](https://link.zhihu.com/?target=https%3A//www.youtube.com/watch%3Fv%3DuNLeVKRh8Vo)

以某 Foot 离地作为起始，某 Foot 落地且靠近 Root 关节作为结束，生成一条落地剩余时间曲线。这条曲线的意义在哪里呢？意义就在于我可以在当前动画的任意帧知道某 Foot 的剩余落地时间。知道了剩余时间才好进行下一步的落点预测。左右 Foot 都要有这么一条曲线。

![](https://pic2.zhimg.com/v2-968c693206536f84ad437f0fa7fb8d11_r.jpg)

![](https://pic1.zhimg.com/v2-287bbdb5723e1ed5dbecc6cf10dab6ae_r.jpg)

### 脚步落点预测

根据上一步生成的曲线，通过一个公式很轻易的就可以得出某 Foot 的落地时角色水平位置。某 Foot 落地时角色位置 = 角色当前位置 + 速度向量 * 某 Foot 曲线采样值。落地点的垂直位置可以通过碰撞检测来获取。

![](https://pic1.zhimg.com/v2-c5dbc1d58f5d92922e6765f9b72f16da_r.jpg)

### Pelvis 运动路径

以预测的落地点作为运动结束点，以上一个结束点作为起始点生成高度曲线。那么曲线就会有下面 4 种形式，平地、上坡、下坡和跨越。其实平地和跨越对于 Pelvis 运动来说，是一样的。再进一步简化的话，就两种——上坡和下坡。平地也就是幅度很小的上坡或下坡罢了。还有一点要注意的是，**Pelvis 的一段运动路径并不是某一 Foot 的从离地到落地的路径，而是前后两个 Foot 落地点之间的路径**。

![](https://pic3.zhimg.com/v2-2a075a4ad5a9d9a860f1c22302b5af74_r.jpg)

按照刺客信条的思路理解，上坡时在某 Foot 落地后开始采样高度曲线，抬高 Pelvis 高度。下坡时在某 Foot 运动开始时便开始采样高度曲线，降低 Pelvis 高度。

下面这段蓝图的逻辑便是区分上坡下坡类型后，根据曲线插值在开始与结束点之间采样出高度值。

![](https://pic2.zhimg.com/v2-0a55769695419834b8ba5bb8ffd01123_r.jpg)

角色 Pelvis 通过预测曲线运动后，可以保证角色在凹凸不平的地形上较为平缓的运动。

这里需要注意的是我们还需要把相机跟随也加上这个偏移，不然相机还是会抖。还有一点要注意的是，Pelvis 的一段运动路径并不是某一 Foot 的从离地到落地的路径，而是前后两个 Foot 落地点之间的路径。起始点为上一个结束点。

### Foot 运动路径

与 Pelvis 运动一样，Foot 运动也是以某 Foot 预测的落地点作为运动结束点，以上一个结束点作为起始点生成路径，然后进行路径检测。

我这里使用的是一种前后交叉式路径检测法。首先从起始点出发向前打射线，在碰撞点前方一点点从上往下打射线，得到了第一个路径点。然后从终点出发向后打射线，这样前后交叉依次执行，直到没有碰撞为止。

![](https://pica.zhimg.com/v2-b5148e8a5190224cfda8ce7df27aa7a6_r.jpg)

得到了路径列表，我们就可以根据 Foot 曲线数据采样得到当前 Foot 的最低高度。然后对该 Foot 做个 IK 处理。

下方蓝图的逻辑便是根据路径点列表和两端曲线值采样出当前 FootBone 的最低高度。

![](https://picx.zhimg.com/v2-9ea8ffac870fb4903c588343e6c2f633_r.jpg)

### 平滑处理

细心观察，我们会发现角色在上坡或者下坡的时候还是会比较快的。看起来不够流畅。这样的话，我们可以加个弹簧插值。如下图，弹簧插值可以将比较硬的曲线修正的较为柔性。而且 UE4 是支持弹簧插值的。我们直接用就行，但是要调好参数。

![](https://pic3.zhimg.com/v2-c4b9c67e236fd56746676477f1f3eb10_r.jpg)

下方蓝图的逻辑便是根据 Pelvis 目标高度插值出当前高度。

![](https://pic4.zhimg.com/v2-14bc249a27b7d243ef74228508bec6a9_r.jpg)

### 待完善功能

1.  Foot 的旋转。上坡时保持水平，下坡时与地面平行。
2.  Foot 的侵入处理。
3.  特殊地形的处理。例如悬崖边以及跳到空中怎么办。
4.  与其他状态的融合。与停止、攻击、跳跃或落地动画怎么融合在一起。我这里初步的想法是这类型状态继续使用传统 IK。

## 声明

能力有限，欢迎大家指正！**求赞求关注！**