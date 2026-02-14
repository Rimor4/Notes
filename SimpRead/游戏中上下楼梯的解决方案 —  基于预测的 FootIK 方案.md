---
url: https://zhuanlan.zhihu.com/p/719758234
title: 游戏中上下楼梯的解决方案 —  基于预测的 FootIK 方案
author: "\r修勾"
aliases:
  - 
date: 2026-02-04 20:01:18
tags:
banner: https://pic1.zhimg.com/v2-6713b31103f6ba3c7480e625536e75b3_720w.jpg?source=172ae18b
banner_icon: 🔖
---
（最近一直做 [Predict IK](https://zhida.zhihu.com/search?content_id=248076625&content_type=Article&match_order=1&q=Predict+IK&zhida_source=entity) 相关的内容，思来想去还是将自己做的相关的一些内容总结一下，以供读者进行参考。）

碎碎念：之前在 Unity 做过 IK，本次使用 Ue 做完 ik 之后感觉 Ue 动画系统开发的体验还可以，很喜欢 Ue 的动画系统的框架。与 [AnimationNode](https://zhida.zhihu.com/search?content_id=248076625&content_type=Article&match_order=1&q=AnimationNode&zhida_source=entity) 节点编写，体验真的很不错。Unity 动画系统和游戏逻辑的耦合稍微有点高。

Predict IK 的分享有很多，大部分的 Predict IK 的分享文章以实现为主，Predict IK 的思路很简单，复杂度在于迭代解决问题的过程。

本文首先简单讲解了一下 [GDC](https://zhida.zhihu.com/search?content_id=248076625&content_type=Article&match_order=1&q=GDC&zhida_source=entity) 的实现大体过程，然后说一下自己的处理过程，最后将迭代的一些过程展示出来。由于本身预测的 IK 方案虽然是 GDC 分享的，但是毕竟没有开源只是分享思路。所以一些很多的细节要进行反推，所以文中一些实现可能并不是最优的解。

**先看一下 IK 处理方案调研与分析先看看楼梯场景的 IK 可能会出现的问题与具体各个游戏中的处理方案：**

![](https://picx.zhimg.com/v2-56a73e3ab3ce4755a5438ae800cd774d_r.jpg)

// 待 TODO

最后的结果：

演示

### 楼梯 IK 可能出现的问题简析

*   **脚趾穿模**

![](https://picx.zhimg.com/v2-894b7d50ca1474e2eeda56ad6ed9b21f_r.jpg)

脚趾穿模问题解决思路其实还是比较简单的，不管是预测的 IK 方案还是传统的 IK 方案中，我们都只需要在 ball（脚趾）骨骼和 foot （脚踝）骨骼上同时做向下的胶囊体 Sweep 即可。

![](https://pic2.zhimg.com/v2-3299499e90d894d790e1e21fbaf76183_r.jpg)

*   稳定性问题：（包括 foot 和 pelvis ）

foot 稳定性主要在两个方面：其一是脚与地面接触的时候，第二个是脱离地面脚在空中移动的过程。

其一主要是由于如果人物的移动的驱动方式是 [CharacterMovementComponent](https://zhida.zhihu.com/search?content_id=248076625&content_type=Article&match_order=1&q=CharacterMovementComponent&zhida_source=entity) ，而非 RootMontion 的话（其实 RootMontion 驱动的话，在混合的时候也会出现一点但整体不明显），脚在楼梯平面滑动，很有可能会出现位置突变，需要做 IK 锁脚。

其二主要要进行[弹簧插值](https://zhida.zhihu.com/search?content_id=248076625&content_type=Article&match_order=1&q=%E5%BC%B9%E7%B0%A7%E6%8F%92%E5%80%BC&zhida_source=entity)，具体的弹簧插值相关的内容就不详细讲解了，可以参考 Ue5 的弹簧插值相关内容。

pelvis 稳定性如果从传统的 IK 方面上讲主要是涉及相关脚的选择，如下可以看到，白色路径是解算出来的臀部路径，黄色是经过弹簧插值后的结果这个变化还是相当剧烈。

![](https://pic4.zhimg.com/v2-b3b7ea78034c490d5bf2712bf024833f_r.jpg)

常见的选取方式有四种：全部腿，真正固定的重心腿，上坡前腿下坡后腿，下坡后腿下坡前腿。（其实说详细点又可以，开一篇新文章，在这个就不说的太详细了避免喧宾夺主）

上坡（楼梯）后腿下坡（楼梯）前腿，平地双腿是稳定性比较高的一种选取方案。但有时候会出现一些比较奇怪的动力学表现。

*   弹簧插值系数问题

弹簧插值系数主要是来源于一个非常进退两难的地方。弹簧插值的目的是为了平滑优化表现效果，一般来说弹簧插值设置的参数越大，弹簧刚性越大，响应越及时，晃动越剧烈。反之同理。这样会出现什么问题呢？如下图，下图是默认 [footplacement](https://zhida.zhihu.com/search?content_id=248076625&content_type=Article&match_order=1&q=+footplacement&zhida_source=entity) 节点的表现。

![](https://pica.zhimg.com/v2-f19f4fbe3775852ae0cefbf82553e25a_r.jpg)

这并不是解算的位置过低，而是快速上升与下降的过程中，由于弹簧插值，插值后的 pelvis 位置与解算的 pelvis 位置差距比较大。所以上楼梯出现这种过于下压的，在下楼梯的时候会出现一直踩空气的情况。

为什传统 IK 不太适合楼梯这种场景其一是被动 IK 系统由于遇到台阶这种突然抬升，Foot 位置改变往往很突然，而 pelvis 位置往往也是根据 foot 位置进行计算，所以如果要保证动画的流畅性，插值参数要设置的很保守。但是预测的 IK 基本上不存在位置突变，所以插值参数可以设置更大，让响应更及时。

依照这个思路引入动态弹簧插值系数。在 GatherData 阶段的收集运动数据来动态更新计算弹簧的 Stiffness。

还有一些其他的问题比如 FootLock 等等问题是传统 IK 中常见的一些问题，在这里不细谈。

### 预测方案相对于传统方案的优点：

​ 对于动画而言程序化生硬的调整，脚步的位置，旋转这一系列动画参数是非常不自然的。所以要尽量去减少程序化动画的调整，GDC 中的分享中主要表达了如下四个点

*   预测性 IK 比被动的 IK 系统更加的智能

体现在被动 IK 系统是落后的，比如脚前进的方向遇到台阶被动的 IK 系统，会做突然的抬起脚步，这样就很不自然

*   保存原始的运动信息

在 GDC 中表达的意思是，原始的动画数据是在理想平面上的标准动画。即使自然场景中也应该更多保存动画的原始信息，而不是使用计算的 IK 位置。

*   比传统 IK 更正常的人体力学

主要体现在数据上传统 IK 完全不能获取未来的数据即使用上插值也没有用。而尽量不使用 IK 修正，使用更多的是角色动画所以更正常。

*   更优秀的性能表现

预测的 IK 系统，每一帧可以复用先前的预测数据，而不需要每一帧去重建路径。

### GDC 刺客信条基于预测的 IK 的实现过程

![](https://picx.zhimg.com/v2-5b15492fe781b5d148e8303f6d17485b_r.jpg)

其实如果你想要去实现这套系统还是建议去看看 GDC 官方的一个讲解，建议连带着后续的的提问一并观看（后面现场提出了一些为问题，会对你的实现有帮助）

### 原始数据准备

每个脚预测各自的下一次移动路径，需要从动画中知道脚步的触地、移动信息，使用程序自动化生成脚步数据而不是手动标记，基于 2 个 Filter：脚趾的位置、速度，以及膝盖的角度、位置和脚踝的位移。

通过 curve 运动曲线来判读 foot lock / foot up （抬起）（这里应该逻辑和 footplacement 中判断 footLock 的条件一直）。预测 Foot 的运动，基于过去的位置和前进的速度以及动画时间计算出下一个脚步位置。

### Pelvis 位置的稳定

稳定 Hips，本身人物运动是就有一定的颠簸，上下坡时幅度加大，但是游戏里看着会奇怪，所以需要平滑减轻。方案是选择一个腿作为支撑腿，用于确定 Hips 高度，不是物理意义的重心腿，上坡时一般选择后腿，下坡时选择前腿。同时可以构造

臀部的插值弹簧在不同的运动情况下插值应该不同：然后使用临界阻尼模型来平滑高度，停下时松一点，跑步时紧一点，对于动画自带的上下颠簸的位移不进行平滑，只对环境造成的位移进行平滑。

![](https://pica.zhimg.com/v2-0cdf904aa31df924837c9bf8e3cf7b94_r.jpg)

### 脚步位置解算

通过原始数据的准备，我们可以知道下一次脚步的落地的位置。通过当前落脚点和下一次落脚点。可以看下面的图片有黄绿蓝三种 foot 路径，原先的 foot 路径会存在障碍物与 foot 碰撞， 蓝色的 是被动 IK 的 foot 路径会存在 foot 路径的突变会很不自然。GDC 分享的方案就是黄色的路径。接下来可以看看刺客信条中的实现。

简单的分割路径不能处理一些突起的情况，GDC 中的分享要去我们对地表进行更精确的建模，方案创建一个跟路径平行且长度一致的胶囊体，从 Hips 位置扫到地面，返回一堆交点跟法线，然后过滤一边这些数据，按从近到远排序，去除重复的点，剩余的点连接起来成新的路径（即下面的红色路径）。此外还要检测点之间的高度看是否可达。新的路径会有凹下去的边，同时还需要简化凸多边形，以使脚移动看起来更加合理，这个处理只针对脚。

![](https://picx.zhimg.com/v2-05305d066c599878554300ecb9fd6199_r.jpg)

**其他的修正：**

*   Foot 旋转的 Pitch 在上坡时保持水平，下坡时和地面平行，Idle 动画时给与一定的 Roll，限定一个较小的值，跑动时不执行任何旋转修正。  
    
*   FootLock 的情况容易出现腿部交叉重叠的情况：包括不锁朝向（在转弯的时候）、脚步启动结束前一小段时间给与一定的滑动、一些临界条件放开锁定。

其实 GDC 中并没有分享有了对于地形建模，计算出来 foot 路径之后如何处理脚步的旋转处理。但是根据文章中的思路 ” **更多保存动画的原始信息** “，来看可以建立一个 FootBase 并应用。

### 实现基于预测脚步的 IK

![](https://pic3.zhimg.com/v2-8b4be86221f97e8724a938d6c3633824_r.jpg)

### **设计思路：**

​ 设计要保证 IK 功能的可移植性与性能，我使用尽量少的蓝图逻辑。所以全部的 IK 功能使用 C++ 编写。（唯一的蓝图逻辑是 AnimationModify 曲线生成工具）。

​ 传统 IK 方式（也就是被动的 IK 系统）与预测的 IK 方式各有其优点，传统 IK 系统往往在鲁棒性，更多地形的适配，各种运动状态比如跑跳上拥有更强的表现力，预测的 IK 系统虽然复杂度要高于传统 IK ，但是效果表现更好，性能更好（只用在重建路径时使用物理检测，其余时间都是查询重建的路径数据）。而我选择将二者结合：制作一个鲁棒性较强，性能与效果更好的 IK 系统。

IK 处理流程图：

![](https://pic2.zhimg.com/v2-b1a7e5a96a1ee6ea0bb9230aae934f2d_r.jpg)

### Foot 数据生成

![](https://pic1.zhimg.com/v2-85c4626e8effa9d50de5d58e115a3056_r.jpg)

唯一的蓝图逻辑就是这里，先写了 AnimationModify 用于生成曲线。通过这个曲线上的数值来告诉我们左右脚脚步还有多久会落下。

蓝图核心逻辑是先通过 ball （脚趾）的 speed 与 Rotation，（一定要注意此时设置的参数与处理逻辑应该与 FootLock 锁步处理时候的逻辑保持一致）判断 footLock 与 footUnLock 两种状态，曲线的目的主要是可以在任意时刻重建路径。

footLock 时候曲线其实没啥意义，footUnLock 阶段代表动画播放到该时间脚还有多久落地。

![](https://picx.zhimg.com/v2-574c616815c1731adb74c7228cebe085_r.jpg)

简单一点的话就是下面的解决方案：

[Youtute 预测脚步](https://link.zhihu.com/?target=https%3A//www.youtube.com/watch%3Fv%3DuNLeVKRh8Vo)

### Gather Data

主要包括 Rumtime 数据：白颜色的数据主要是正常 IK 过程中要记录的数据，紫色的数据是预测所需要的数据在 GatherData 阶段收集，黄色的是动态插值参数。

![](https://pic1.zhimg.com/v2-68e17547e530d31387e8637ba41fc294_r.jpg)

比如从动画曲线中读取等一系列的操作，进行一些预计算，计算与地面对齐参数：

```
PelvisData.PredictPelvisTime =
        Context.CSPContext.Curve.Get(PelvisData.SwitchFrontLegCurve, bValidSpeedCurve, DefaultSpeedCurveValue);


```

### Foot 路径预测

只在 FootLock 到 FootunLock 的时刻进行预测路径，之后每一帧只检测当前时刻的预测值与之前时刻预测的误差。超过误差范围才开始进行重新预测。

预测的过程：

脚步的起点的 StartLocationWS：

​ StartLocationWS = FootTransfromWS(即为当前脚所在的位置)

脚步的终点 EndLocationWS：

​ EndLocationWS = StartLocationWS + 角色速度 * 脚步落下时间 + FootUnLockToLockCS

![](https://pic4.zhimg.com/v2-7b21dd6a9832986743d4bf8c8434a73f_r.jpg)

**胶囊体 Sweep 生成路径信息**

![](https://pica.zhimg.com/v2-fb79ce3684cdb377f387481fb38dc774_r.jpg)

方案创建一个跟路径平行且长度一致的胶囊体，从 Hips 位置扫到地面，返回一堆交点跟法线，然后过滤一边这些数据，按从近到远排序，去除重复的点，剩余的点连接起来成新的路径。

**单端单向迭代生成路径**

首先从当前落脚点向前打射线，在碰撞点前方一点点从上往下打射线，（但是要注意每一次做射线检测需要做一点偏移）得到了第一个路径点。然后从终点出发向后打射线，这样前后交叉依次执行，前面的迭代点，到后面的迭代点没有就结束。

![](https://picx.zhimg.com/v2-84679be9766c2474f6c2a0b50eb5f6b7_r.jpg)

**双端交叉式迭代生成路径**

首先从当前落脚点向前打射线，在碰撞点前方一点点从上往下打射线，（但是要注意每一次做射线检测需要做一点偏移）得到了第一个路径点。然后从终点出发向后打射线，这样前后交叉依次执行，前面的迭代点，到后面的迭代点没有就结束。

![](https://pica.zhimg.com/v2-6c593ab091d2e751220f84bc0b142e8c_r.jpg)

后面两个迭代建立路径的方法会有一个问题，如果出现上下坡的情况会出现很多次迭代，在实际中要注意设置迭代的最大次数，与每一次迭代可以做一点偏移。

![](https://pic2.zhimg.com/v2-c6ae9b53240b97865693192d3bb9eb7b_r.jpg)

将地形建模之后的点依次放入数组中，这样就有未来的地形数据。Foot 最终是在我们建模的平面上去移动的。

如何通过 FootPath （一些点的集合）获取到 FootPathPlane。会先根据上一步的落脚点到当前脚所在位置的向量，投影到上一步的位置到下一步位置的向量上获取，已经完成步伐多少的百分比，通过这个百分比就可以查询到在 FootPath 的哪一个点。（法向量可以通过叉乘获取平面的另外一个点，再做一次叉乘获取）

![](https://pica.zhimg.com/v2-f4e58ddb851454b61c7f6f5ac4075d72_r.jpg)

### FootLock

第一步是根据脚部的 Speed，Rotation，Distance(脚到地面) 判断脚步锁定的百分比，如果脚步锁定的百分比是接近 0 那么直接最终脚所在的平面就是上一步 FootPathPlane 求解的平面上。如果脚步锁定的百分比不为 0 ，做一个向下的射线检测根据求解出脚步锁定的平面 FootLockPlane，然后根据脚步锁定的百分比在二者中处理就可以求解出最终所在的平面了。

这一步主要是用于上下坡的时候与地面接触的时候脚步可以完美的与地面贴合。

![](https://pic3.zhimg.com/v2-45c05377f3e250ef9d9782b37e1f0434_r.jpg)

具体的插值比例和开始的百分比如下图：

![](https://pic1.zhimg.com/v2-d4466ac36174dcacd6a1df3064ac912c_r.jpg)

![](https://pic2.zhimg.com/v2-894d133f789026a2b94a09712c3d160b_r.jpg)

![](https://picx.zhimg.com/v2-451c49a845047bd91383134fb2881209_r.jpg)

图中的方框为脚步的 FootLockPlane 与 FootPathPlane 插值过后的平面，白色代表是左脚，蓝色代表是右脚。红色代表当前以 FootLockPlane 的结果为主。

### Pelvis 位置预测

```
FLegRuntimeData &FrontLeg = (LeftLeg.AlignedFootTransformCS.GetLocation().Y > RightLeg.AlignedFootTransformCS.GetLocation().Y ? LeftLeg : RightLeg );
FLegRuntimeData &BackLeg = (LeftLeg.AlignedFootTransformCS.GetLocation().Y <= RightLeg.AlignedFootTransformCS.GetLocation().Y ? LeftLeg : RightLeg );


```

根据在 Component 坐标系的 Y 值的大小来判断前后，直到前后脚交换时开始预测。起点为上一次预测的落点，终点为后脚的预测的落地点作为运动结束点，这样就可以生成 Pelvis 路径数据。

![](https://pic1.zhimg.com/v2-fa6d2785129aa9ee7c9ddacd9f7c4db8_r.jpg)

### 一些细节处理

何时使用传统 IK 处理：

**避免运动的突变导致，频繁的重建路径消耗比较多的性能**

*   当加速度与速度同向时候，并且加速度近似等于 CharacterMovementComponent 的最大加速度时。即原地 Idle 加速到 Jog 的过程。  
    
*   当加速度与速度反向时候，并且加速度近似等于 CharacterMovementComponent 的最大减速度时。即从 Jog 到 Idle 的过程。  
    
*   当速度比较低的时候，比如点按移动键人物抽搐移动时，转弯幅度过大时。  
    

**避免预测 IK 出现 bug，保证系统的鲁棒性**

*   玩家刚进入游戏时，并没有之前的步伐数据  
    
*   玩家从别的地方跳下来，之前的步伐数据失效（我判断的很简单就是使用距离去判断的）  
    
*   玩家跳跃的时候不需要进行 Predict IK  
    

### Predict IK 缺陷与解决思路

**缺陷一：由于需要做 FootPathPlane 需要和 FootLockPlane 去做插值，如果在边缘可能会如下所示的问题，FootLockPlane 与 FootPathPlane 差别比较大这样就会出现先登上去，在滑下来的情况。反之，下楼梯也会出现这样的情况。**

![](https://pic4.zhimg.com/v2-5db51368127f5b108b52534099e163a7_r.jpg)

![](https://pica.zhimg.com/v2-7fb9a6b90faa1cbacab85680d73f9550_r.jpg)

这个问题如果在障碍物或者台阶高度差比较大的时候会比较明显

**解决思路如下：**

1.  前面提及的曲线生成一定要与 PredictIK 中 FootLock 的处理一致，但是局限性是在如果在各种动画的混合上，会差一点。

![](https://pic1.zhimg.com/v2-45634e138543c5e5892197d93aeba594_r.jpg)

1.  FootLockType 必须加以限制，并且 FootLock 的处理很关键。必须限制 FootLock 必须使得 FootLocation 的位置不变。

注：以 Footplacement FootLockType 节点中的表现排序：

RotationAroundAnkle（脚踝位置不变，但是脚趾可以绕着旋转） == LockRotation（脚踝脚趾都不变） > RotationAroundBall （脚踝绕着脚趾动）> UnLock （不做处理）

**核心是一定要锁住脚踝位置。**

FootLockType 必须加以限制，并且 FootLock 的处理很关键。必须限制 FootLock 必须使得 FootLocation 的位置不变。

**缺陷二：Predict IK 在急转向急停，停步，起步时候的。需要频繁的重建路径，性能消耗比较大。**

![](https://pic3.zhimg.com/v2-0d5370ee83794d9c3cec3dfc33fbf6a8_r.jpg)

红线就是重建的路径，我采取的思路是检测当前运动状态在传统 IK 与 预测 IK 中做切换。可以看到当转向剧烈的时候使用的是 TraditionIK , 如上图中在强烈转向中使用的是传统 IK。

**缺陷三：由于 pelvis 路径 pelvis 位置的提前抬升会导致脚够不到地面**

![](https://pic3.zhimg.com/v2-4932ae3f542a7ae09e67e77296f95972_r.jpg)

特别要提一句： 2016 年大革命分享的时候他们说让臀部在虚拟平面上移动，即对臀部运动路径也进行建模，据我观察（从登上楼梯的第一步的 pelvis 移动来看）2023 刺客信条 - 幻景的人物的 pelvis 解算大概率不是在虚拟平面上移动，大概率是根据后脚位置来算的。

原因可能是 predict IK 预测 pelvis 移动的过程中发现了一个下问题（红）：Lock 锁住的腿浮空。由于 pelvis 路径 pelvis 位置的提前抬升会导致脚够不到地面。

![](https://pic2.zhimg.com/v2-57a1e0716975871d98dec280aaaf578b_r.jpg)

解决思路：更多还是要依靠弹簧插值，或者是直接采用最上面说过的臀部稳定方式。

参考链接：

[https://www.gdcvault.com/play/1023316/Fitting-the-World-A-Biomechanical](https://link.zhihu.com/?target=https%3A//www.gdcvault.com/play/1023316/Fitting-the-World-A-Biomechanical)[Shadow：【UE5】预测脚步 IK 实现 - PredictFootIK](https://zhuanlan.zhihu.com/p/679650155)[左加右：《刺客信条》基于预测的 FootIK 方案分析与 UE4 实现](https://zhuanlan.zhihu.com/p/552056254)[sincezxl：UE4 实现基于预测的 FootIK](https://zhuanlan.zhihu.com/p/380222928)