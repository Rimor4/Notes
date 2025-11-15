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

## UI


## 调试

[](https://www.bilibili.com/video/BV1iQ4y1j73A/)
[](https://www.bilibili.com/video/BV1st46z6ECv/)
