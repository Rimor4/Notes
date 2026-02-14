---
url: https://zhuanlan.zhihu.com/p/687084876
title: UE5 动画技术详解 - FootPlacement
author: zhuanlan.zhihu.com
aliases: 
 - 
date: 2026-02-04 12:55:12
tags:

banner: "https://pic2.zhimg.com/v2-f5345707268de7e822fd2d54eb6b8987_r.jpg"
banner_icon: 🔖
---
![](https://pic2.zhimg.com/v2-f5345707268de7e822fd2d54eb6b8987_r.jpg)

## 简介

[FootPlacement](https://zhida.zhihu.com/search?content_id=240815508&content_type=Article&match_order=1&q=FootPlacement&zhida_source=entity) 是 ue5 新增的一个动画节点，用来取代之前的 [SlopeWarping](https://zhida.zhihu.com/search?content_id=240815508&content_type=Article&match_order=1&q=SlopeWarping&zhida_source=entity)。具体作用是给有腿的角色提供自动的脚步锁定，可靠的盆骨控制，IK 的预计算，并且自动适配地形。跟 SlopeWarping 一样，使用了大量的弹簧插值来使角色的动作更加平滑。

这里每一个功能点，都可能有人做过（ALS 里面就有简化版的 FootLock 和 [IK 预计算](https://zhida.zhihu.com/search?content_id=240815508&content_type=Article&match_order=1&q=IK%E9%A2%84%E8%AE%A1%E7%AE%97&zhida_source=entity)），但是 UE5 把他们都整合在一起了，你只需要一个节点，这些功能就帮你全做了。

当然，与之相对应的，是这个节点的代码量差不多有 2000 行（毕竟是把其他几套功能都写一起了）。但是只要理解了他的功能和原理，阅读源码起来一点也不难。

## 用法简述

![](https://pic3.zhimg.com/v2-ae3a653ec531b2f33f09c1082330308c_r.jpg)

首先我们要设置 [PlantSpeedMode](https://zhida.zhihu.com/search?content_id=240815508&content_type=Article&match_order=1&q=PlantSpeedMode&zhida_source=entity)，设置为 Manual 模式需要你自己把速度 k 到动画上，让节点读取。否则还是建议用 Graph 模式让节点自动根据位移来算速度。

然后需要设置脚的骨骼 (foot)，脚趾骨骼（ball），盆骨(pelvis)，后面用到的髋关节(hip) 他会自己根据 NumBonesInLimb 去往上获取。

然后设置 PelvsSetting 的 [==PelvisHeightMode==](https://zhida.zhihu.com/search?content_id=240815508&content_type=Article&match_order=1&q=PelvisHeightMode&zhida_source=entity)，是具体从哪个腿计算盆骨高度。

然后就是 PlantSetting 的 [==LockType==](https://zhida.zhihu.com/search?content_id=240815508&content_type=Article&match_order=1&q=LockType&zhida_source=entity)，确定脚步锁定的时候锁定脚趾，脚跟，还是旋转。（我写 footlock 的时候就没写那么详细）然后由于他自动计算脚步锁定，所以你要在 PlantSetting 里面填一下脚趾离地的高度，还有锁定解锁的速度阈值和角度阈值等。提示也很足够。

然后插值设置，追踪设置等就略过不讲了，用过的人都知道啥意思。

## 原理简述

FootPlacement 的各个部分原理，其实之前很多人都可能实现过了。

比如 FootLock，首先他会根据速度和传入 FK 的高度来决定一个脚要不要处于锁定状态，锁定的 Alpha 是多少。这个很好理解，毕竟之前的计算都在平地，如果平地的脚掌离地过近的话，就可以视为踩地状态，需要世界空间的锁定。然后通过形状追踪找到地面，把脚对应地放在当前的地面上。

计算盆骨位置用到了 rune skovbo johansen 的论文 [[1]](#ref_1)。这篇论文也是老熟人了，Half-Life: Alyx 的 Siggraph 文章 [[2]](#ref_2) 也用到了他的一些方法。

下面我们展开说说他的完整处理流程。

## 工作流程

FootPlacement 的工作流程主要分为 5 个部分。

### 收集 FK 状态

在这个过程中，FootPlacement 会收集输入的骨骼状态（下面简称 FK），还有角色的运动状态，是否着地和移速等等。

### 处理脚

这里主要是由`ProcessFootAlignment`函数来处理。

首先，要确定脚掌的移动状态来决定要不要实行锁定。这里锁定的规则很简单，脚掌位置低于设定值，然后速度也低于设定值，就视为需要锁定。但是这锁定不是无条件的，当位移超了、角度超了或者速度超了，锁定会结束掉。当条件合适的时候，锁定会再次激活。

类似的逻辑可以在 [Daniel Holden](https://zhida.zhihu.com/search?content_id=240815508&content_type=Article&match_order=1&q=Daniel+Holden&zhida_source=entity) 的这篇文章 [[3]](#ref_3) 里找到。估计很多人也自己写过类似的逻辑，然后再加一个临界弹簧，构成简单的 Footlock 节点。但是，这只是 FootPlacement 的很小的一部分。

确定锁定状态后，如果锁定的话，会==根据设置的锁定方式 (LockType) 来决定用什么方式锁住脚（脚趾，脚跟，固定旋转），然后算出一个 Foot 关节的世界坐标位置；非锁定的话，就直接进行位置插值==。

这里会输出一个`UnalignedFootTransformWS`，表示这是没跟地面对齐的转换 (Transform)。

然后尝试找到地面，FootPlacement 会对每只脚进行一次简单碰撞 Sweep 和一次复杂碰撞 Sweep，简单碰撞主要是为非触地时候，因为简单碰撞包围着复杂面，锁定状态则使用复杂碰撞来更好地贴地。==找到地面后，根据法线决定腿部的位置和旋转，然后把 Twist 按比例移除掉==。

这里会产生一个`AlignedFootTransformWS`，表示跟地面对齐过的转换。(`AlignPlantToGround`)

### 处理盆骨

根据计算出来的这些脚的位置，计算盆骨位置。(`SolvePelvis`)

首先会尝试把盆骨挪到两腿重心之间，避免倾侧。(`HorizontalRebalancing`)

然后根据每条腿腿的位置和 FK 的状态，计算各自的合适的盆骨偏移，然后平均出来作为盆骨位置。

这一段原理来自 Automated Semi‐Procedural Animation for Character Locomotion [[1]](#ref_1) 的 7.4.2 节，说实话每次看这篇论文都会吐不少血，内容太多了。

![](https://pic3.zhimg.com/v2-3673ded6962f01034fc498136d6032d8_r.jpg)

从原理上，这部分操作主要是找到一个极限拉伸的点 Q，和一个保持原来弯曲程度的点 P。当然也很简单，只要在新的位置画同心圆，找到和髋关节的垂线交点即可。然后最终的盆骨位置会根据所有腿的这个点根据一定规则处理。具体可以看论文细节。

最后会对盆骨位置进行弹簧插值，保证核心稳定。

### 修正腿长度

最后一步是修正腿的长度，避免腿过度拉伸 (`FinalizeFootAlignment`)。

这里的处理很细节，会把脚尖绷直来获取一点额外长度，不行再往回拉。后面一堆细节处理，看得出作者遇到过各种奇怪的形态，打了不少补丁。

经过处理后得出最终的脚的位置，写入 IK 节点，至此工作结束。

## 其他建议

目前来看这个节点消耗还是略高的，如果用在手机上，worker 线程每只脚每帧 2 次形状追踪消耗不能忽视，当然我们自己可以把他改成 Lazy + 异步 ik，降低线程消耗。当然，如果是主机或者 pc 游戏，这点消耗也算不上什么。

如果你只是想当做 SlopeWarping 来使用的话，可以修改`FindPlantTraceImpact` 函数，在 `!TraceSettings.bEnabled`的时候设置`OutImpactNormalWS = Context.GetMovementComponentFloorNormal();` 这样只要关掉 TraceSettings 的开关，就可以跟 Slopewarping 一样只靠移动组件的地面法线驱动了。

从注释上看，作者之后会添加预测等功能，但是不知道会不会实装到正式版里。当然，如果要自己添加的话，也比较简单，只需要把脚在空中的时候，放置在当前与预测点之间连线的平面上即可。就是消耗可能会额外高一点。（不过你都走预测了，应该可以接受）

如果需要移植到 UE4，需要把 UE5 的 Spring 插值系统一起移植过来，因为 UE5 的弹簧插值系统重写了，整个表现差异非常大。具体原因可以参考这篇：[sincezxl：UE4 中的阻尼弹簧平滑](https://zhuanlan.zhihu.com/p/412291354) 。

## 结语

得益于这个节点代码的大量注释，可以轻松理解作者的思路。

毫无疑问这应该是 UE5 用来打包之前的一大堆解决方案的节点，但是从 Daniel Holden 的文章来看，这个节点会更好地服务于 MotionMatch 系统，可以大幅度地减少系统的滑步，和地形穿透（当然代价是动作没那么自然）。有空的话可以另开一篇更详细地展开讲一下里面的各部分逻辑和技巧。

## 参考

1.  ^[a](#ref_1_0)[b](#ref_1_1)Automated Semi‐Procedural Animation for Character Locomotion [https://runevision.com/thesis/rune_skovbo_johansen_thesis.pdf](https://runevision.com/thesis/rune_skovbo_johansen_thesis.pdf)
2.  [^](#ref_2_0)The Right Foot in the Wrong Place: Half-Life Character LocomotionCharacter Locomotion in Half-Life: Alyx [https://dl.acm.org/doi/10.1145/3450623.3469664](https://dl.acm.org/doi/10.1145/3450623.3469664)
3.  [^](#ref_3_0)https://theorangeduck.com/page/code-vs-data-driven-displacement#footlocking [https://theorangeduck.com/page/code-vs-data-driven-displacement#footlocking](https://theorangeduck.com/page/code-vs-data-driven-displacement#footlocking)