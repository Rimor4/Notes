# UE 笔记

## 动画

### 动画更新流程

[动画蓝图更新流程 - 南京周润发](https://zhuanlan.zhihu.com/p/676529450)

#### 两个阶段

核心逻辑清晰地分为两个阶段：**Update（逻辑更新）** 和 **Evaluate（姿态评估）**。这是 UE 动画系统的基石。

------

1. 总述：核心架构（Update vs Evaluate）

> “UE 的动画系统为了支持多线程并行优化，核心流程被严格拆分为两个阶段：**Update（更新）** 和 **Evaluate（评估）**。
>
> - **Update**：负责计算‘我们要播放什么？’（计算时间、权重、状态机跳转）。通常在 **GameThread**（或者 Worker 线程的 Update 阶段）执行。
> - **Evaluate**：负责计算‘骨骼具体在哪？’（计算 Bone Transform）。为了性能，这一步通常是在 **Worker Threads**（并行评估）中执行的。”

------

2. 第一阶段：Update (逻辑更新)

这是动画数据的准备阶段。

- **入口**：`USkeletalMeshComponent::TickComponent` 触发。
- **数据收集 (Gathering Data)**：
  - **Proxy 机制**：UE 为了线程安全，设计了 `FAnimInstanceProxy`。主线程的数据（如角色速度、状态）会被拷贝到 Proxy 中。**Lyra 的 Thread Safe Update** 就是为了让这一步也能并行化，避免在 GameThread 访问非线程安全的数据。
- **计算 (Calculation)**：
  - 遍历动画树（AnimGraph），调用每个节点的 `Update_AnyThread`。
  - **状态机**：判断是否切换状态（State Machine Transition）。
  - **时间推进**：更新动画的 `CurrentTime`，处理循环、Notify 触发。
  - **权重计算**：计算各个 Blend 节点的 Alpha 值（比如混合 50% 的走和 50% 的跑）。
- **关键点**：此时**不涉及**具体的骨骼坐标计算，只处理逻辑变量。

------

3. 第二阶段：Evaluate (姿态评估)

这是最消耗 CPU 的数学计算阶段，也是优化的核心。

- **入口**：通常在 `ParallelAnimationEvaluation` 任务中触发。
- **递归遍历**：从 Root Node 开始，调用 `Evaluate_AnyThread`，一直递归到叶子节点（如 `AnimSequencePlayer`）。
- **叶子节点 (Leaf Nodes)**：
  - 解压动画压缩数据（Decompression），根据当前的 Time 获取原始的骨骼姿态（Local Pose）。
- **中间节点 (Blend Nodes)**：
  - 执行混合运算（Lerp）。比如 Layered Blend Per Bone，会将两个来源的 Pose 按骨骼遮罩进行混合。
- **输出结果**：
  - 最终生成一个 `FCompactPose`。这是一个**针对缓存优化（Cache Friendly）**的紧凑数组结构，存储了所有骨骼的 Transform。



#### 重要函数更新顺序

第一阶段：初始化（只执行一次）

当一个角色被 Spawn 或者 `SkeletalMeshComponent` 初始化时：

1. **`NativeInitializeAnimation` (C++)**
   - 最早执行，用于缓存组件引用、初始化变量。
2. **`BlueprintInitializeAnimation` (Blueprints)**
   - 蓝图中的 `Event Blueprint Initialize Animation`。
3. **AnimNode `Initialize_AnyThread`**
   - 动画蓝图（AnimGraph）中每个节点的初始化。注意后缀 `_AnyThread`，意味着它可能在后台线程重置（比如状态机重置时）。

------

第二阶段：每一帧的更新循环（核心流程）

这个流程非常关键，分为 **Game Thread（主线程准备数据）** 和 **Worker Thread（后台线程计算数据）**。

1. 主线程准备 (Game Thread)

由 `USkeletalMeshComponent::TickComponent` 触发。

- **`NativeUpdateAnimation` (C++)**
  - **老式/非线程安全**。这是最传统的入口，用于每一帧在主线程获取 Pawn、Controller 等数据。
  - *缺点*：会阻塞主线程，Lyra 极力避免在这里做繁重逻辑。
- **`BlueprintUpdateAnimation` (Blueprints)**
  - 对应蓝图中的 `Event Blueprint Update Animation`。
  - **警告**：这也是运行在主线程的！很多新手喜欢在这里写一堆逻辑，导致游戏掉帧。

2. 并行更新开始 (Transition to Worker Thread)

此时，UE 会尝试将任务分发给后台线程。这里引入了 **Proxy (代理)** 概念。

- **`FAnimInstanceProxy::PreUpdate`**
  - 数据拷贝阶段。将主线程（AnimInstance）的数据拷贝到线程安全的代理（Proxy）结构体中。

3. 线程安全更新 (Worker Thread - 逻辑层)

这是 **Lyra** 和现代 UE5 推荐的逻辑处理位置。

- **`NativeThreadSafeUpdateAnimation` (C++)**
  - 你自定义的 C++ `UAnimInstance` 如果覆写了这个函数，它会在后台线程运行。在这里访问 `UObject` 是不安全的（除非使用 `TWeakObjectPtr` 或 Property Access）。
- **`BlueprintThreadSafeUpdateAnimation` (Blueprints)**
  - 对应蓝图中的 **`Event Blueprint Thread Safe Update Animation`**。
  - 这是 Lyra 放置状态机逻辑、Locomotion 计算的核心位置。它极快，因为它不通过蓝图虚拟机与主线程交互，只操作局部数据。

4. 动画图表更新 (Worker Thread - AnimGraph Update)

逻辑计算完毕后，引擎开始**从根节点（Root）向下**遍历动画图表，计算“权值”和“时间”。

- **AnimNode `Update_AnyThread`**
  - **顺序**：从 Root Node -> Blend Nodes -> Leaf Nodes (Sequence Player)。
  - **职责**：
    - 推进动画时间（Time Advancement）。
    - 解析 Notify（加入队列，稍后在主线程触发）。
    - 计算混合权重（Alpha）。
    - **注意**：这里**不产生**骨骼姿态，只计算“要播放多少秒”、“混合比例是多少”。

5. 动画图表评估 (Worker Thread - AnimGraph Evaluate)

这是最耗时的步骤。引擎**从根节点（Root）向叶子节点（Leaf）拉取（Pull）\**数据，然后\**从叶子向根**返回姿态。

- **AnimNode `Evaluate_AnyThread`**
  - **顺序**：Root 请求数据 -> 递归调用子节点 Evaluate -> 子节点解压动画数据 -> 返回 `FPoseContext` -> 父节点混合（Lerp）。
  - **职责**：
    - **Leaf Node**：解压骨骼 Transform。
    - **Blend Node**：执行数学混合（如 `Blend Per Bone`）。
    - **SkeletalControl / Control Rig**：执行 IK、LookAt 等骨骼修正。
  - **输出**：最终生成 `FCompactPose`（紧凑的骨骼数组）。

6. 后处理与应用 (Post-Process)

回到主线程或渲染线程同步。

- **`NativePostEvaluateAnimation` (C++)**
  - 评估完成后的回调，通常用于调试或极特殊的后处理。
- **Swap Buffers & Render**
  - 将计算好的 Transform 提交给渲染线程，更新 Skinned Mesh。
- **Trigger Notifies (Game Thread)**
  - 注意！**AnimNotify 是在所有计算完成后，在主线程触发的**。所以 Notify 里的逻辑如果修改了变量，是**下一帧**才会生效。

------

总结图谱（面试速记版）

如果面试官让你画出顺序，按这个写绝对满分：

1. **[GT] NativeUpdateAnimation** (传统入口，Lyra 几乎不用)
2. **[GT] BlueprintUpdateAnimation** (传统蓝图入口)
3. **[COPY] Proxy Data Copy** (主线程 -> 代理)
4. **[WT] NativeThreadSafeUpdateAnimation** (现代 C++ 逻辑核心)
5. **[WT] BlueprintThreadSafeUpdateAnimation** (现代蓝图逻辑核心 - Lyra Locomotion 在此)
6. **[WT] Graph Update** (计算时间/权重)
   - `Node::Update_AnyThread`
7. **[WT] Graph Evaluate** (计算骨骼位置/IK)
   - `Node::Evaluate_AnyThread`
8. **[GT] Fire Notifies** (触发音效/特效)



#### 性能优化

1. **URO**：

   ![](D:\Projects\NOTES\images\v2-9a14fd9bf3eb8fb67f670aab7123ccb5_r.jpg)

2. **ShifitBucket**：负载均衡：让不同的角色更新错峰进行

3. EvaluateRate可以比UpdateRate低一点

4. **LOD**：也可以省略远处的IK解算、动画通知等计算，还可以移除部分骨骼（LOD预先设定好的RequiredBones）

5. **LeaderPoseComponent**

   > 有一个东西叫做LeaderPoseComponent，如果它存在的话并且bFollowerShouldTickPose为false的话，自己就不去TickPose了，而是从LeaderPoseComponent输出的Pose中取。
   >
   > 当然，这玩意不止应用于SKM的拆分，还会应用于群体动画。假如我有一堆角色，它们的骨骼结构是相同的，并且要执行相同的动画，那么就完全不需要每一个都去更新和评估，而是只需要一个角色的SKM作为LeaderPoseComponent，别的作为它的从属，这样LeaderPoseComponent输出的Pose就能被别的角色共用了。

6. `EVisibilityBasedAnimTickOption` 决定各种情况下要不要进行 TickPose 和 RefreshBoneTransforms。

7. 缓存：动画节点通过**CacheBones_AnyThread**缓存动画所引用的骨骼索引等

**LeaderPose**、**CopyPose** 和 **MeshMerge**对比

1. Leader Pose Component

这是最常用的模块化角色实现方式。

- **原理**：指定一个“领头”组件（Leader），其他“跟随”组件（Follower）直接复用 Leader 的**骨骼变换缓冲（Bone Transform Buffer）**。
- **优点**：
  - **CPU 开销极低**：跟随组件不运行任何动画逻辑，不更新 AnimBP，节省了大量的 Game Thread 计算。
  - **同步完美**：因为共用同一套骨骼数据，绝对不会出现穿模或同步延迟。
- **缺点**：
  - **灵活性差**：跟随组件不能有自己的动画逻辑，无法运行独立的物理模拟（如布料或物理须子）。
  - **渲染开销未减**：虽然节省了 CPU 动画计算，但 Draw Call（绘制调用）数量没有减少，每个部件仍是独立的渲染批次。
  - **骨骼限制**：跟随组件的骨骼结构必须是 Leader 骨骼的子集。

2. Copy Pose From Mesh

这是一种基于动画蓝图（AnimBP）节点的方案。

- **原理**：在跟随组件的 AnimBP 中使用 `Copy Pose From Mesh` 节点，每帧从目标 Mesh 拷贝当前的骨骼姿态。
- **优点**：
  - **高度灵活**：拷贝完姿态后，你可以继续在该组件的 AnimBP 中添加额外的动画逻辑（如 Control Rig、LookAt、动力学效果等）。
  - **支持不同骨骼**：只要骨骼名称匹配即可拷贝，不要求完全一致的骨架结构。
- **缺点**：
  - **CPU 开销最大**：每个跟随组件都需要运行自己的 AnimBP。
  - **可能存在延迟**：如果组件的 Tick 顺序不当，可能会看到跟随组件比主体慢一帧。

3. Mesh Merge (Skeletal Mesh Merge)

这是一种在底层将多个 Skeletal Mesh 合并成一个 Mesh 的技术。

- **原理**：在运行时（通常是角色生成时）将多个网格体及其材质合并成一个单一的网格体资产。
- **优点**：
  - **渲染性能最强**：极大地减少了 **Draw Call**。合并后，原本 10 个部位可能只需要 1 个 Draw Call（取决于材质数量）。
  - **动画开销低**：只需要为一个组件计算动画。
- **缺点**：
  - **合并开销**：在执行合并的瞬间会有明显的 CPU 掉帧（Hitch），通常在加载界面或后台处理。
  - **材质局限**：如果各部件材质不同，合并后可能需要处理复杂的 Texture Atlas（纹理图集）。
  - **维护复杂**：不支持 Morph Targets（混合形状）的动态合并（除非使用特殊插件如 Mutable）。



#### 多线程

主线程更新：

<img src="D:\Projects\NOTES\images\v2-a1df1a24be903f7a4a3972fd67c7b603_1440w.jpg" alt="img" style="zoom:50%;" />

多线程更新：

![img](D:\Projects\NOTES\images\v2-a9daf51b9c51669807a691be79ea4a51_r.jpg)



[[UnrealCircle深圳\]《黑神话：悟空》的Motion Matching | 游戏科学 招文勇_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1GK4y1S7Zw/)

<img src="D:\Projects\NOTES\images\image-20251116115328799.png" alt="image-20251116115328799" style="zoom: 25%;" />



## AI

[](Ready\面试准备.md#Behavior Tree vs. State Tree)

### 行为树

#### 节点的**单例问题**

行为树节点：默认一棵行为树资源对应一个单例（不管是不是不同的ai跑）

解决办法：

1. ```cpp
   UBTMyTask::UBTMyTask()
   {
       bCreateNodeInstance = true;
   }
   ```

2. 把成员数据独立出来，交给UE托管:

   首先你要为自己的成员变量定义一个结构体。

   然后要在你的节点里面重写`GetInstanceMemorySize`，返回结构体的大小（这一步的作用是告诉UE为你的成员数据开辟对应的内存空间）。

   ```cpp
   USTRUCT()
   struct FMyTaskMemory
   {
       GENERATED_BODY()
   
       UPROPERTY()
       AMyCharacter* ControlledCharactor = nullptr;
   }
   
   UCLASS()
   class UBTMyTask
   {
       GENERATAED_BODY()
   
       virtual uint16 GetInstanceMemorySize() const override
       {
           return sizeof(FMyTaskMemory);
       }
   
       virtual void InitializeMemory(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory, EBTMemoryInit::Type InitType) const override
       {
           // 获取数据内存块
           FMyTaskMemory* MemberMemory = CastInstanceNodeMemory<FMyTaskMemory>(NodeMemory);
           MemberMemory->ControlledCharactor = StaticCast<AMyCharacter>(OwnerComp->GetOwner()->GetControlledPawn());
       }
   
       virtual void TickTask(UBehaviorTreeComponent& OwnerComp, uint8* NodeMemory, float DeltaSeconds) override
       {
           FMyTaskMemory* MemberMemory = CastInstanceNodeMemory<FMyTaskMemory>(NodeMemory);
           UE_LOG(LogTemp, Log, TEXT("角色的id是：%d"), MemberMemory->ControlledCharactor->CharacterID)
       }
   }
   ```

#### 打断（Decorator）

【打断行为可以提高性能，而不是每帧都从root开始】

##### 注册阶段 (Registering)

当行为树运行并执行到带有黑板装饰器的节点时，`UBTDecorator_Blackboard` 会调用基类中的逻辑：

1. **获取黑板组件**：通过 `BehaviorTreeComponent` 找到关联的 `UBlackboardComponent`。
2. **绑定委托（Delegate）**：它会向黑板组件注册一个回调。在源码层级，这通常涉及到 `UBlackboardComponent::RegisterObserver`。它将装饰器自身的一个函数（即 `OnBlackboardKeyValueChange`）与特定的 **黑板键（Blackboard Key）** 绑定。

##### 触发阶段 (Firing)

1. **黑板值变更**：当你在代码或行为树任务中调用 `SetValueAs...`（如 `SetValueAsObject`）时，黑板组件内部会检测新旧值是否相等。
2. **分发通知**：如果值确实发生了改变，黑板组件会遍历所有注册在该 Key 上的观察者（Observers）。



##### 打断流程 (ConditionalFlowAbort) 的详细逻辑

一旦监听到值改变，装饰器必须决定是否要“掐断”当前正在执行的任务。这就是 `ConditionalFlowAbort` 的职责。

它会根据你在编辑器中设置的 **Observer Aborts** 选项（None, Self, Lower Priority, Both）来执行不同的逻辑：

逻辑判断：是否满足打断条件？

在调用打断前，它会先执行一次 `CalculateRawConditionValue`。

- 比如你设置“当 `Enemy` 不为空时打断”。如果黑板值变动是从 `EnemyA` 变成 `EnemyB`（依然不为空），虽然触发了监听，但逻辑判定没变，则不会打断。

执行打断：如何停止当前任务？

如果判定需要打断，`ConditionalFlowAbort` 会触发核心的 **Flow Abort** 流程：

1. **确定打断范围**：
   - **Self**：打断当前装饰器下的子树。
   - **Lower Priority**：打断当前节点右侧（优先级更低）的所有正在运行的任务，强制让行为树重新评估并回到这个高优先级的装饰器节点。
2. **调用 `RequestExecution`**：
   - 装饰器通过 `UBehaviorTreeComponent` 发起一个“重评估请求”。
3. **清理与转换**：
   - 行为树会立即调用当前运行任务的 `AbortTask`，触发清理逻辑（如停止移动）。
   - 下一帧（或立即），行为树**从打断点重新选择路径**，切换到符合条件的高优先级分支。
4. **只有两种情况会重新寻路**：
   1. 当前任务执行完毕（返回 `Succeeded` 或 `Failed`）。
   2. 发生了**打断（Abort）**。



### 状态树

[StateTree 架构深度源码解析：核心机制、内存布局与执行流水线](https://zhuanlan.zhihu.com/p/1975009910996615830)

<img src="D:\Projects\NOTES\images\StateTree运行逻辑.png" alt="StateTree运行逻辑"  />

#### 一、 初始化流程（Initialization）

当 StateTree 开始运行（Tree Start）时，系统会进行数据准备和初始状态选择。

1. **Evaluator 数据准备**：
   - 首先调用所有 Evaluator 的 `TreeStart`，随后执行一次 `Tick`。
   - **目的**：在状态选择前，预先从环境中提取并处理好必要的数据（如感知到的敌人信息、玩家距离等）。
2. **初始状态选择（SelectState）**：
   - 目标状态默认为 **Root**。
   - **条件判定**：自根向叶检查 `Enter Conditions`，判断哪些子状态可以被激活。
   - **Task 入场**：一旦确定了激活路径，系统会**自根到叶（正序）**依次调用路径上每个 State 中 Task 的 `EnterState`。

------

#### 二、 运行时每帧更新（Tick）

StateTree 的 Tick 分为两个主要部分：数据更新与 Task 执行。

1. **Evaluator 更新**：

   - 每帧最先调用 `Evaluator's Tick`，更新全局或上下文数据。

2. **Task 逻辑执行**：

   - **执行顺序**：**自根到叶（正序）**调用每个激活状态里 Task 的 `Tick`。
   - **阻断机制**：如果某个 Task 在 Tick 中返回了“成功”或“失败”，系统会**停止后续（更深层级）Task 的 Tick**。

3. **任务完成处理（StateComplete）**：

   - 一旦 Task 完成（不再返回 Running），系统会**自叶到根（逆序）**调用所有已激活状态中 Task 的 `StateComplete`。

4. 过渡流程（Transition）

   1. **条件检查**：

      - **执行顺序**：**自叶到根（逆序）**检查 Transition 上的 Conditions 是否满足。
      - **逻辑逻辑**：子状态（叶子）通常具有更高的逻辑优先级，若子状态满足跳转条件，则优先触发。

   2. **退出旧状态**：

      - 如果条件满足，系统会**自叶到根（逆序）**依次调用当前路径上所有 Task 的 `ExitState`，清理旧状态逻辑。

   3. **进入新状态**：

      - 调用 `SelectState` 跳转至目标状态（Target State），重新开始“自根到叶”的 Enter 流程。

      - **SelectState**流程
        - **识别目标：** 系统首先解析目标状态句柄（State Handle）。
        - **寻找共同祖先：** 这是一个关键的算法步骤。执行上下文在树的层级结构中向上回溯，直到找到当前状态（Current State）和目标状态（Target State）的**最近公共祖先**（或者根节点）。
        - **退出阶段（Exit Phase）：** 系统自底向上（From Leaf to Root）调用所有状态的 `ExitState()`，直到（但不包括）共同祖先。
        - **进入阶段（Enter Phase）：** 系统自顶向下（From Root to Leaf）调用从共同祖先到新目标叶子节点路径上所有状态的 `EnterState()` 。

#### 数据绑定与属性传播机制

在 UE5 的 **StateTree** 体系中，数据绑定（Data Binding）与传播机制是其区别于行为树（Behavior Tree）最核心的特性之一。它通过显式的**属性链接（Property Link）**代替了行为树中基于字符串索引的黑板（Blackboard）模式。

以下是根据你提供的运行逻辑图及 UE5 核心机制进行的详解：

------

1. 数据绑定的核心概念：Schema 与 Context

在 StateTree 中，数据传播的起点是 **Schema（架构）**。

- **Schema 定义上下文**：每个 StateTree 必须选择一个 Schema（如 `StateTreeGameplayTasksSchema`）。Schema 规定了该树能访问哪些**外部数据**（Context Data），例如 `AActor`、`UAIPerceptionComponent` 或自定义的 `UObject`。
- **Context 注入**：当 StateTree 开始运行（初始化阶段），外部系统会将这些具体的对象实例注入到 StateTree 的**实例数据（Instance Data）**中。

------

2. 数据传播路径：Evaluator -> Task

第一步：Evaluator 的数据提取（数据源）

- **初始化阶段**：调用 `Evaluator` 的 `TreeStart` 和 `Tick`。
- **传播逻辑**：Evaluator 作为“传感器”，从 Context（如感知组件）中提取原始数据，并将其转化为 StateTree 内部易于处理的属性（如 `bIsEnemyVisible` 或 `TargetActor`）。
- **数据绑定**：Evaluator 定义的这些输出属性，可以被后续的 Task 或 Transition 直接绑定。

第二步：属性绑定（Property Binding）

- **显式链接**：在 StateTree 编辑器中，你可以直接将 Task 的输入参数“连线”到 Evaluator 的输出参数上。
- **底层机制**：这在底层是通过 **`FStateTreePropertyBinding`** 实现的。它不依赖黑板键的字符串查询，而是直接记录属性在内存中的偏移量（Offset），因此访问速度极快。

------



#### Behavior Tree vs. State Tree

| **特性**     | **Behavior Tree (行为树)**                                   | **State Tree (状态树)**                                      |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **设计范式** | **基于决策 (Decision-based)**。每帧（或事件触发）从根节点重新评估，寻找可执行的叶节点。 | **基于状态 (State-based)**。它是分层状态机 (HFSM) 的进化版，明确“当前正处于哪个状态”。 |
| **运行机制** | 依靠 **Selector/Sequence** 控制流，通过失败/成功反馈来回溯。 | 依靠 **Transition (过渡)** 驱动。状态切换是显式的，只有满足条件才进行状态跳转。 |
| **数据通信** | 强绑定 **Blackboard (黑板)**。数据读写往往需要显式的 Key 绑定。 | 采用 **Property Bag / Data Bindings**。支持更直接的属性引用，减少了黑板带来的维护成本。 |

2. 核心区别 (三大维度)

A. 决策机制：逻辑搜索 vs. 状态迁移

- **行为树 (BT)**：依靠**打断 (Interrupt)**。
  - BT 每一帧都在扫描优先级。如果低优先级的“巡逻”正在运行，但高优先级的“看到敌人”条件满足了，BT 必须通过**装饰器 (Decorator)** 强制中断当前任务。
  - *缺点*：当条件极多时，树上到处都是重复的装饰器，逻辑容易变成“意大利面条”。
- **状态树 (ST)**：依靠**过渡 (Transition)**。
  - ST 显式地定义了从 A 状态到 B 状态的连线。它只在当前状态完成或特定触发器触发时，才去评估“我该去哪”。
  - *优点*：逻辑流非常直观，像流程图一样，状态切换是受控且唯一的。

B. 性能模型：内存与评估效率

- **行为树 (BT)**：
  - 每个节点都是一个对象，且带有运行时状态。在大规模 AI（如几百个僵尸）时，内存开销和每帧扫树的 CPU 开销不容忽视。
- **状态树 (ST)**：
  - **极度紧凑**。ST 将所有数据存储在一个连续的内存块（Instance Data）中。它的评估过程类似于查表，且支持**并行化更新**。
  - *优势*：在同等硬件下，ST 能支撑的 AI 数量远超 BT。

C. **数据共享**：Blackboard vs. Data Bindings

- **行为树 (BT)**：通常绑定一个 **Blackboard (黑板)**。所有数据读写都要通过 String/Name 去查找黑板键，存在一定的开销且类型检查较弱。
- **状态树 (ST)**：使用了 **Property Bindings (属性绑定)**。它可以直接在编辑器里将一个 Task 的输出连到另一个 Task 的输入，类似于蓝图参数传递。这在开发时更安全，运行时也更高效。





## GAS

[unreal5 GAS框架 源码深入解析（持续更新中） - 知乎](https://zhuanlan.zhihu.com/p/1919752469812061854)

1. GAS 整体实现与各模块功能

**面试官问：请简述 GAS 的整体架构，各核心模块是如何分工的？**

- **Ability System Component (ASC)：** 核心枢纽。它挂载在 Actor 上，负责管理所有的 Gameplay Abilities、Attributes、Gameplay Effects 以及 Gameplay Tags。它是所有 GAS 交互的入口。
- **Gameplay Ability (GA)：** 定义“做什么”。包含技能的触发逻辑、资源消耗（Cost）、冷却（Cooldown）以及具体的执行流程。它通常是异步的，通过 Ability Task 处理持续性逻辑。
- **Gameplay Effect (GE)：** 定义“改变什么”。是修改属性的唯一合法手段。它不仅可以做加减乘除，还能处理时长（Duration）、周期性触发（Period）以及复杂的叠加（Stacking）逻辑。
- **Attribute Set：** 存储数值。定义生命、魔法、力量等属性。它负责属性的计算逻辑（通过 `PreAttributeChange` 等钩子）和网络同步。
- **Gameplay Tags：** 系统的“粘合剂”。通过层级化的标签（如 `State.Frozen`）实现逻辑解耦。GA 的触发、GE 的过滤、甚至 UI 的显示都依赖于 Tag 的状态判断。
- **Gameplay Cue (GC)：** 表现层解耦。负责播放特效、音效，与具体的逻辑计算完全分离，优化了网络带宽（通过 RPC 触发一次性或循环表现）。



**激活一个技能流程**

PlayerState初始化/动态根据GameFeature添加 `GiveAbility`，内部注册这个技能AbilityTrigger中的Tag；

 主动触发GA: InputTag绑定的Press等输入事件->`TryActivateAbility` （check一系列条件，tag符合、cd、cost等，不行的话服务器在这里通过RPC告知客户端回滚）

被动触发：见下GE

**GE流程**

GE可以通过GrantedAbility赋予角色GA，再通过添加/移除Tag来自动触发GA

GE: UGameplayEffectComponent通过注册 `AllowGameplayEffectApplication` 函数来通过GameplayEffectQuery来实现自定义的复杂过滤逻辑

------

2. 网络同步、预测与回滚（重点：PredictionKey）

**面试官问：GAS 是如何处理客户端预测（Prediction）的？PredictionKey 的作用和生命周期是什么？**

**核心原理：**

GAS 使用一种“同步令牌”机制。当客户端尝试启动一个技能时，它会生成一个 **FPredictionKey**。

- **PredictionKey 的详细介绍：**
  - **定义：** 它是一个轻量级的结构体（包含序列号 ID），本质上是客户端向服务器发起请求时携带的“凭证”。
  - **生命周期：**
    1. **生成 (Client)：** 客户端按下按键，ASC 生成一个新的 `ScopedPredictionKey`。
    2. **发送 (RPC)：** 客户端将此 Key 随 `ServerTryActivateAbility` 发送给服务器。
    3. **本地执行：** 客户端在收到服务器回包前，基于此 Key **直接执行** GA 逻辑（如播放本地蒙太奇、扣除瞬时属性）。
    4. **服务器处理：** 服务器收到 RPC 后，使用相同的 Key 进行逻辑验证。如果验证通过，服务器执行逻辑并产生同步数据。
    5. **确认与销毁 (Replication)：** 服务器将处理结果（带有相同的 Key）同步回客户端。客户端发现服务器已处理该 Key，则将本地的预测标记为“已确认（Confirmed）”。
  - **功能：**
    - **防止逻辑重复：** 确保同一操作不会在两端重复触发两次。
    - **原子性：** 保证 GA 的激活、GE 的应用、Tag 的变更在同一个预测域内。

**预测与回滚的实现：**

- **属性同步：** GAS 并不通过回滚物理状态（如位置）的方式来处理属性。它依靠服务器的**权威同步**。
- **回滚机制：** 如果服务器拒绝了客户端的 GA 激活（例如：服务器判定蓝量不足），客户端会根据服务器发回的状态直接**覆盖**本地值。对于 GA，客户端会强制调用 `EndAbility` 并撤销预测产生的 Effect。

------

3. GAS 设计中“比较好”的点及对应实现

**面试官问：你认为 GAS 框架设计最精妙的地方在哪里？**

- **1. 逻辑与表现的高度解耦 (Gameplay Cue)：**
  - **实现：** 逻辑只管改属性加 Tag，表现层（特效/音效）只订阅 Tag 变化。
  - **优点：** 极大节省了带宽。不需要同步每个火球的每一个粒子，只需同步一个“火球碰撞”的 Tag，各端本地触发 GC。
- **2. 声明式的数据驱动 (Gameplay Effect)：**
  - **实现：** 通过简单的配置（Modifier、Execution）实现复杂的战斗公式，而不需要写冗余的代码。
  - **优点：** 方便策划调整平衡性，且 GE 的计算逻辑在底层是高度优化且经过严格同步验证的。
- **3. 异步任务处理 (Ability Task)：**
  - **实现：** 通过 `UAbilityTask` 实现“等待输入”、“等待延迟”、“等待碰撞”等异步逻辑。
  - **优点：** 避免了传统状态机中由于网络延迟导致的 `Tick` 轮询，使技能逻辑代码呈现流式（Streamline）结构，易于维护。
- **4. 灵活的预测支持 (Scoped Prediction Key)：**
  - **实现：** 允许开发者指定哪些逻辑可以被预测，哪些必须等待服务器。
  - **优点：** 在“强打击感（低延迟反馈）”和“数据安全性（防作弊）”之间达到了极佳的平衡。

------

面试加分点：弱网下的表现

**面试官问：弱网下 GAS 会有什么问题？**

- **回答：** 主要是**预测失效**后的表现。例如在极高延迟下，客户端预测释放了技能，但 500ms 后服务器返回“失败”，玩家会看到技能特效消失、属性值突跳。
- **优化建议：** 1. 针对核心技能做平滑插值（如果涉及数值）；2. 对于不可预测的行为（如随机抽奖），明确设置 `NonPredictive`。







## UI

### UI框架（MVVM的实践）

**MVC (Model-View-Controller)**：

- **Model**：数据层，负责存储和逻辑运算。
- **View**：表现层，负责显示（如 UGUI/UMG 里的 Prefab）。
- **Controller**：控制层，作为中转站。它监听 View 的输入，修改 Model；同时也监听 Model 的变化，去更新 View。
- **痛点**：Controller 容易变得非常臃肿（厚 C 问题），且 Controller 需要显式地持有 View 的引用，耦合度依然存在。

**MVVM (Model-View-ViewModel)**：

- **Model** 和 **View** 职责不变。

- **ViewModel**：不持有 View 的引用，它只负责暴露**可观察的数据（Observable Properties）\**和\**命令（Commands）**。

- **关键点**：引入了**数据绑定（Data Binding）**机制。View 和 ViewModel 之间是自动同步的，不需要手动在代码里写 `label.text = data.value`。

  （相当于框架帮你写了一些胶水代码，你只需要写接口

  个人理解：MVC像是面向”数据接口“（直接访问View的组件引用等数据）；而MVVM像是面向“行为接口”（VM中各种行为））

**A. 耦合度**

- **MVC**：在 Lua 侧，你的 Controller 通常需要写 `self.view.button:SetClick(...)`。这意味着 Controller 必须知道 View 的具体结构。
- **MVVM**：ViewModel 完全不知道 View 的存在。View 上的 UI 组件通过“绑定脚本”自动关联到 ViewModel 的某个字段。这意味着你换掉整个 UI 布局（View），只要变量名对得上，ViewModel 代码一行都不用改。

**B. 更新机制（如何“驱动”表现）**

- **MVC**：通常采用 **事件驱动**。Model 变了抛出一个 Event，Controller 收到 Event 后，手动调用 View 的更新接口。
- **MVVM**：采用 **数据驱动**。通过属性监听（如 C# 的 `INotifyPropertyChanged` 或 Lua 的 `setmetatable` 拦截）。当 ViewModel 的数值变了，UI 自动刷新。

后者可能**性能开销**更大，**调试困难**



### 点击流程

<img src="D:\Projects\NOTES\images\IMG_20251115-153540054.png" alt="picture 0" style="zoom: 67%;" />  

- PreviewMouseButtonDown 阶段
  从路径第 0 个元素开始往后遍历，依次调用 Root → Panel A → Button B 的 OnPreviewMouseButtonDown。

- MouseButtonDown / TouchStart 阶段
  从路径最后一个元素往前遍历，依次调用 Button B → Panel A → Root 的 OnMouseButtonDown/OnTouchStarted。

### Slate

语法实例

```cpp
void SScrollBox::ConstructVerticalLayout()
{
	TSharedPtr<SHorizontalBox> PanelAndScrollbar;
	this->ChildSlot
	[
		SAssignNew(PanelAndScrollbar, SHorizontalBox)

		+ SHorizontalBox::Slot()
		.FillWidth(1)
		[
			SNew(SOverlay)

			+ SOverlay::Slot()
			.Padding(Style->VerticalScrolledContentPadding)
			[
				// Scroll panel that presents the scrolled content
				ScrollPanel.ToSharedRef()
			]

			+ SOverlay::Slot()
			.HAlign(HAlign_Fill)
			.VAlign(VAlign_Top)
			[
				// Shadow: Hint to scroll up
				SNew(SImage)
				.Visibility(EVisibility::HitTestInvisible)
				.ColorAndOpacity(this, &SScrollBox::GetStartShadowOpacity)
				.Image(&Style->TopShadowBrush)
			]

			+ SOverlay::Slot()
			.HAlign(HAlign_Fill)
			.VAlign(VAlign_Bottom)
			[
				// Shadow: a hint to scroll down
				SNew(SImage)
				.Visibility(EVisibility::HitTestInvisible)
				.ColorAndOpacity(this, &SScrollBox::GetEndShadowOpacity)
				.Image(&Style->BottomShadowBrush)
			]
		]
	];
   ......
}
```

控件树结构

```tree
SScrollBox (this)          ← 外层控件，ChildSlot 只能挂 1 个孩子
└── SHorizontalBox (PanelAndScrollbar)      ← 第 1 层：一行两列（这里只放了 1 列）
    └── SOverlay                             ← 第 2 层：叠放 3 张“透明胶片”
        ├── Slot 0: 实际可滚动的内容区
        │      ScrollPanel.ToSharedRef()     ← 用户塞进来的真正内容
        │
        ├── Slot 1: 顶部阴影提示
        │      SImage（TopShadowBrush）
        │
        └── Slot 2: 底部阴影提示
               SImage（BottomShadowBrush）
```





## Lyra

### Lyra Gameplay 框架总览

[](Ready\面试准备#Lyra#Gameplay框架)

1. _EnhancedInput_ 框架：InputStack、InputMappingContext（优先级） TODO: InputAction 回调触发机制

2. 引擎运行流程 [UE5 -- 引擎运行流程（从 main 到 BeginPlay）](https://zhuanlan.zhihu.com/p/577433224)

3. Subsystems [《InsideUE4》GamePlay 架构（十一）Subsystems](https://zhuanlan.zhihu.com/p/158717151)

4. Lyra Gameplay 框架总览 [UE5 新项目 Gameplay 框架设计（以 Lyra 为例）](https://zhuanlan.zhihu.com/p/614718286)

   

### GameFeatures 架构

[https://zhuanlan.zhihu.com/p/467236675]

1. Lyra 中的 GameFeature:
   GameMode -启动-> Experience -封装-> GameFeatureAction..(以及 PawnData)
   
   

### ModularGameplay 架构

给原有的**PlayerController**等大类外面套一层子类**ModularPlayerController**，在初始化时用 `UGameFrameworkComponentManager::AddGameFrameworkComponentReceiver(this);` 侵入式地注册为GFCM中的Recevier；

之后继承**UGameFrameworkComponent**的各种Component (如 **ULyraControllerComponent_CharacterParts** -> **UControllerComponent**(表示绑在ModularPlayerController中的comp) -> UGFC) 在Controller中调用 **ReceivePlayer**（初始化）和**PlayerTick**（更新）等生命周期函数。

相比原先的Actor - Component组合方案，主要现在可以支持动态添加（通过Experience->**GameFeature**)



### Lyra 动画模块

1. 模块化动画: 同种动画状态，但状态内容不同时，如何解耦不同的状态内容（使用**动画层**解耦）<img src="D:\Projects\NOTES\images\v2-7a66f1bf107eb0ec877d5644a04c02d0_r.jpg" alt="img" style="zoom: 50%;" />

   LinkAnimClassLayers函数:
   	利用反射机制将原始蓝图上的LinkNode指向目标蓝图上相同的LinkNode，然后在更新时进行跳转即可。

   

2. Locomotion 如何避免 “滑步”

   【本质是用移动组件的位移去驱动动画进度】

   - a. **启动动画**怎么避免滑步

     **Advance Time By Distance Matching（函数）** 每帧让“启动动画中反映每帧位移量的 Curve(DistanceCurveName)   的进度” **匹配上** 传入的 “实际**每帧位移**量”(DistanceTraveled) ，然后进行播放速率缩放，使得动画匹配移动速度。

   - b. **循环动画**怎么避免滑步

     **SetPlayRateToMatchSpeed（函数）** 每帧用 传入的“角色实际速度 (cm/s)” (SpeedToMatch) 除以 “循环动画 RootMotion 的移动速度(总RootMotion距离除以动画长度)” 得到期望的播放速率的缩放率并设置到SequencePlayer

   - c. **停止动画**怎么避免滑步

     **DistanceMatchToTarget（函数）** 每帧让 “停止动画中指定的 Curve 的进度”匹配上 传入的 “距离到停止点距离” (DistanceToTarget) (由PredictGroundMovementStopLocation算出) ，输出对应帧，使得动画匹配移动速度。

   - d. **动画的步幅**如何更好的匹配实际移动速度

     **Stride Warping(步幅扭曲函数)** 每帧根据 “实际速度 (cm/s)” 比上 “动画 RootMotion 的速度”对双足水平距离进行缩放, 并通过IK调整盆骨位置，最终使得双足的步幅能和移动速度匹配

   

3. 如何原地转身（TurnInPlace）

   为了控制整个骨骼的旋转不受角色旋转影响，在 Lyra 中，解决的方式是通过 “RotateRootBone” 节点来旋转 Root 骨骼，通过输入 Yaw 轴的补偿值来 “抵消”Character 本身旋转产生的 Yaw 值。这个补偿值的变量名为 **RootYawOffset**。

   Lyra 中只有在 Idle 状态才能触发 TurnInPlace 效果，**RootYawOffset** 值只有在 Idle 和 Stop 这两个状态下才会进行计算，其他状态下都只会向 0 进行插值（0 的话就是没有补偿，Root 朝向和角色 Actor 朝向一致）。

4. 如何不使用 BlendSpace 做出 Locomotion 的全向混合效果

   **Orientation Warping**

   <img src="D:\Projects\NOTES\images\image-20260115233909829.png" alt="image-20260115233909829" style="zoom: 67%;" />

   

5. 如何做急停反向移动（Pivot）
   双状态处理不断反向（ue没有状态机重进功能）

   ```
   a. 在StartState或者CycleState中，侦测到速度和加速度方向相反，则进入PivotState
   b. PivotState中，Tick判断加速度方向和速度方向是否一致
   c. 若不一致，表明还处于“刹车”过程中，当做Stop状态来处理，使用DistanceMatchToTarget函数
   d. 若一致，表明已经到达了Pivot点，并开始向反向移动，此时当做Start状态来处理，使用AdvanceTimeByDistanceMatching函数
   ```

6. 双足 IK 如何实现

   FootPlacement / ControlRig





### Lyra CommonUI 框架

[https://zhuanlan.zhihu.com/p/700905273]

- **Lyra 的 UI 层级结构**：Lyra 定义了四层 UI 结构，从低到高为 Game、GameMenu、Menu、Modal 层。层级配置在 GameUIPolicy 类中，由 GameUIManagerSubsystem 管理并在 DefaultGame.ini 文件中设置。每层有明确用途，像 Game 层放置特定的 LyraHudLayout UI，GameMenu 层放置局内非主界面 UI，Menu 层放置局外非主界面 UI，Modal 层作为通知层，这种分层结构让 UI 管理更有序。
- **各层 UI 的生成与交互**：Game 层 UI 借助 **GameFeatures** 动态添加，涉及一系列初始化与加载流程，还能设置返回键路由界面；GameMenu 层 UI 通过 **GA** 触发，依赖玩家状态相关设置，技能触发时生成 UI；Menu 层 UI 主要源于按钮点击；Modal 层用于放置通知及特定设置 UI。各层 UI 生成方式不同，且存在如返回键操作关联不同层 UI 等交互逻辑，共同构成完整 UI 体系。





## UObject

### 生成（UHT）

生成的代码主要分为两部分:

- 各种 Z_辅助方法用来构造出各种 UClass * 等对象；
- 另一部分是都包含着一两个 static 对象用来在程序启动的时候驱动登记，继而调用到前者的 Z_方法，最终完成注册

### 注册

Q&A:

1. 为什么构造（InnerRegister）要放到最前? 
   只先构造那些内建的;

2. UObjectLoadAllCompiledInDefaultProperties真正构造UClass（OuterRegister）时如果UClass中包含了别的UClass属性不会有依赖问题吗？
   不会，之前已经有UObjectProcessRegistrants内部构造过一遍所有注册的UClass了





## 智能指针

**SharedReferenceCount（强引用）** 和 **WeakReferenceCount（弱引用）** 在 **UE 智能指针体系（FSharedRef / FSharedPtr / TWeakPtr）** 中的**典型操作**“变化表”： 

| 操作                     | 对象类型     | SharedRefCount变化 | WeakRefCount变化 | 备注                                             |
| ------------------------ | ------------ | ------------------ | ---------------- | ------------------------------------------------ |
| `TSharedPtr`构造         | 新控制器     | **+1**             | **+1**           | 强引用诞生时弱引用也必须存在，以便后续弱指针查询 |
| `TSharedPtr`拷贝构造     | 已有控制器   | **+1**             | 不变             | 仅增加强引用                                     |
| `TSharedPtr`赋值(`=`)    | 已有控制器   | 旧-1 / 新+1        | 不变             | 先Add新再Release旧，顺序见前回答                 |
| `TSharedPtr`销毁         | 任意         | **-1**             | 不变             | 当减到0时立即**析构对象**，但控制器本身还活着    |
| 最后一个`TSharedPtr`销毁 | 控制器       | **-1→0**           | 不变             | 对象已析构；控制器等待弱引用归零                 |
| `TWeakPtr`构造           | 已有控制器   | 不变               | **+1**           | 纯弱引用                                         |
| `TWeakPtr`拷贝/赋值      | 已有控制器   | 不变               | **+1**           | 仅弱引用计数变化                                 |
| `TWeakPtr`销毁           | 任意         | 不变               | **-1**           | 当弱引用也归零→**释放控制器内存**                |
| `TWeakPtr::Pin()`成功    | 已有控制器   | **+1**             | 不变             | 临时提升为`TSharedPtr`                           |
| `TWeakPtr::Pin()`失败    | 控制器已失效 | 不变               | 不变             | 返回空指针                                       |
| `TSharedPtr::Reset()`    | 任意         | **-1**             | 不变             | 手动置空                                         |
| `TSharedRef`构造         | 新控制器     | **+1**             | **+1**           | 与`TSharedPtr`相同，但不可为空                   |
| `TSharedRef`拷贝         | 已有控制器   | **+1**             | 不变             | 同`TSharedPtr`                                   |

---

### 🔑 关键规则一句话
> **强引用归零→对象立刻析构；弱引用归零→控制器内存回收；** 
> **弱引用计数始终 ≥ 强引用计数（差值 = 纯弱引用持有者）。**



**MakeShared、 MakeShareable、 AsShared**  对比：

| 特性/函数                  | MakeShared                                                   | MakeShareable                                  | AsShared                                                     |
| -------------------------- | ------------------------------------------------------------ | ---------------------------------------------- | ------------------------------------------------------------ |
| 作用                       | 在**单个内存块**中同时分配对象和引用控制器，返回一个 `TSharedPtr<T>` | 将一个**已存在的裸指针**包装成 `TSharedPtr<T>` | 将**当前对象**（`this`）转换为 `TSharedRef<T>` 或 `TSharedPtr<T>` |
| 是否分配内存               | ✅（一次）                                                    | ✅（两次）                                      | ❌（不分配，仅包装已有对象）                                  |
| 是否支持私有构造           | ❌                                                            | ✅                                              | ✅（前提是类继承 TSharedFromThis）                            |
| 是否支持自定义删除器       | ❌                                                            | ✅                                              | ❌                                                            |
| 是否用于已有裸指针         | ❌                                                            | ✅                                              | ❌                                                            |
| 是否用于类内部获取自身引用 | ❌                                                            | ❌                                              | ✅                                                            |
| 性能                       | 高                                                           | 较低                                           | 高（无额外分配）                                             |

---



## 调试

[](https://www.bilibili.com/video/BV1iQ4y1j73A/)
[](https://www.bilibili.com/video/BV1st46z6ECv/) 
