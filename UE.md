# UE 学习记录

## Lyra

### Lyra Gameplay 框架总览

1. _EnhancedInput_ 框架：InputStack、InputMappingContext（优先级） TODO: InputAction 回调触发机制
2. 引擎运行流程 [UE5 -- 引擎运行流程（从 main 到 BeginPlay）](https://zhuanlan.zhihu.com/p/577433224)
3. Subsystems [《InsideUE4》GamePlay 架构（十一）Subsystems](https://zhuanlan.zhihu.com/p/158717151)
4. Lyra Gameplay 框架总览 [UE5 新项目 Gameplay 框架设计（以 Lyra 为例）](https://zhuanlan.zhihu.com/p/614718286)

### GameFeatures 架构

[https://zhuanlan.zhihu.com/p/467236675]

1. Lyra 中的 GameFeature:
   GameMode -启动-> Experience -封装-> GameFeatureAction..(以及 PawnData)

### UE 栈状态机的 UI 管理

[https://zhuanlan.zhihu.com/p/143882791]

1. UI 框架思考(MVVM) ()[https://www.kimi.com/chat/d320k9af7cam31pf005g]

 [LyraUI框架.md](Ready\面试准备.md#UI框架（CommonUI）) 

 [MVVM与MVC.md](Ready\面试准备.md#UI框架（MVVM的实践）) 

### Lyra CommonUI 框架

[https://zhuanlan.zhihu.com/p/700905273]

### Lyra 示例项目解读

[https://zhuanlan.zhihu.com/p/518029029]

1. Lyra ModularGameplay 初始化状态机 [https://www.kimi.com/chat/d396dmr12h64h658i0f0]



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

## 动画

[[UnrealCircle深圳\]《黑神话：悟空》的Motion Matching | 游戏科学 招文勇_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1GK4y1S7Zw/)

<img src="D:\Projects\NOTES\images\image-20251116115328799.png" alt="image-20251116115328799" style="zoom: 33%;" />



## UI

### 点击流程

![picture 0](D:\Projects\NOTES\images\IMG_20251115-153540054.png)  

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
