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

![[v2-a1df1a24be903f7a4a3972fd67c7b603_1440w.jpg]]

多线程更新：

![img](D:\Projects\NOTES\images\v2-a9daf51b9c51669807a691be79ea4a51_r.jpg)



[[UnrealCircle深圳\]《黑神话：悟空》的Motion Matching | 游戏科学 招文勇_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1GK4y1S7Zw/)

![[image-20251116115328799.png]]



### 【实践】脚部IK (Foot Placement + Predict)





## AI

### 行为树

#### 不同情境下的执行逻辑

**1. 初始启动（首次执行）**

当 AI 控制器调用 `Run Behavior Tree` 时：

- **逻辑流程**：它会从 **Root** 开始，从左到右扫描第一个有效的分支。
- **搜索过程**：它会检查遇到的每一个选择器（Selector）、序列（Sequence）以及它们身上的装饰器（Decorator）。
- **落脚点**：直到找到第一个可以执行的任务节点（Task），将其标记为 `In Progress`，搜索停止。

**2. 任务进行中（Running / In Progress）**

这是最常见的情况，也是 UE 最省性能的地方：

- **逻辑流程**：**不再从 Root 开始搜索**。
- **执行状态**：行为树系统会直接“挂起”在当前正在运行的任务节点上。
- **Tick 逻辑**：只有当该任务显式开启了 `bNotifyTick`，或者它上方挂载了带间隔（Interval）的 **Service** 时，才会发生局部的 Tick。
- **黑板静默**：如果没有任何黑板值变化触发中断，树的其他部分完全处于“休眠”状态。

**3. 任务成功或失败（正常的执行流转）**

当当前任务调用 `Finish Execute`（Success 或 Failed）时：

- **逻辑流程**：**不会回到 Root**，而是向上回溯到父节点（如 Sequence）。
- **后续动作**：
  - 如果是 **Sequence** 且任务成功，它会向右移动到下一个子节点。
  - 如果是 **Selector** 且任务失败，它会尝试向右移动到下一个备选分支。
- **重新搜索**：只有当整个分支跑完，逻辑才会回到父级节点继续向右寻找。



#### 基于事件的打断（Decorator）

【打断行为可以提高性能，而不是每帧都从root开始】

**注册阶段 (Registering)**

当行为树运行并执行到带有黑板装饰器的节点时，`UBTDecorator_Blackboard` 会调用基类中的逻辑：

1. **获取黑板组件**：通过 `BehaviorTreeComponent` 找到关联的 `UBlackboardComponent`。
2. **绑定委托（Delegate）**：它会向黑板组件注册一个回调。在源码层级，这通常涉及到 `UBlackboardComponent::RegisterObserver`。它将装饰器自身的一个函数（即 `OnBlackboardKeyValueChange`）与特定的 **黑板键（Blackboard Key）** 绑定。

**触发阶段 (Firing)**

1. **黑板值变更**：当你在代码或行为树任务中调用 `SetValueAs...`（如 `SetValueAsObject`）时，黑板组件内部会检测新旧值是否相等。
2. **分发通知**：如果值确实发生了改变，黑板组件会遍历所有注册在该 Key 上的观察者（Observers）。

---

**打断流程 (ConditionalFlowAbort) 的详细逻辑**

一旦监听到值改变，装饰器(通常挂载在一个黑板键上) 必须决定是否要“掐断”当前正在执行的任务。这就是 `ConditionalFlowAbort` 的职责。

它会根据你在编辑器中设置的 **Observer Aborts** 选项（None, Self, Lower Priority, Both）来执行不同的逻辑：

逻辑判断：是否满足打断条件？(OnValueChange和OnResultChange)

在调用打断前，它会先执行一次 `CalculateRawConditionValue`。

- 比如如果是OnResultChange, 当你设置“当 `Enemy` 不为空时打断”。如果黑板值变动是从 `EnemyA` 变成 `EnemyB`（依然不为空），虽然触发了监听，但逻辑判定没变，则不会打断。

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
   
      

#### 节点的单例问题

