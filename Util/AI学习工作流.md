# AI 学习工作流

## UE（学习 + 博客输出）

以学习StateTree为例（实践项目：重现Lyra 示例项目中的基于行为树的Bot AI逻辑）

### 第一阶段：工程制作提效 (Dev Phase)

#### ~~1. AI 辅助策略：代码与逻辑迁移~~

- ~~**C++ 任务迁移 (Code Conversion):**~~

  ~~StateTree 的 `FStateTreeTaskBase` 和 BT 的 `UBTTaskNode` 结构不同。~~

  - ~~**操作:** 将 Lyra 的某个 BT Task (如 `BTT_FindPickup`) 的 `.h` 和 `.cpp` 代码复制给 AI。~~
  - ~~**Prompt:** “我正在将 Lyra 的 AI 从 Behavior Tree 迁移到 StateTree。这是一个 BT Task 的代码，请帮我重构它为一个 `FStateTreeTaskCommonBase` 的子类。请注意保留核心逻辑，但将 `Blackboard` 的读写操作转换为 StateTree 的 `Context` 或 `Parameters` 绑定形式。”~~

- ~~**逻辑映射 (Logic Mapping):**~~

  - ~~**操作:** 描述 BT 的一段逻辑（例如：Selector 节点下挂着“寻找掩体”和“射击”）。~~
  - ~~**Prompt:** “在 Behavior Tree 中，我用 Selector 来优先执行‘寻找掩体’，失败后执行‘射击’。在 StateTree 中，我应该如何设置 State 的 `Transition` (转换) 条件来实现相同的优先权逻辑？是使用 Utility 还是固定的 Condition？”~~

#### 2. 非 AI 策略：工具与调试

- **双屏对照法 (The Side-by-Side View):**

  不要凭空重写。打开 Lyra 的 BT 资产在左屏，新建 ST 资产在右屏。StateTree 的核心在于**Data Binding (数据绑定)**。

  - *重点:* 观察 BT 中 Blackboard Key 是如何传递数据的，在 ST 中你需要通过 `Context` (上下文) 或 `Global Tasks` 来暴露这些数据。

- **利用 Visual Logger (VisLog):**

  这是调试 AI 最重要的非 AI 工具。

  - 在迁移过程中，AI 经常会“发呆”或状态卡死。
  - **技巧:** 确保你的 StateTree Task 实现了 `DescribeBody` 方法（C++），这样在 VisLog 里就能看到每一帧当前状态的参数值。

- **Gameplay Debugger (`'` 键):**

  实时查看 StateTree 的激活状态。确保开启了 StateTree 的调试分类，这能让你在运行游戏时直接看到当前 Bot 处于哪个 State。

------

### 第二阶段：知识内化与记录 (Learning Phase)

不要等到做完了再写，那样会遗漏很多细节。

#### 1. AI 辅助策略：概念解释与坑点预警

- **即时概念查询:** 当你对 StateTree 的 `Schema`、`Evaluators` 或 `Context` 感到困惑时，不要只查文档（文档可能不全）。
  - **Prompt:** “用通俗易懂的语言解释 UE5 StateTree 中 `Evaluators` 的作用，它和 Behavior Tree 的 `Service` 有什么区别？在 Lyra 的语境下，我可以用它来持续更新‘最近的敌人’位置吗？”
- **错误日志分析:** 遇到 `Ensure` 报错或编译错误，直接丢给 AI。StateTree 的数据绑定错误通常很难排查，AI 能很快指出是类型不匹配还是缺少 Context 注入。

#### 2. 非 AI 策略：碎片化记录

- **Obsidian / Notion 截图流:**
  - 每解决一个报错，或者每成功实现一个状态跳转（比如从 Idle 成功跳到 Move），**立刻截图**。
  - 用一句话记下刚才的问题（例如：“坑点：StateTree 的 Enter State 条件如果是 false，它不会自动跳出，而是会卡住，必须配置 Transition”）。
  - 这些碎片是知乎文章中最有价值的“避坑指南”。

------

### 第三阶段：博客总结与输出 (Blogging Phase)

知乎的技术区读者喜欢：**硬核干货 + 清晰的逻辑 + 真实的踩坑经历**。

#### 1. AI 辅助策略：文章结构化与润色

- **大纲生成:**
  - **Prompt:** “我完成了一个将 Lyra Bot AI 迁移到 StateTree 的 Demo。请帮我构思一篇知乎技术文章的大纲。核心亮点包括：StateTree 数据绑定的难点、C++ Task 的重写、性能对比。语气要专业但幽默，适合 UE5 开发者阅读。”
- **复杂概念可视化描述:**
  - 如果你不擅长画图，可以描述逻辑给 AI，让它生成 mermaid 流程图代码，然后你再根据这个画出精美的图表。
  - **Prompt:** “帮我生成一个 Mermaid 流程图代码，对比 Behavior Tree 的‘每帧轮询’机制和 StateTree 的‘事件驱动/状态转换’机制。”

#### 2. 非 AI 策略：素材准备

- **对比 GIF:**

  文章里一定要有动图。录制两个并排的画面：

  - 左边：原始 Lyra Bot 的行为。
  - 右边：你用 StateTree 复刻的行为。
  - **价值:** 证明你确实还原了逻辑，这是建立信任感的关键。

- **代码块高亮:**

  在知乎贴代码时，重点标注出 `Context` 定义的部分和 `Transition` 的判断逻辑，这是读者最关心的。

------

### 建议的知乎文章结构 (模板)

为了提高写作效率，你可以直接套用这个结构，填空即可：

1. **背景 (The Why):** 为什么要做这个？（StateTree 是未来，Lyra 是标准，两者的结合是最佳学习路径）。
2. **核心差异 (The Concept):** 用一张图解释 BT 和 ST 在思维上的区别（树形选择 vs 状态机转换）。
3. **实战复刻 (The How - 核心部分):**
   - **数据流 (Context):** 怎么把 PlayerState、Controller 传进去？（这是最难的）。
   - **移动逻辑:** `MoveTo` 节点在 ST 中怎么配？
   - **战斗逻辑:** 怎么实现“看见人就开枪”的打断机制？
4. **踩坑记录 (The Pitfalls):** 列出3个你卡得最久的地方（例如：Linked State 怎么用，Parameters 没传递导致崩溃等）。
5. **性能对比/总结:** 如果可能，简单对比一下两者的 CPU 开销（Stat AI），或者谈谈开发手感的区别。