> 详细请参考：[【图解UE4源码】AI行为树系统 其一 行为树节点的单例设计 - 知乎](https://zhuanlan.zhihu.com/p/369100301)

问题：行为树节点：默认一棵行为树资源对应一个单例（不管是不是不同的ai跑）

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


### 状态树

#### 初始化及Tick逻辑

![[StateTree运行逻辑.png]]
参考图来自：[[UOD2022\]从行为树到状态树 | Epic 周澄清_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1ed4y1b7Zk/)

**一、 初始化流程（Initialization）**

当 StateTree 开始运行（StartTree）时，系统会进行数据准备和初始状态选择。

1. **Evaluator 数据准备**：
   - 首先调用所有 Evaluator 的 `TreeStart`，随后执行一次 `Tick`。
   - **目的**：在状态选择前，预先从环境中提取并处理好必要的数据（如感知到的敌人信息、玩家距离等）。
2. **初始状态选择（SelectState[下面有详解] ）**：
   - 目标状态默认为 **Root**。
   - **条件判定**：自根向叶检查 `Enter Conditions`，判断哪些子状态可以被激活。
   - **Task 入场**：一旦确定了激活路径，系统会**自根到叶（正序）**依次调用路径上每个 State 中 **Task** 的 `EnterState`。

------

**二、 运行时每帧更新（Tick）**

StateTree 的 Tick 分为两个主要部分：数据更新/Task执行（**TickUpdateTasksInternal**）与 过渡判定（**TickTriggerTransitionsInternal**）。

1. **TickUpdateTasksInternal**

   1.  `TickTasks` 中进行 **Evaluator** 更新（更新全局或上下文数据）、**GlobalTasks**执行和**自根到叶**进行所有**ActiveState中Task**的**Tick**。
      **返回值”坏消息优先“原则**：结果是上面所有Task执行结果的**聚合**，且`Failed` > `Succeeded` > `Stopped` > `Running`
   2. 一旦上面的Tick最终结果不是Running，系统会调用`Context.StateCompleted` ，内部**自叶到根（逆序）**调用所有已激活状态中 Task 的 `StateCompleted`，以及如果需要，也调用Condition上的state change events.

2.  **TickTriggerTransitionsInternal**

   1.  MaxIterations次迭代，每次调用**TriggerTransitions**（内部检查顺序为**从叶到根**逆序）

  ```cpp
  bool FStateTreeExecutionContext::TriggerTransitions()
  {
	//1. Process transition requests. Keep the single request with the highest priority.
	//2. Process tick/event/delegate transitions and tasks. TriggerTransitions, from bottom to top.
	// If delayed,
	//	If delayed completed, then process.
	//	Else add them to the delayed transition list.
	//3. If no transition, Process completion transitions, from bottom to top.
	//4. If transition occurs, check if there are any frame (sub-tree) that completed.
	...
  }
  ```

内部调用**RequestTransitionInternal**，进一步在内部调用SelectState（转换到一个父状态（如 "Combat"）必须通过 `SelectState` 递归决定最终进入哪个叶子子状态（如 "Melee_Attack"））

   2. 如成功过渡：
      - 调用Context的**ExitState**，内部从叶到根逆序调用Task的ExitState
      - 调用 **EnterState**（同初始化时）
        



**SelectState**逻辑

```cpp
bool FStateTreeExecutionContext::SelectState(const FSelectStateArguments& Args, const TSharedRef<FSelectStateResult>& OutSelectionResult)
{
	// 1. Find the target root state.
	// 2. If the the target frame is active, then copy all previous states from the previous frames.
	//   If any state inside the frame is completed (before the target), then the transition fails.
	//   See EStateTreeStateSelectionRules::CompletedStateBeforeTargetFailTransition.
	// 3. In the target frame, start adding states that match the source/target.
	//   If the state is completed (before the target), then the transition fails.
	//   See EStateTreeStateSelectionRules::CompletedStateBeforeTargetFailTransition.
	// 4. New/Sustained states need to be reevaluated (see SelectStateInternal).
	// 5. Else, handle fallback.
	// 
	// Source is from where the transition request occurs.
	//  The source frame is valid but the source state can be invalid. It will be the top root state if needed.
	//  It doesn't need to be in the same frame as the target.
	// Target is where we wish to go. The selection can stop or select another state (depending on the state's type).
	// Selected is the selection result.
	//
	// ExitState: If the state >= Target and is in the selected, then the transition is "sustained" and the instance data is untouched.
	// ExitState: If the state not in the selected (removed state), then the transition is "changed" and the state is removed.
	// EnterState: If the state >= Target and it is in the actives, then the transition is "sustained" and the instance data is untouched.
	// EnterState: If the state >= Target and it is not in the actives (new state), then the transition is "changed" and the state is added.
	// 
	// Rules might impact the results.
	// In the examples,  "New State if completed" is only valid if EStateTreeStateSelectionRules::CompletedTransitionStatesCreateNewStates
	// Examples: Active | Source | Target | Selected
	//
	//           ABCD   | ABCD   | ABCDE  | ABCDEF, ABCD'EF
	//  New D if completed, always new EF.
	//  ExitState: Sustained -. Changed -, D. EnterState: Sustained -. Changed EF, D'EF.
	//
	//           ABCD   | AB     | ABI    | ABIJ, AB'IJ
	//  New B if completed, always new IJ.
	//  ExitState: Sustained -. Changed CD, BCD. EnterState: Sustained -. Changed IJ, B'IJ.
	// 
	//           ABCD   | AB     | ABC    | ABCD, ABCD', ABC'D', AB'C'D'
	//  New D, CD, BCD if completed.
	//  ExitState: Sustained CD, C, -, -. Changed -, D, CD, BCD. EnterState: Sustained CD, C, -, -. Changed -, D', C'D', B'C'D'.
	//
	//           ABCD   | ABC     | AB    | ABCD, ABCD', ABC'D', AB'C'D'
	//  New D, CD, BCD if completed.
	//  ExitState: Sustained BCD, BC, B, -. Changed -, D, CD, BCD. EnterState: Sustained BCD, BC, B, -. Changed -, D', C'D', B'C'D'.
	// 
	//           ABCD   | AB     | AX     | AXY
	//  Source is not in target. New XY.
	//  ExitState: Sustained -. Changed BCD. EnterState: Sustained -. Changed XY
	//
	//           ABCD   | AB     | AB     | ABCD, ABCD', ABC'D', AB'C'D'
	//  Source is target. New D, CD, BCD if completed.
	//  ExitState: Sustained BCD, BC, B, -. Changed -, D, CD, BCD. EnterState: Sustained BCD, BC, B, -. Changed -, D', C'D', B'C'D'.

```

1. 路径寻找与合法性检查 (Step 1-3)

- **Target Root State**: 确定目标状态所属的帧（Frame）。State Tree 支持 Linked Tree，因此目标可能在子树中。
- **CompletedStateBeforeTargetFailTransition**: 这是一个关键的安全阀。如果目标路径上的某个中间状态已经处于 `Completed` 状态，且规则不允许跳过，则整个转换请求失败。这保证了逻辑的原子性，防止进入一个已经失效的父状态。

2. 重评估逻辑 (Step 4: SelectStateInternal)

- 新状态（New States）或需要重评估的状态会进入 `SelectStateInternal`。这是真正的**递归探测点**。系统会运行该状态的 `Enter Conditions` 和 `Selector` 逻辑，决定最终激活哪个子状态。

3. 状态生命周期：Sustained vs Changed

| **状态分类**         | **定义**                                                     | **行为 (Execution)**                         | **内存 (Instance Data)**              |
| -------------------- | ------------------------------------------------------------ | -------------------------------------------- | ------------------------------------- |
| **Sustained (维持)** | 状态在 `Active` 和 `Selected` 路径中都存在，且在 `Target` 之下（或作为祖先）。 | 不触发 `Exit/Enter`。                        | **保留**。Task 的成员变量值保持不变。 |
| **Changed (变更)**   | 状态不在新路径中，或者是新加入路径的。                       | 触发 `ExitState` (旧) -> `EnterState` (新)。 | **重置/初始化**。重新从模板拷贝数据。 |

4. 示例场景

场景 A：向上跳转再向下探测

```
Active: ABCD | Source: AB | Target: ABI | Selected: ABIJ
```

- **推理**：
  1. 当前路径是 `A->B->C->D`。
  2. 我们在 `B` 状态触发了一个转换，目标是 `I`。
  3. 系统保留 `A` 和 `B`（Sustained）。
  4. 系统销毁 `C` 和 `D`（Exit）。
  5. 系统进入 `I`，并根据 `I` 的选择逻辑自动选择了子状态 `J`（Enter）。
- **结果**：路径变为 `ABIJ`。

场景 B：同级或自身重入

```
Active: ABCD | Source: AB | Target: ABC | Selected: ABCD'
```

- **推理**：
  1. 目标是 `C`，而 `C` 已经在当前路径中。
  2. 如果 `D` 已经完成（Completed）且设置了 `CreateNewStates`，系统会 Exit `D` 并 Enter 一个全新的 `D'`。
- **设计动机**：这解决了“循环任务”的需求。比如在“巡逻”状态下，完成一个路点后再次请求进入“巡逻”，系统会刷新子状态的执行进度。

场景 C：跨分支跳转

```
Active: ABCD | Source: AB | Target: AX | Selected: AXY
```

- **推理**：
  1. `Source` 的目标 `X` 不在当前路径 `B` 分支下。
  2. 找到最近公共祖先 `A`。
  3. 维持 `A`，销毁 `B, C, D`。
  4. 进入 `X, Y`。



#### 数据绑定与属性传播机制

##### 概览

在 UE5 的 **StateTree** 体系中，数据绑定（Data Binding）与传播机制是其区别于行为树（Behavior Tree）最核心的特性之一。它通过显式的**属性链接（Property Link）**代替了行为树中基于字符串索引的黑板（Blackboard）模式。



1. 数据绑定的核心概念：Schema 与 Context

在 StateTree 中，数据传播的起点是 **Schema（架构）**。

- **Schema 定义上下文**：每个 StateTree 必须选择一个 Schema（如 `StateTreeGameplayTasksSchema`）。Schema 规定了该树能访问哪些**外部数据**（Context Data），例如 `AActor`、`UAIPerceptionComponent` 或自定义的 `UObject`。
- **Context 注入**：当 StateTree 开始运行（初始化阶段），外部系统会将这些具体的对象实例注入到 StateTree 的**实例数据（Instance Data）**中。

2. 数据传播路径：Evaluator -> Task

第一步：Evaluator 的数据提取（数据源）

- **初始化阶段**：调用 `Evaluator` 的 `TreeStart` 和 `Tick`。
- **传播逻辑**：Evaluator 作为“传感器”，从 Context（如感知组件）中提取原始数据，并将其转化为 StateTree 内部易于处理的属性（如 `bIsEnemyVisible` 或 `TargetActor`）。
- **数据绑定**：Evaluator 定义的这些输出属性，可以被后续的 Task 或 Transition 直接绑定。

第二步：属性绑定（Property Binding）

- **显式链接**：在 StateTree 编辑器中，你可以直接将 Task 的输入参数“连线”到 Evaluator 的输出参数上。
- **底层机制**：这在底层是通过 **`FStateTreePropertyBinding`** 实现的。它不依赖黑板键的字符串查询，而是直接记录属性在内存中的偏移量（Offset），因此访问速度极快。

------

##### 详细分析

###### 数据布局与内存模型

###### 数据绑定与上下文

四种变量数据

- **Context**类型数据存储在FStateTreeExecutionContext中的ContextAndExternalDataViews中，被Schema中的SetContextDataByName函数所手动设置（在StartTree时）。

- **Input/Output/Parameter**类型变量则是通过PropertyBinding机制绑定，具体区别为：
  Input只能绑定到外部如 Context、Evaluator或上层State 的 Output、其他状态的 Parameter）的外部信息；
  Output不“绑定”到外部源；
  Parameters包含Task/Condition 局部 Parameter或Global Parameters (资产全局参数)。



### Behavior Tree vs. State Tree

1. 表格总结

| **特性**     | **Behavior Tree (行为树)**                                   | **State Tree (状态树)**                                      |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **设计范式** | **基于决策 (Decision-based)**。每帧（或事件触发）从根节点重新评估，寻找可执行的叶节点。 | **基于状态 (State-based)**。它是分层状态机 (HFSM) 的进化版，明确“当前正处于哪个状态”。 |
| **运行机制** | 依靠 **Selector/Sequence** 控制流，通过失败/成功反馈来回溯。 | 依靠 **Transition (过渡)** 驱动。状态切换是显式的，只有满足条件才进行状态跳转。 |
| **底层架构** | 基于对象: 树上的每个节点都是一个独立的对象，大型行为树会产生数以千计的小对象。 | 基于数据: 将状态和数据存储在连续的内存块（Task Data) 中      |
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



### 【实践】用状态树重现Lyra Bot AI

之前的文章[UE5 行为树与状态树浅析及对比 - 知乎](https://zhuanlan.zhihu.com/p/1999137935317033648)对状态树的一些细节及相比行为树的优势做了一些分析。本篇文章则进入到实践环节。用UE5.7的状态树重现Lyra示例项目中原本用行为树开发的AI敌人逻辑。

> Github仓库：https://github.com/Rimor4/LyraStarterGame/tree/StateTree


#### 重现步骤

**Evaluator**

将原行为树中部分需要高频监测的 **Service**（如 CheckAmmo、Shoot）转换为 StateTree 的 **Global Evaluator**。这样可以将如 `OutOfAmmo` 等需要全局更新的变量逻辑剥离出来，统一在 Evaluator 中处理，而非分散在各个节点中。

**Condition**

用StateTree中的状态Enter Condition和Transition Condition，即状态之间的转换条件模拟行为树中的Decorator

**Task**

我自己的实现Task可以分为两种：

- 一种是类似行为树中的**Service**，一般放在非叶节点（最后也可以和第二种task共同放在叶状态中），不做打断，隔一段时间就会运行
- 一种是行为树叶节点的task，一般也放在状态树的叶状态。

**实现Service**

见下文“踩到的坑”第二条

**实现Selector**

为了复现行为树中的selector逻辑，主要用状态之间的OnStateFailed类型过渡方式。
![image-20260130223757007](D:\Projects\NOTES\images\image-20260130223757007.png)

**使用SubTree等方法简化状态**

实践中先按照行为树的连法将节点一一对应地转换到状态树，但最终有部分分支逻辑是重复的，以及也有单个分支过深而丢失了状态树扁平化状态的优势的问题。

解决方法便是

1. 将多个状态简化、合并为一个，task可以聚合在同一个状态内部。
2. 将大部分相同的逻辑用SubTree + Link（两种状态类型）方式，像函数一样提取为子树，逻辑不同的地方用参数加以区分。

![[image-20260201180122473.png]]

最终的LyraBot状态树
![[image-20260201173739210.png]]



#### 踩到的坑

这一部分是本文的重点，也是学习一个系统/框架时花费精力最多的地方。希望可以给正在学习状态树的朋友提供一些有限的帮助，以免将时间花费到不必要的debug上。

1. **问题**：如下一个“最简”StateTree，没有发生Tick。

   ![[image-20260127113752371.png]]
   **解决办法**：叶子状态必须有Task
   **原因分析**：根据 StateTree 的调度机制，如果状态树 StartTree 后，激活状态（Root 或 Idle）中没有任何 Task、Transition 且未收到 Event，调度器判定当前没有需要处理的逻辑，便会在 **GetNextScheduledTick** 中将状态树置为 Sleep，导致后续 Tick 不再执行。

   **进一步疑问**：Idle 状态明明配置了 Transition to Root，为什么没触发？
   这是因为该 Transition 的触发条件是 OnStateCompleted。由于 Idle 状态内没有配置任何 Task，状态机无法从 Task 获得 Succeed 或 Failed 的执行结果，因此该状态默认保持 Running 状态。既然状态从未“完成”，OnStateCompleted 自然无法被触发》

**StateTree 调度机制核心**
在`StartTree`初始化那一帧的`Context.Start`以及之后的每帧`Context.Tick`【主体都为FStateTreeExecutionContext】之**后**，**GetNextScheduledTick** 函数决定当前状态树是否tick以及下次tick的时间。有四种返回值（判断优先级自上而下），对应情况如下：

- EveryFrames(当前帧需要Tick）：
  1. Schema配置为强制阻止整个调度机制（**IsScheduledTickAllowed**为false）
  2. **全局任务请求**：如果 `StateTree` 资产配置为需要 Tick 全局任务（`DoesRequestTickGlobalTasks`），且当前有待处理事件。
  3. 激活状态ActiveState有任务正在Tick（`DoesRequestTickTasks`）
  4.  `ScheduledTickRequests`（一般为**DelayTask**） 设置了 `ShouldTickEveryFrames`
  5. 零值 CustomTickRate：如果某个状态设置了 `CustomTickRate` 但其值为 `0.0f`（或负值），它会提升优先级至 EveryFrames
- NextFrame（延迟到下一帧再Tick）：
  1.  `ScheduledTickRequests` 中设置了`ShouldTickOnceNextFrame`的请求
  2. 此时有**待处理转换**（TransitionRequests）
  3. 队列中有**未处理事件** (Events)
  4. **挂起的完成状态** (Pending Completed State)：如果有任务或状态已经标记为 `Completed`，需要下一帧触发后续的清理或转换逻辑。
- CustomTickRate (自定义频率/延迟)
  1. 状态节点配置了 `bHasCustomTickRate`。若多个状态都有，则取最小值
  2. 如果有ScheduledTickRequests，且不为ShouldTickEveryFrames/ShouldTickOnceNextFrame，CustomTickRate设为该请求的TickRate和上面取最小值。
  3. **延迟转换 (Delayed Transitions)**：当一个 Transition 设置了延迟时间（Delay），CustomTickRate设为剩余时间`TimeLeft`）和上面取的最小值。
- Sleep（不触发Tick）
  

2. **状态树中没有行为树中的Service节点**

   ![image-20260128213830741](D:\Projects\NOTES\images\image-20260128213830741.png)

​	即没有像这样支持间隔时间执行的service task，需要自己实现，最后实现了一个cpp类：
主要函数及重载：
![image-20260201172218925](D:\Projects\NOTES\images\image-20260201172218925.png)

数据结构：
![image-20260201170546456](D:\Projects\NOTES\images\image-20260201170546456.png)

而且目前引擎提供的RunEnvQuery任务有很多用起来不方便的地方：一是不支持蓝图继承（底层是结构体），二是Result参数绑定不到其他状态中，只能提升为全局参数，再把全局参数绑定到用到的地方（回归行为树了属于是）。 

具体使用：
![image-20260201173243060](D:\Projects\NOTES\images\image-20260201173243060.png)

3. ”**Failed to select initial state**“报错

   ![image-20260128230548109](D:\Projects\NOTES\images\image-20260128230548109.png)

   ![[image-20260128234654944.png]]

在开发时时刻注意要像switch case一样留一个default case（可以为idle），以免状态树初始化时无法选择初始状态。

这里其实还有一个**inactive parent state**的警告，原因是初始化时，任何state都不是Active的，当进行test conditions时会对source data进行拷贝验证，没有通过则会发出警告，代码如下，不言自明：

![image-20260128232716713](D:\Projects\NOTES\images\image-20260128232716713.png)

![image-20260128232720601](D:\Projects\NOTES\images\image-20260128232720601.png)

但因为是警告而不是报错，所以可以忽略(bushi
一种方法是将相关参数提升为 Global Parameter，从而绕过 Context 初始化的检查。不过个人感觉还是现在这种方法更加优雅和符合直觉。

4. 把框架内建的RunEnvQuery Task作为像行为树中的Service节点那样，放在**非叶状态**中了。导致该RunEnvQuery的每帧触发StateComplete，进而导致持续性任务（下面的Moving）根本无法执行。

   ![[image-20260130000758591.png]]

同理，自己写的Service节点放到非叶节点也要注意不要在内部触发**FinishTask**(succeed/failed)。
如果逻辑需要必须触发，比如为了模拟Selector（见上文“重现步骤”），一定要检查清楚有没有子状态可能会被这个查询带来的FinishTask影响。比如我这里遇到了一个因为AI视野内找不到敌人，即DetectEnemy状态中以FinishTask(failed)结束，导致下面的SearchWeapon任务无法执行，进而导致每帧Moving的输入值都是坐标（0,0,0）
![[image-20260130225328763.png]]

5. 父状态逻辑被子状态“截胡”

   这里需要特别注意 **Transition 的评估顺序**。StateTree 每帧检测 Transition 时是**自底向上（从叶子节点向根节点）**进行的。这意味着，处于激活链末端的子状态/叶状态定义的过渡条件拥有更高优先级。
   如果子状态满足了某个过渡条件并发生了跳转，父状态的过渡条件甚至不会被评估。因此，如果必须在父状态（非叶节点）设置过渡，务必理清与子状态过渡条件的互斥关系。

   ![[image-20260130222310988.png]]

   



#### 学习过程中学到的其他知识

##### 通用EQS Task的实现

![](D:\Projects\NOTES\images\image-20260128190332866.png)

因为`RunEQS`后得到的返回结果中可能有Actor和Vector类型（以及对应的数组类型），为了支持这样的”泛型返回值“。在代码逻辑中用到`GetPtrTupleFromStrongExecutionContext`函数，它基于UE InstanceStruct（简单理解为支持多态的UStruct）、PropertyBinding/Ref（前者类似C++中的按值传递，后者类似按引用传递）等基建，能够在运行时，找到FStateTreePropertyRef变量内存对应的实际变量（如Actor）地址，并供这里的Task赋值。
![image-20260128102317718](D:\Projects\NOTES\images\image-20260128102317718.png)

但是正如前文所述，有很多使用不便的地方。



##### 兼容蓝图类Task

再比如，由于状态树框架内部的Task都是以`FStateTreeTaskBase`这样的结构体存储的，那么在编辑器中添加Task时，如何支持蓝图类（蓝图类一定是UClass）Task呢？

查看源码发现为支持蓝图Task，有一个`UStateTreeTaskBlueprintBase`的U类，而它的旁边有一个以`FStateTreeTaskBase`为基类的`FStateTreeBlueprintTaskWrapper`，在ide中搜索相关引用发现，具体实现方式是这样的：

从在编辑器中点击”Add Task“时开始，首先当我们选择`UStateTreeTaskBlueprintBase`的子类后，引擎内部通过下面的方式，即依赖FInstanceStruct的多态结构体特性，将Task对应的`FStateTreeEditorNode`的内部数据（FInstanceStruct类型）显式初始化成了`FStateTreeBlueprintTaskWrapper`，并在后续通过Wrapper中的EnterState、ExitState等方法转发到蓝图类的对应函数。

总体来说，是一个适配器模式的很好应用。

![](D:\Projects\NOTES\images\image-20260128194938490.png)

![image-20260128190152267](D:\Projects\NOTES\images\image-20260128190152267.png)



##### InstanceData数据模型相关

发现在实现Task时，引擎内部采用将变量存放到一个FxxxInstanceData结构体中（如FStateTreeRunEnvQueryInstanceData）而不是直接放到Task类的成员变量中：

![image-20260128194530859](D:\Projects\NOTES\images\image-20260128194530859.png)

![image-20260128194551407](D:\Projects\NOTES\images\image-20260128194551407.png)

原因类似[](UE.md#节点的单例问题)，除了解决**单例/多例问题**，还有支持**动态绑定**等好处。

结构体类Task是如上定义的，那么蓝图类（U类）呢，答案仍是藏在之前说的`FStateTreeBlueprintTaskWrapper`中，

![[image-20260128200553632.png]]

虽然存的是UClass，但是显然，编辑器侧发现是UClass后会NewObject出来类实例，存储在InstanceObject中。

![[image-20260128200923910.png]]

当然，InstanceData的最大好处是**内存连续性**：
传统的行为树中，每个节点通常是一个独立的 UObject，这导致其成员变量分散在堆内存的各个角落，容易产生 Cache Miss。
而状态树采用了数据与逻辑分离的设计。每个运行实例仅需持有一个 FStateTreeExecutionContext。其中的 **InstanceDataStorage** 是一块紧凑的连续内存，集中存放了编译期确定的所有数据（包含 Global Parameters、各 State 和 Task 的 InstanceData）。这种布局极大地提高了 CPU 缓存命中率，也是 StateTree 高性能的关键所在。

![image-20260128202150438](D:\Projects\NOTES\images\image-20260128202150438.png)





#### 目前状态树仍存在的问题

1. 其次和其他AI系统如EQS**兼容性**不如行为树，比如EnvQueryContext中想要用状态树中的参数，目前只能通过在状态树逻辑中设置黑板值，再在Context中读黑板。因为目前StateTree框架中没有提供便利的数据读取相关接口。

   ![image-20260129153047203](D:\Projects\NOTES\images\image-20260129153047203.png)

​	当然如果有可行的解决方案，欢迎各路大佬现身评论区:)

2. **打断关系**不好维护/实现（这也是我目前觉得最大的缺点），不像行为树有很直观的装饰器节点用于优先级+事件打断。状态树的打断应该只能通过为状态之间添加过渡条件来实现。而过渡本身机制需要我们时刻关注来自子状态的过渡条件和整个激活状态链上可能存在的任何Task（包括Global）的执行结果/是否调用FinishTask。本人就在实践过程中在这一点上修了好几个难查的bug。

​	具体来说，比如根据下面的状态树，当敌人初始化时由于DetectEnemy的EQS还没运行完毕，于是进入到低优先级的SearchNew Weapon，但是如果此时有敌人就在脸上，AI就会仍然走到武器那里，直到触发当前Move To Task的State Complete后才会评估进入高优先级的Shoot。
​	![[image-20260130202222869.png]]

这一点最后我的解决方法是在这种长时间运行Task的叶子状态中加入OnTick/Event的跳转。一般OnTick检查Global Paramter，OnEvent则用事件解耦。

##### 总结

目前看来，如果只针对常见的游戏AI行为控制，行为树仍有很多不可替代的地方，比如==方便的打断机制==，和其他AI基建的兼容性等。

但状态树相比行为树最大的优势还是在于

1. **高性能**（数据在内存紧凑排布）。
2. **数据安全性**：通过显式的 **Data Binding**（数据绑定）替代了行为树中隐式的全局黑板（Blackboard）读写，数据流向更清晰，不易出错。
3. 表达一个完整的**状态**（包括子树【类似函数】的复用）的能力
4. **通用性强**：StateTree 不仅限于 AI，还非常适合处理 **Game Logic** 或 **UI Flow**（如复杂的界面导航与状态切换）。这类逻辑如果强行用行为树表达会显得非常别扭，而状态机模型则天然契合。



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

![[IMG_20251115-153540054.png]]  

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

1. _EnhancedInput_ 框架：InputStack、InputMappingContext（优先级） InputAction 回调触发机制

2. 引擎运行流程 [UE5 -- 引擎运行流程（从 main 到 BeginPlay）](https://zhuanlan.zhihu.com/p/577433224)

3. Subsystems [《InsideUE4》GamePlay 架构（十一）Subsystems](https://zhuanlan.zhihu.com/p/158717151)

4. Lyra Gameplay 框架总览 [UE5 新项目 Gameplay 框架设计（以 Lyra 为例）](https://zhuanlan.zhihu.com/p/614718286)

   

### GameFeatures 架构

[https://zhuanlan.zhihu.com/p/467236675]

1. Lyra 中的 GameFeature:
   GameMode -启动-> GameState上挂载ExperienceManagerComponent -> 加载Experience -封装-> GameFeatureAction..(以及 PawnData)
   
   

### ModularGameplay 架构

给原有的**PlayerController**等大类外面套一层子类**ModularPlayerController**，在初始化时用 `UGameFrameworkComponentManager::AddGameFrameworkComponentReceiver(this);` 侵入式地注册为GFCM中的Recevier；

之后继承**UGameFrameworkComponent**的各种Component (如 **ULyraControllerComponent_CharacterParts** -> **UControllerComponent**(表示绑在ModularPlayerController中的comp) -> UGFC) 在Controller中调用 **ReceivePlayer**（初始化）和**PlayerTick**（更新）等生命周期函数。

相比原先的Actor - Component组合方案，主要现在可以支持动态添加（通过Experience->**GameFeature**)



### Lyra 动画模块

1. 模块化动画: 同种动画状态，但状态内容不同时，如何解耦不同的状态内容（使用**动画层**解耦）![[v2-7a66f1bf107eb0ec877d5644a04c02d0_r.jpg]]

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

   ![[image-20260115233909829.png]]

   

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





## 类型系统（UObject）

![[image-20260128094725848.png]]

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

### TObjectPtr

不仅仅是“为了安全”，更主要的原因涉及 **编辑器性能（加载优化）** 和 **对象句柄抽象（Object Handle Abstraction）**。

**资源加载**

- **原始指针的问题**：
  在 UE4 中，当你加载一个资源（比如一个 Blueprint 类）时，序列化器会读取该类引用的所有 UPROPERTY。如果是一个原始指针 UTexture* MyTex，引擎**必须**立刻找到并加载那个纹理，以便把内存地址填进指针里。
  这会导致**级联加载（Cascading Load）**：加载 Asset A -> 触发加载 Asset B -> 触发加载 Asset C... 打开一个文件可能会导致半个项目的资源被强行加载到内存中。
- **TObjectPtr 的解决方案**：
  在编辑器（Editor）构建中，TObjectPtr 并不仅仅存一个内存地址。它能够存储一个 **“未解析的句柄（Handle）”** 或者 **“对象路径”**。
  当序列化加载一个 Asset 时，TObjectPtr 可以保持“未解析”状态。**它知道它指向谁**，但它**没有真正加载那个对象**。只有当你**代码里第一次访问**它（例如调用 MyPtr->GetName()）时，它才会触发“解析（Resolve）”，此时才真正去加载对象并获取内存地址。

**底层**

**UnResolved**状态存的是

- [ Handle ID (63 bits) ] + [ 1 (1 bit) ]
- **存储内容**: 存储的是一个全局 **Object Handle ID**。
- 用这个 Handle ID 去全局表（FObjectHandlePackageMap）里查，就能查到这个对象对应的 PackageName 和 ObjectName

**Resolved** 状态存的就是对象在 RAM 中的64位物理地址



## 智能指针

**SharedReferenceCount（强引用）** 和 **WeakReferenceCount（弱引用）** 在 **UE 智能指针体系（FSharedRef / FSharedPtr / TWeakPtr）** 中的**典型操作**“变化表”： 

| 操作                    | 对象类型   | SharedRefCount变化 | WeakRefCount变化 | 备注                        |
| --------------------- | ------ | ---------------- | -------------- | ------------------------- |
| `TSharedPtr`构造        | 新控制器   | **+1**           | **+1**         | 强引用诞生时弱引用也必须存在，以便后续弱指针查询  |
| `TSharedPtr`拷贝构造      | 已有控制器  | **+1**           | 不变             | 仅增加强引用                    |
| `TSharedPtr`赋值(`=`)   | 已有控制器  | 旧-1 / 新+1        | 不变             | 先Add新再Release旧，顺序见前回答     |
| `TSharedPtr`销毁        | 任意     | **-1**           | 不变             | 当减到0时立即**析构对象**，但控制器本身还活着 |
| 最后一个`TSharedPtr`销毁    | 控制器    | **-1→0**         | 不变             | 对象已析构；控制器等待弱引用归零          |
| `TWeakPtr`构造          | 已有控制器  | 不变               | **+1**         | 纯弱引用                      |
| `TWeakPtr`拷贝/赋值       | 已有控制器  | 不变               | **+1**         | 仅弱引用计数变化                  |
| `TWeakPtr`销毁          | 任意     | 不变               | **-1**         | 当弱引用也归零→**释放控制器内存**       |
| `TWeakPtr::Pin()`成功   | 已有控制器  | **+1**           | 不变             | 临时提升为`TSharedPtr`         |
| `TWeakPtr::Pin()`失败   | 控制器已失效 | 不变               | 不变             | 返回空指针                     |
| `TSharedPtr::Reset()` | 任意     | **-1**           | 不变             | 手动置空                      |
| `TSharedRef`构造        | 新控制器   | **+1**           | **+1**         | 与`TSharedPtr`相同，但不可为空     |
| `TSharedRef`拷贝        | 已有控制器  | **+1**           | 不变             | 同`TSharedPtr`             |

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



## PropertyBinding/Ref

### PropertyBinding

【可以理解为C++中的按**值**参数传递】

一种**单向数据同步**机制。

- Link：编辑器中通过绑定变量，记录源数据和目标数据的FPropertyBindingPath（理解为地址）
- Runtime：在特定时机（如 状态树Exit State）用CopyProperty等函数将源数据的值**拷贝**到目标的成员变量中。

### PropertyRef

【可以理解为C++中的按**引用**参数传递】

对应状态树框架中的FStateTreePropertyRef。不存储具体的值，而是存储一个**指向源数据地址的指针（或偏移信息）**。



## 调试

[[UFSH2023]总有一个你不知道的虚幻引擎调试技巧 | 陈拓 Epic Games_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1iQ4y1j73A/)
[[UFSH2025]超越PrintString: 虚幻引擎调试工具 | Matt oztalay Epic Games 开发者关系高级 TA(官方字幕)_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1st46z6ECv/)

