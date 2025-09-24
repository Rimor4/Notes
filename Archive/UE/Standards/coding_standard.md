> Write maintainable code by adhering to established standards and best practices.

At Epic Games, we have a few simple coding standards and conventions. This document reflects the state of Epic Games' current coding standards. Following the coding standards is mandatory.  
在 Epic Games，我们有一些简单的编码标准和约定。本文档反映了 Epic Games 当前编码标准的状态。遵循编码标准是强制性的。

Code conventions are important to programmers for several reasons:  
代码约定对程序员很重要，原因如下：

- 80% of the lifetime cost of a piece of software goes to maintenance.  
  软件生命周期成本的 80% 用于维护。
- Hardly any software is maintained for its whole life by the original author.  
  几乎没有任何软件是由原作者终生维护的。
- Code conventions improve the readability of software, allowing engineers to understand new code quickly and thoroughly.  
  代码约定提高了软件的可读性，使工程师能够快速、彻底地理解新代码。
- If we decide to expose source code to mod community developers, we want it to be easily understood.  
  如果我们决定向 mod 社区开发人员公开源代码，我们希望它易于理解。
- Many of these conventions are required for cross-compiler compatibility.  
  其中许多约定是交叉编译器兼容性所必需的。

The coding standards below are C++-centric; however, the standard is expected to be followed no matter which language is used. A section may provide equivalent rules or exceptions for specific languages where it's applicable.  
以下编码标准以 C++ 为中心；然而，无论使用哪种语言，都应该遵循该标准。某个部分可能会在适用的情况下为特定语言提供等效规则或例外情况。

## Class Organization   班级组织

**Classes** should be organized with the reader in mind rather than the writer. Since most readers will use the public interface of the class, the public implementation should be declared first, followed by the class's private implementation.  
**课程的**组织应该考虑到读者而不是作者。由于大多数读者将使用类的公共接口，因此应首先声明公共实现，然后声明类的私有实现。

```
	UCLASS()

	class EXAMPLEPROJECT_API AExampleActor : public AActor
	{
	    GENERATED_BODY()

	public:
	    // Sets default values for this actor's properties
	    AExampleActor();

	protected:

	    // Called when the game starts or when spawned
	    virtual void BeginPlay() override;
	};

```

## Copyright Notice   版权声明

Any source file (`.h`, `.cpp`, `.xaml`) provided by Epic Games for public distribution must contain a copyright notice as the first line in the file. The format of the notice must exactly match that shown below:  
Epic Games 提供的用于公开分发的任何源文件（ `.h` 、 `.cpp` 、 `.xaml` ）都必须包含版权声明作为文件的第一行。通知的格式必须与以下所示完全一致：

If this line is missing or not formatted properly, CIS will generate an error and fail.  
如果此行丢失或格式不正确，CIS 将生成错误并失败。

## Naming Conventions   命名约定

When using Naming Conventions, all code and comments should use U.S. English spelling and grammar.  
使用命名约定时，所有代码和注释都应使用美国英语拼写和语法。

- The first letter of each word in a name (such as type name or variable name) is capitalized. There is usually no underscore between words. For example, `Health` and `UPrimitiveComponent` are correct, but `lastMouseCoordinates` or `delta_coordinates` are not.  
  名称（例如类型名称或变量名称）中每个单词的首字母大写。单词之间通常没有下划线。例如， `Health`和`UPrimitiveComponent`是正确的，但`lastMouseCoordinates`或`delta_coordinates`则不正确。

This is PascalCase formatting for users who may be familiar with other object oriented programming languages  
这是 PascalCase 格式，适合熟悉其他面向对象编程语言的用户

- Type names are prefixed with an additional upper-case letter to distinguish them from variable names. For example, `FSkin` is a type name, and `Skin` is an instance of type `FSkin`.  
  类型名称带有一个额外的大写字母作为前缀，以将其与变量名称区分开。例如， `FSkin`是类型名称，而`Skin`是类型`FSkin`的实例。
- Template classes are prefixed by T.  
  模板类以 T 为前缀。

```
	template <typename ObjectType>
	class TAttribute

```

- Classes that inherit from [UObject](https://dev.epicgames.com/documentation/en-us/unreal-engine/objects-in-unreal-engine?application_version=5.4) are prefixed by U.  
  从 [UObject](https://dev.epicgames.com/documentation/en-us/unreal-engine/objects-in-unreal-engine?application_version=5.4) 继承的类以 U 为前缀。

```
class UActorComponent


```

- Classes that inherit from [AActor](https://dev.epicgames.com/documentation/en-us/unreal-engine/actors-in-unreal-engine?application_version=5.4) are prefixed by A.  
  继承自 [AActor](https://dev.epicgames.com/documentation/en-us/unreal-engine/actors-in-unreal-engine?application_version=5.4) 的类以 A 为前缀。

```
class AActor


```

- Classes that inherit from [SWidget](https://dev.epicgames.com/documentation/en-us/unreal-engine/slate-user-interface-programming-framework-for-unreal-engine?application_version=5.4) are prefixed by S.  
  从 [SWidget](https://dev.epicgames.com/documentation/en-us/unreal-engine/slate-user-interface-programming-framework-for-unreal-engine?application_version=5.4) 继承的类以 S 为前缀。

```
class SCompoundWidget


```

- Classes that are abstract interfaces are prefixed by I.  
  作为抽象接口的类以 I 为前缀。

```
class IAnalyticsProvider


```

- Epic's concept-alike struct types are prefixed by C.  
  Epic 的类似概念的结构类型以 C 为前缀。

```
	struct CStaticClassProvider
	{
		template <typename T>
		auto Requires(UClass*& ClassRef) -> decltype(
			ClassRef = T::StaticClass()
		);
	};

```

- Enums are prefixed by E.  
  枚举以 E 为前缀。

```
	enum class EColorBits
	{
	    ECB_Red,
	    ECB_Green,
	    ECB_Blue
	};

```

- Boolean variables must be prefixed by b.  
  布尔变量必须以 b 为前缀。

```
	bPendingDestruction
	bHasFadedIn

```

- Most other classes are prefixed by F, though some subsystems use other letters.  
  大多数其他类都以 F 为前缀，但某些子系统使用其他字母。
- Typedefs should be prefixed by whatever is appropriate for that type, such as:  
  Typedef 应该以适合该类型的任何内容作为前缀，例如：

  - F for typedef of a struct  
    F 表示结构体的 typedef
  - U for typedef of a `UObject`  
    U 代表`UObject`的 typedef

- A typedef of a particular template instantiation is no longer a template and should be prefixed accordingly.  
  特定模板实例化的 typedef 不再是模板，应添加相应的前缀。

```
typedef TArray<FMytype> FArrayOfMyTypes;


```

- Prefixes are omitted in C#.  
  C# 中省略了前缀。
- Unreal Header Tool requires the correct prefixes in most cases, so it's important to provide them.  
  在大多数情况下，虚幻标头工具需要正确的前缀，因此提供它们非常重要。
- Type template parameters and nested type aliases based on those template parameters are not subject to the above prefix rules, as the type category is unknown.  
  类型模板参数和基于这些模板参数的嵌套类型别名不受上述前缀规则的约束，因为类型类别未知。
- Prefer a Type suffix after a descriptive term.  
  最好在描述性术语后加上类型后缀。
- Disambiguate template parameters from aliases by using an In prefix:  
  使用 In 前缀消除模板参数与别名的歧义：

```
	template <typename InElementType>
	class TContainer
	{
	public:
	    using ElementType = InElementType;
	};

```

- Type and variable names are nouns.  
  类型和变量名称是名词。
- Method names are verbs that either describe the method's effect, or the return value of a method without an effect.  
  方法名称是动词，要么描述方法的效果，要么描述没有效果的方法的返回值。
- Macro names should be fully capitalized with words separated by underscores, and prefixed with `UE_`.  
  宏名称应完全大写，单词之间用下划线分隔，并以`UE_`为前缀。

```
#define UE_AUDIT_SPRITER_IMPORT


```

Variable, method, and class names should be:  
变量、方法和类名称应该是：

- Clear   清除
- Unambiguous   明确
- Descriptive   描述性的

The greater the scope of the name, the greater the importance of a good, descriptive name. Avoid over-abbreviation.  
名称的范围越大，一个好的描述性名称就越重要。避免过度缩写。

All variables should be declared on their own line so that you can provide comment on the meaning of each variable.  
所有变量都应该在自己的行上声明，以便您可以对每个变量的含义提供注释。

The JavaDocs style requires it.  
JavaDocs 风格需要它。

You can use multi-line or single-line comments before a variable Blank lines are optional for grouping variables.  
您可以在变量之前使用多行或单行注释。对于对变量进行分组，空行是可选的。

All functions that return a bool should ask a true/false question, such as `IsVisible()` or `ShouldClearBuffer()`.  
所有返回 bool 的函数都应该询问 true/false 问题，例如`IsVisible()`或`ShouldClearBuffer()` 。

A procedure (a function with no return value) should use a strong verb followed by an Object. An exception is, if the Object of the method is the Object it is in. In this case, the Object is understood from context. Names to avoid include those beginning with "Handle" and "Process" because the verbs are ambiguous.  
过程（没有返回值的函数）应该使用强动词，后跟对象。一个例外是，如果方法的对象是它所在的对象。在这种情况下，可以从上下文中理解该对象。要避免的名称包括以 “Handle” 和“Process”开头的名称，因为这些动词是不明确的。

We encourage you to prefix function parameter names with "Out" if:  
如果出现以下情况，我们鼓励您在函数参数名称前添加 “Out” 前缀：

- The function parameters are passed by reference.  
  函数参数通过引用传递。
- The function is expected to write to that value.  
  该函数预计会写入该值。

This makes it obvious that the value passed in this argument is replaced by the function.  
这很明显表明该参数中传递的值已被函数替换。

If an In or Out parameter is also a boolean, put "b" before the In/Out prefix, such as `bOutResult`.  
如果 In 或 Out 参数也是布尔值，请将 “b” 放在 In/Out 前缀之前，例如`bOutResult` 。

Functions that return a value should describe the return value. The name should make clear what value the function returns. This is particularly important for boolean functions. Consider the following two example methods:  
返回值的函数应该描述返回值。名称应该清楚地表明函数返回什么值。这对于布尔函数尤其重要。考虑以下两个示例方法：

```
	// what does true mean?
	bool CheckTea(FTea Tea);

	// name makes it clear true means tea is fresh
	bool IsTeaFresh(FTea Tea);

	float TeaWeight;
	int32 TeaCount;
	bool bDoesTeaStink;
	FName TeaName;
	FString TeaFriendlyName;
	UClass* TeaClass;
	USoundCue* TeaSound;
	UTexture* TeaTexture;

```

## Inclusive Word Choice   包容性的词语选择

When you work in the Unreal Engine codebase, we encourage you to strive to use respectful, inclusive, and professional language.  
当您在虚幻引擎代码库中工作时，我们鼓励您努力使用尊重、包容和专业的语言。

Word choice applies when you:  
词语选择适用于以下情况：

- name classes.   命名类。
- functions.   功能。
- data structures.   数据结构。
- types.   类型。
- variables.   变量。
- files and folders.   文件和文件夹。
- plugins.   插件。

It applies when you write snippets of user-facing text for the UI, error messages, and notifications. It also applies when writing about code, such as in comments and changelist descriptions.  
当您为 UI、错误消息和通知编写面向用户的文本片段时，它适用。它也适用于编写代码，例如注释和更改列表描述。

The following sections provide guidance and suggestions to help you choose words and names that are respectful and appropriate for all situations and audiences, and be a more effective communicator.  
以下部分提供指导和建议，帮助您选择尊重并适合所有情况和受众的词语和名称，并成为更有效的沟通者。

### Racial, ethnic, and religious inclusiveness

种族、民族和宗教包容性

- Do not use metaphors or similes that reinforce stereotypes. Examples include contrast black and white or _blacklist_ and _whitelist_.  
  不要使用强化刻板印象的隐喻或明喻。示例包括黑白对比或*黑名单*和*白名单*。
- Do not use words that refer to historical trauma or lived experience of discrimination. Examples include _slave_, _master_, and _nuke_.  
  不要使用涉及历史创伤或歧视经历的词语。示例包括 _Slave_ 、 _Master_ 和 _nuke_ 。

### Gender inclusiveness   性别包容性

- Refer to hypothetical people as _they_, _them_, and _their_, even in the singular.  
  将假设的人称为 _“他们”_ 、 _“他们_” 和 _“他们的”_ ，即使是单数。
- Refer to anything that is not a person as _it_ and _its_. For example, a module, plugin, function, client, server, or any other software or hardware component.  
  将任何不是人的事物称为 _“it”_ 和 _“its”_ 。例如，模块、插件、功能、客户端、服务器或任何其他软件或硬件组件。
- Do not assign a gender to anything that doesn't have one.  
  不要为没有性别的事物指定性别。
- Do not use collective nouns like _guys_ that assume gender.  
  不要使用集体名词，例如具有性别的*家伙*。
- Avoid colloquial phrases that contain arbitrary genders, like "a poor _man_'s X".  
  避免使用包含任意性别的口语短语，例如 “a Poor _man_ 's X”。

### Slang   俚语

- Remember that your words are being read by a global audience that may not share the same idioms and attitudes, and who might not understand the same cultural references.  
  请记住，您的文字正在被全球读者阅读，他们可能不具有相同的习语和态度，并且可能不理解相同的文化参考。
- Avoid slang and colloquialisms, even if you think they are funny or harmless. These may be hard to understand for people whose first language is not English, and might not translate well.  
  避免使用俚语和口语，即使您认为它们很有趣或无害。对于母语不是英语的人来说，这些可能很难理解，并且可能翻译得不好。
- Do not use profanity.   请勿使用脏话。

### Overloaded Words   超载的词语

- Many terms that we use for their technical meanings also have other meanings outside of technology. Examples include _abort_, _execute_, or _native_. When you use words like these, always be precise and examine the context in which they appear.  
  我们用于技术含义的许多术语还具有技术之外的其他含义。示例包括 _abort_ 、 _execute_ 或 _native_ 。当您使用此类词语时，请始终保持精确并检查它们出现的上下文。

### Word List   单词表

The following list identifies some terminology that we have used in the Unreal codebase in the past, but that we believe should be replaced with better alternatives:  
以下列表列出了我们过去在虚幻代码库中使用过的一些术语，但我们认为应该用更好的替代方案来替换它们：

<table data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><thead data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><tr data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><th data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1">Word Name&nbsp;&nbsp;字名</th><th data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1">Alternative Word Name&nbsp;&nbsp;替代词名</th></tr></thead><tbody data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><tr data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1">Blacklist&nbsp;&nbsp;黑名单</td><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1"><code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">_deny list_</code>, <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">_block list_</code>,<code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"> _exclude list_</code>, <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">_avoid list_</code>,<code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"> _unapproved list_</code>,<code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"> _forbidden list_</code>,<code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">_permission list_</code><br><code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">_deny list_</code> 、 <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">_block list_</code> 、 <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">_exclude list_</code> 、 <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">_avoid list_</code> 、 <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">_unapproved list_</code> 、 <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">_forbidden list_</code> 、 <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">_permission list_</code></td></tr><tr data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1">Whitelist&nbsp;&nbsp;白名单</td><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1">allow list_, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">include list</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">trust list</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">safe list</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">prefer list</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">approved list</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">permission list</em><br>允许列表_、<em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">包含列表</em>、<em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">信任列表</em>、<em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">安全列表</em>、<em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">偏好列表</em>、<em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">批准列表</em>、<em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">权限列表</em></td></tr><tr data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1">Master&nbsp;&nbsp;掌握</td><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1"><em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">primary</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">source</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">controller</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">template</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">reference</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">main</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">leader</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">original</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">base</em><br><em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">主要</em>,<em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"> 源</em>,<em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"> 控制器</em>,<em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"> 模板</em>,<em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"> 参考</em>,<em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"> 主要</em>,<em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"> 领导者</em>,<em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"> 原始</em>,<em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"> 基础</em></td></tr><tr data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1">Slave&nbsp;&nbsp;奴隶</td><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1"><em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">secondary</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">replica</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">agent</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">follower</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">worker</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">cluster node</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">locked</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">linked</em>, <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">synchronized</em><br><em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">secondary</em> 、 <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">replica</em> 、 <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">agent</em> 、 <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">follower</em> 、 <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">worker</em> 、<em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">集群节点</em>、 <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">locked</em> 、 <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">linked</em> 、 <em data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">synchronized</em></td></tr></tbody></table>

We are actively working to bring our code in line with the principles laid out above.  
我们正在积极努力使我们的代码符合上述原则。

## Portable C++ code   可移植的 C++ 代码

The `int` and unsigned `int` types vary in size across platforms. They are guaranteed to be at least 32 bits in width and are acceptable in code when the integer width is unimportant. Explicitly-sized types are used in serialized or replicated formats.  
`int`和 unsigned `int`类型的大小在不同平台上有所不同。它们保证宽度至少为 32 位，并且当整数宽度不重要时，它们在代码中是可以接受的。显式大小的类型用于序列化或复制格式。

Below is a list of common types:  
以下是常见类型的列表：

- `bool` for boolean values (NEVER assume the size of bool). `BOOL` will not compile.  
  `bool`表示布尔值（切勿假设 bool 的大小）。 `BOOL`无法编译。
- `TCHAR` for a character (NEVER assume the size of TCHAR).  
  `TCHAR`表示字符（切勿假设 TCHAR 的大小）。
- `uint8` for unsigned bytes (1 byte).  
  `uint8`表示无符号字节（1 个字节）。
- `int8` for signed bytes (1 byte).  
  `int8`表示有符号字节（1 个字节）。
- `uint16` for unsigned shorts (2 bytes).  
  `uint16`表示无符号短整型（2 个字节）。
- `int16` for signed shorts (2 bytes).  
  `int16`表示有符号的 Shorts（2 个字节）。
- `uint32` for unsigned ints (4 bytes).  
  `uint32`表示无符号整数（4 个字节）。
- `int32` for signed ints (4 bytes).  
  `int32`用于有符号整数（4 字节）。
- `uint64` for unsigned quad words (8 bytes).  
  `uint64`表示无符号四字（8 字节）。
- `int64` for signed quad words (8 bytes).  
  `int64`用于有符号四字（8 字节）。
- `float` for single precision floating point (4 bytes).  
  `float`表示单精度浮点数（4 字节）。
- `double` for double precision floating point (8 bytes).  
  `double`表示双精度浮点数（8 字节）。
- `PTRINT` for an integer that may hold a pointer (NEVER assume the size of PTRINT).  
  `PTRINT`表示可以保存指针的整数（永远不要假定 PTRINT 的大小）。

Use of standard libraries  
标准库的使用

---

Historically, UE has avoided direct use of the C and C++ standard libraries for the following reasons:  
从历史上看，UE 避免直接使用 C 和 C++ 标准库，原因如下：

- Replace slow implementations with our own that provide additional control over memory allocation.  
  用我们自己的实现替换缓慢的实现，以提供对内存分配的额外控制。
- Add new functionality before it's widely available, such as:  
  在广泛使用之前添加新功能，例如：

  - Making desirable, but non-standard, behavioral changes.  
    做出理想但非标准的行为改变。
  - Having consistent syntax across the codebase.  
    整个代码库具有一致的语法。
  - Avoiding constructs which are incompatible with UE's idioms.  
    避免与 UE 习惯用法不兼容的构造。

However, the standard library has matured and includes functionality that we don't want to wrap with an abstraction layer or reimplement ourselves.  
然而，标准库已经成熟，并且包含我们不想用抽象层包装或自己重新实现的功能。

When there is a choice between a standard library feature instead of our own, you should prefer the option that gives superior results. It's also important to remember that consistency is valued. If a legacy UE implementation is no longer serving a purpose, we may choose to deprecate it and migrate all usage toward the standard library.  
当在标准库功能而不是我们自己的功能之间进行选择时，您应该更喜欢能够提供卓越结果的选项。同样重要的是要记住，一致性很重要。如果旧版 UE 实现不再发挥作用，我们可能会选择弃用它，并将所有使用迁移到标准库。

Avoid mixing UE idioms and standard library idioms in the same API. The following table lists common idioms along with recommendations on when to use them.  
避免在同一个 API 中混合 UE 惯用法和标准库惯用法。下表列出了常见习语以及有关何时使用它们的建议。

<table data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><thead data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><tr data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><th data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1">Idiom&nbsp;&nbsp;成语</th><th data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1">Description&nbsp;&nbsp;描述</th></tr></thead><tbody data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><tr data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1"><code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">&lt;atomic&gt;</code></td><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1">The atomic idiom should be used in new code and old migrated when touched. Atomics are expected to be implemented fully and efficiently on all supported platforms. Our own <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">TAtomic</code> is only partially implemented, and it isn't in our interest to maintain and improve it.<br>原子习惯用法应该在新代码中使用，旧代码在触及时迁移。预计原子将在所有支持的平台上全面有效地实施。我们自己的<code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">TAtomic</code>仅实现了部分，维护和改进它不符合我们的利益。</td></tr><tr data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1"><code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">&lt;type_traits&gt;</code></td><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1">The type traits idiom should be used where there's overlap between a legacy UE trait and a standard trait. Traits are often implemented as compiler intrinsics for correctness, and compilers can have knowledge of the standard traits and select faster compilation paths instead of treating them as plain C++. One concern is that our traits typically have an upper case <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">Value</code> static or <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">Type</code> typedef, whereas standard traits are expected to use <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">value</code> and <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">type</code>. This is an important distinction, as a particular syntax is expected by compositional traits, for example <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">std::conjunction</code>. New traits we add should be written with lowercase <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">value</code> or <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">type</code> to support composition. Existing traits should be updated to support either case.<br>当遗留 UE 特征和标准特征之间存在重叠时，应使用类型特征习惯用法。为了保证正确性，特征通常被实现为编译器内在函数，编译器可以了解标准特征并选择更快的编译路径，而不是将它们视为普通的 C++。一个问题是我们的特征通常具有大写的<code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">Value</code> static 或<code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">Type</code> typedef，而标准特征预计使用<code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">value</code>和<code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">type</code> 。这是一个重要的区别，因为组合特征需要特定的语法，例如<code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">std::conjunction</code> 。我们添加的新特征应该用小写<code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">value</code>或<code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">type</code>编写以支持组合。应更新现有特征以支持这两种情况。</td></tr><tr data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1"><code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">&lt;initializer_list&gt;</code></td><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1">The initializer list idiom must be used to support braced initializer syntax. This is a case where the language and the standard libraries overlap. There is no alternative if you want to support it.<br>必须使用初始化列表惯用法来支持花括号初始化语法。这是语言和标准库重叠的情况。如果你想支持它，别无选择。</td></tr><tr data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1"><code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">&lt;regex&gt;</code></td><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1">The regex idiom may be used directly, but its use should be encapsulated within editor-only code. We have no plans to implement our own regex solution.<br>正则表达式习惯用法可以直接使用，但其使用应封装在仅限编辑器的代码中。我们没有计划实施我们自己的正则表达式解决方案。</td></tr><tr data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1"><code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">&lt;limits&gt;</code></td><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1"><code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">std::numeric_limits</code> can be used in its entirety.<br><code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">std::numeric_limits</code>可以完整使用。</td></tr><tr data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1"><code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">&lt;cmath&gt;</code></td><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1">All floating point functions from this header may be used.<br>可以使用该头文件中的所有浮点函数。</td></tr><tr data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e"><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1"><code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">&lt;cstring&gt;</code>: <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">memcpy()</code> and <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">memset()</code><br><code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">&lt;cstring&gt;</code> : <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">memcpy()</code>和<code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">memset()</code></td><td data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e" data-immersive-translate-paragraph="1">These idioms may be used instead of <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">FMemory::Memcpy</code> and <code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">FMemory::Memset</code> respectively, when they have a demonstrable performance benefit.<br>当这些习惯用法具有明显的性能优势时，可以分别使用它们代替<code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">FMemory::Memcpy</code>和<code data-immersive-translate-walked="9e4703ce-562e-4792-8be6-7d796b876e5e">FMemory::Memset</code> 。</td></tr></tbody></table>

Standard containers and strings should be avoided except in interop code.  
除互操作代码外，应避免使用标准容器和字符串。

Comments are communication and communication is vital. The following sections detail some important things to keep in mind about comments (from Kernighan & Pike _The Practice of Programming_).  
评论就是沟通，沟通至关重要。以下部分详细介绍了有关注释时需要记住的一些重要事项（来自 Kernighan 和 Pike _编程实践_）。

### Guidelines   指南

- Write self-documenting code. For example:  
  编写自记录代码。例如：

```
	// Bad:
	t = s + l - b;

	// Good:
	TotalLeaves = SmallLeaves + LargeLeaves - SmallAndLargeLeaves;

```

- Write useful comments. For example:  
  写有用的评论。例如：

```
	// Bad:
	// increment Leaves
	++Leaves;

	// Good:
	// we know there is another tea leaf
	++Leaves;

```

- Do not over comment bad code — rewrite it instead. For example:  
  不要过度评论不好的代码——而是重写它。例如：

```
	// Bad:
	// total number of leaves is sum of
	// small and large leaves less the
	// number of leaves that are both
	t = s + l - b;

	// Good:
	TotalLeaves = SmallLeaves + LargeLeaves - SmallAndLargeLeaves;

```

- Do not contradict the code. For example:  
  不要与代码相矛盾。例如：

```
	// Bad:
	// never increment Leaves!
	++Leaves;

	// Good:
	// we know there is another tea leaf
	++Leaves;

```

### Const Correctness

Const is documentation as much as it is a compiler directive. All code should strive to be const-correct. This includes the following guidelines:

- Pass function arguments by const pointer or reference if those arguments are not intended to be modified by the function.
- Flag methods as const if they do not modify the object.
- Use const iteration over containers if the loop isn't intended to modify the container.

Const Example:

```
void SomeMutatingOperation(FThing& OutResult, const TArray<Int32>& InArray)
{
    // InArray will not be modified here, but OutResult probably will be
}

void FThing::SomeNonMutatingOperation() const
{
    // This code will not modify the FThing it is invoked on
}

TArray<FString> StringArray;
for (const FString& : StringArray)
{
    // The body of this loop will not modify StringArray
}


```

Const is also preferred for by-value function parameters and locals. This tells the reader that the variable will not be modified in the body of the function, which makes it easier to understand. If you do this, make sure that the declaration and the definition match, as this can affect the JavaDoc process.

```
void AddSomeThings(const int32 Count);

void AddSomeThings(const int32 Count)
{
    const int32 CountPlusOne = Count + 1;
    // Neither Count nor CountPlusOne can be changed during the body of the function
}


```

One exception to this is pass-by-value parameters, which are moved into a container. For more information, see the "Move semantics" section on this page.

Example:

```
void FBlah::SetMemberArray(TArray<FString> InNewArray)
{
    MemberArray = MoveTemp(InNewArray);
}


```

Put the const keyword on the end when making a pointer itself const (rather than what it points to). References can't be"reassigned"anyway, and so can't be made const in the same way.

Example:

```
// Const pointer to non-const object - pointer cannot be reassigned, but T can still be modified
T* const Ptr = ...;

// Illegal
T& const Ref = ...;


```

Never use const on a return type. This inhibits move semantics for complex types, and will give compile warnings for built-in types. This rule only applies to the return type itself, not the target type of a pointer or reference being returned.

Example:

```
// Bad - returning a const array
const TArray<FString> GetSomeArray();

// Fine - returning a reference to a const array
const TArray<FString>& GetSomeArray();

// Fine - returning a pointer to a const array
const TArray<FString>* GetSomeArray();

// Bad - returning a const pointer to a const array
const TArray<FString>* const GetSomeArray();


```

### Example Formatting

We use a system based on JavaDoc to extract comments from the code and build documentation automatically, therefore we recommend specific comment formatting rules.

The following example demonstrates the format of **class**, **method**, and **variable** comments. Remember that comments should augment the code. Code documents the implementation while comments document the intent. Make sure to update comments when you change the intent of a piece of code.

Note that two different parameter comment styles are supported, shown by the `Steep` and `Sweeten` methods. The `@param` style used by `Steep` is the traditional multi-line style. For simple functions, it can be clearer to integrate the parameter and return value documentation into the descriptive comment for the function. This is demonstrated in the `Sweeten` example. Special comment tags like `@see` or `@return` should only be used to start new lines following the primary description.

Method comments should only be included once: where the method is publicly declared. The method comments should only contain information relevant to callers of the method, including any information about overrides of the method that may be relevant to the caller. Details about the implementation of the method and its overrides, that are not relevant to callers, should be commented within the method implementation.

Class comments should include:

- A description of the problem this class solves.
- The reason why was this class created.

Multi-line method comments should include:

- **Function purpose**: Documents the problem this function solves. As previously stated, comments document intent, and code documents implementation.
- **Parameter comments**: Each parameter comment should include:

  - units of measure;
  - the range of expected values;
  - "impossible" values;
  - and the meaning of status/error codes.

- **Return comment**: Documents the expected return value, just as an output variable is documented. To avoid redundancy, an explicit `@return` comment should not be used if the sole purpose of the function is to return this value and that is already documented in the function purpose.
- **Extra information:** `@warning`, `@note`, `@see`, and `@deprecated` can optionally be used to document additional relevant information. Each should be declared on their own line following the rest of the comments.

## Modern C++ Language Syntax

Unreal Engine is built to be massively portable to many C++ compilers, so we are careful to use features that are compatible with the compilers we might be supporting. Sometimes, features are so useful that we will wrap them in macros and use them pervasively. However, we usually wait until all the compilers we support are up to the latest standard.

Unreal Engine compiles with a language version of C++20 by default and requires a minimum version of C++17 to build. We use many modern language features that are well-supported across modern compilers. In some cases, we wrap usage of these features in preprocessor conditionals. However, sometimes we decide to avoid certain language features entirely, for portability or other reasons.

Unless specified below, as a modern C++ compiler feature we are supporting, you should not use compiler-specific language features unless they are wrapped in preprocessor macros or conditionals and used sparingly.

### Static Assert

The `static_assert` keyword is valid for use where you need a compile-time assertion.

### Override and Final

The `override` and `final` keywords are valid for use, and their use is strongly encouraged. There might be many places where these have been omitted, but they will be fixed over time.

### Nullptr

You should use `nullptr` instead of the C-style `NULL` macro in all cases.

One exception to this is the use of `nullptr` in C++/CX builds (such as for Xbox One). In this case, the use of `nullptr` is actually the managed null reference type. It is mostly compatible with `nullptr` from native C++ except in its type and some template instantiation contexts, and so you should use the `TYPE_OF_NULLPTR` macro instead of the more usual `decltype(nullptr)` for compatibility.

### Auto

You shouldn't use `auto` in C++ code, except for the few exceptions listed below. Always be explicit about the type you're initializing. This means that the type must be plainly visible to the reader. This rule also applies to the use of the `var` keyword in C#.

C++17's structured binding feature should also not be used, as it is effectively a variadic `auto`.

Acceptable use of `auto`:

- When you need to bind a lambda to a variable, as lambda types are not expressible in code.
- For iterator variables, but only where the iterator's type is verbose and would impair readability.
- In template code, where the type of an expression cannot easily be discerned. This is an advanced case.

It's very important that types are clearly visible to someone who is reading the code. Even though some IDEs are able to infer the type, doing so relies on the code being in a compilable state. It also won't assist users of merge/diff tools, or when viewing individual source files in isolation, such as on GitHub.

If you're sure you are using `auto` in an acceptable way, always remember to correctly use `const`, `&`, or `*` just like you would with the type name. With `auto`, this will coerce the inferred type to be what you want.

### Range-Based For

This is preferred to keep the code easier to understand and more maintainable. When you migrate code that uses old `TMap` iterators, be aware that the old `Key()` and `Value()` functions, which were methods of the iterator type, are now simply `Key` and `Value` fields of the underlying key-value `TPair`.

Example:

```
TMap<FString, int32> MyMap;

// Old style
for (auto It = MyMap.CreateIterator(); It; ++It)
{
    UE_LOG(LogCategory, Log, TEXT("Key: %s, Value: %d"), It.Key(), *It.Value());
}

// New style
for (TPair<FString, int32>& Kvp : MyMap)
{
    UE_LOG(LogCategory, Log, TEXT("Key: %s, Value: %d"), *Kvp.Key, Kvp.Value);
}


```

We also have range replacements for some standalone iterator types.

Example:

```
// Old style
for (TFieldIterator<UProperty> PropertyIt(InStruct, EFieldIteratorFlags::IncludeSuper); PropertyIt; ++PropertyIt)
{
    UProperty* Property = *PropertyIt;
    UE_LOG(LogCategory, Log, TEXT("Property name: %s"), *Property->GetName());
}

// New style
for (UProperty* Property : TFieldRange<UProperty>(InStruct, EFieldIteratorFlags::IncludeSuper))
{
    UE_LOG(LogCategory, Log, TEXT("Property name: %s"), *Property->GetName());
}


```

### Lambdas and Anonymous Functions

Lambdas can be used freely but come with additional safety concerns. The best lambdas should be no more than a couple of statements in length, particularly when used as part of a larger expression or statement, for example as a predicate in a generic algorithm.

Example:

```
// Find first Thing whose name contains the word "Hello"
Thing* HelloThing = ArrayOfThings.FindByPredicate([](const Thing& Th){ return Th.GetName().Contains(TEXT("Hello")); });

// Sort array in reverse order of name
Algo::Sort(ArrayOfThings, [](const Thing& Lhs, const Thing& Rhs){ return Lhs.GetName() > Rhs.GetName(); });|


```

Be aware that stateful lambdas can't be assigned to function pointers, which we tend to use a lot. Non-trivial lambdas should be documented in the same manner as regular functions. Lambdas can also be used as [Delegates](https://dev.epicgames.com/documentation/en-us/unreal-engine/delegates-and-lamba-functions-in-unreal-engine?application_version=5.4) for deferred execution using functions like `BindWeakLambda` where captured variables function as a payload.

#### Captures and Return Types

Explicit captures should be used rather than automatic capture (`[&]` and `[=]`). This is important for readability, maintainability, safety, and performance reasons, particularly when used with large lambdas and deferred execution.

Explicit captures declare the author's intent; therefore, mistakes are caught during code review. Incorrect captures can cause serious bugs and crashes, which are more likely to become problematic as the code is maintained over time. Here are some additional things to keep in mind about lambda captures:

- By-reference capture and by-value capture of pointers (including the `this` pointer) can cause data corruption and crashes if the execution of the lambda is deferred. Local and member variables should never be captured by reference for deferred lambdas.
- By-value capture can be a performance concern if it makes unnecessary copies for a non-deferred lambda.
- Accidentally captured UObject pointers are invisible to the garbage collector. Automatic capture catches `this` implicitly if any member variables are referenced, even though `[=]` gives the impression of the lambda having its own copies of everything.
- Delegate wrappers like `CreateWeakLambda` and `CreateSPLambda` should be used for deferred execution as they will automatically unbind if the UObject or shared pointer are freed. Other shared objects can be captured as TWeakObjectPtr or TWeakPtr and then validated inside the lambda.
- Any deferred lambda use that does not follow these guidelines must have a comment explaining why the lambda capture is safe.

Explicit return types should be used for large lambdas or when you are returning the result of another function call. These should be considered in the same way as the `auto` keyword.

### Strongly-Typed Enums

Enumerated (Enum) classes are a replacement for old-style namespaced enums, both for regular enums and `UENUMs`. For example:

```
// Old enum
UENUM()
namespace EThing
{
    enum Type
    {
        Thing1,
        Thing2
    };
}

// New enum
UENUM()
enum class EThing : uint8
{
    Thing1,
    Thing2
}


```

Enums are supported as `UPROPERTYs`, and replace the old `TEnumAsByte<>` workaround. Enum properties can also be any size, not just bytes:

```
// Old property
UPROPERTY()
TEnumAsByte<EThing::Type> MyProperty;

// New property
UPROPERTY()
EThing MyProperty;


```

Enums exposed to Blueprints must continue to be based on `uint8`.

Enum classes used as flags can take advantage of the `ENUM_CLASS_FLAGS(EnumType)` macro to automatically define all of the bitwise operators:

```
enum class EFlags
{
    None = 0x00,
    Flag1 = 0x01,
    Flag2 = 0x02,
    Flag3 = 0x04
};

ENUM_CLASS_FLAGS(EFlags)


```

The one exception to this is the use of flags in a _truth_ context - this is a limitation of the language. Instead, all enum flags should have an enumerator called `None` which is set to 0 for comparisons:

```
// Old
if (Flags & EFlags::Flag1)

// New
if ((Flags & EFlags::Flag1) != EFlags::None)


```

### Move Semantics

All of the main container types — `TArray`, `TMap`, `TSet`, `FString` — have move constructors and move assignment operators. These are often used automatically when passing or returning these types by value. They can also be explicitly invoked by using `MoveTemp`, UE's equivalent of `std::move`.

Returning containers or strings by value can be beneficial for expressivity, without the usual cost of temporary copies. Rules around pass-by-value and use of `MoveTemp` are still being established, but can already be found in some optimized areas of the codebase.

### Default Member Initializers

Default member initializers can be used to define the defaults of a class inside the class itself:

```
UCLASS()
class UTeaOptions : public UObject
{
    GENERATED_BODY()

public:
    UPROPERTY()
    int32 MaximumNumberOfCupsPerDay = 10;

    UPROPERTY()
    float CupWidth = 11.5f;

    UPROPERTY()
    FString TeaType = TEXT("Earl Grey");

    UPROPERTY()
    EDrinkingStyle DrinkingStyle = EDrinkingStyle::PinkyExtended;
};


```

Code written like this has the following benefits:

- It doesn't need to duplicate initializers across multiple constructors.
- It isn't possible to mix the initialization order and declaration order.
- The member type, property flags, and default values are all in one place. This helps readability and maintainability.

However, there are also some downsides:

- Any change to the defaults requires a rebuild of all dependent files.
- Headers can't change in patch releases of the engine, so this style can limit the kinds of fixes that are possible.
- Some things can't be initialized in this way, such as base classes, `UObject` subobjects, pointers to forward-declared types, values deduced from constructor arguments, and members initialized over multiple steps.
- Putting some initializers in the header and the rest in constructors in the .cpp file, can reduce readability and maintainability.

Use your best judgment when deciding whether to use default member initializers. As a rule of thumb, default member initializers make more sense with in-game code than engine code. Consider using config files for default values.

## Third Party Code

Whenever you modify the code to a library that we use in the engine, be sure to tag your changes with a //@UE5 comment, as well as an explanation of why you made the change. This makes merging the changes into a new version of that library easier, and ensures licensees can easily find any modifications we have made.

Any third party code included in the engine should be marked with comments formatted to be easily searchable. For example:

```
// @third party code - BEGIN PhysX
#include <physx.h>
// @third party code - END PhysX
// @third party code - BEGIN MSDN SetThreadName
// [http://msdn.microsoft.com/en-us/library/xcb2z8hs.aspx]
// Used to set the thread name in the debugger
...
//@third party code - END MSDN SetThreadName


```

## Code Formatting

### Braces

Brace wars are foul. Epic Games has a long standing usage pattern of putting braces on a new line. Please adhere to this usage, regardless of the size of the function or block. For example:

```
// Bad
int32 GetSize() const { return Size; }

// Good
int32 GetSize() const
{
    return Size;
}


```

Always include braces in single-statement blocks. For example:

```
if (bThing)
{
    return;
}


```

### If - Else

Each block of execution in an if-else statement should be in braces. This helps prevent editing mistakes. When braces are not used, someone could unwittingly add another line to an if block. The extra line wouldn't be controlled by the if expression, which would be bad. It's also bad when conditionally compiled items cause if/else statements to break. So always use braces.

```
if (bHaveUnrealLicense)
{
    InsertYourGameHere();
}
else
{
    CallMarkRein();
}


```

A multi-way if statement should be indented with each `else if` indented the same amount as the first `if`; this makes the structure clear to a reader:

```
if (TannicAcid < 10)
{
    UE_LOG(LogCategory, Log, TEXT("Low Acid"));
}
else if (TannicAcid < 100)
{
    UE_LOG(LogCategory, Log, TEXT("Medium Acid"));
}
else
{
    UE_LOG(LogCategory, Log, TEXT("High Acid"));
}


```

### Tabs and Indenting

Below are some standards for indenting your code.

- Indent code by execution block.
- Use tabs for whitespace at the beginning of a line, not spaces. Set your tab size to 4 characters. Note, spaces are sometimes necessary and allowed for keeping code aligned regardless of the number of spaces in a tab. For example, when you are aligning code that follows non-tab characters.
- If you are writing code in C#, please also use tabs, not spaces. The reason for this is that programmers often switch between C# and C++, and most prefer to use a consistent setting for tabs. Visual Studio defaults to using spaces for C# files, so you need to remember to change this setting when working on Unreal Engine code.

### Switch Statements

Except for empty cases (multiple cases having identical code), switch case statements should explicitly label that a case falls through to the next case. Either include a break, or include a "falls through" comment in each case. Other code control-transfer commands (return, continue, and so on) are fine as well.

Always have a default case. Include a break just in case someone adds a new case after the default.

```
switch (condition)
{
    case 1:
        ...
        // falls through

    case 2:
        ...
        break;

    case 3:
        ...
        return;

    case 4:
    case 5:
        ...
        break;

    default:
        break;
}


```

## Namespaces

You can use namespaces to organize your classes, functions and variables where appropriate. If you do use them, follow the rules below.

- Most UE code is currently not wrapped in a global namespace.

  - Be careful to avoid collisions in the global scope, especially when using or including third party code.

- Namespaces are not supported by UnrealHeaderTool.

  - Namespaces should not be used when defining `UCLASSes`, `USTRUCTs` and so on.

- New APIs which aren't `UCLASSes`, `USTRUCTs` etc, should be placed in a `UE::` namespace, and ideally a nested namespace, e.g. `UE::Audio::`.

  - Namespaces which are used to hold implementation details which are not part of the public-facing API should go in a `Private` namespace, e.g. `UE::Audio::Private::`.

- `Using` declarations:

  - Do not put `using` declarations in the global scope, even in a `.cpp` file (it will cause problems with our "unity" build system.)

- It's okay to put `using` declarations within another namespace, or within a function body.
- If you put `using` declarations within a namespace, this will carry over to other occurrences of that namespace in the same translation unit. As long as you are consistent, it will be fine.
- You can only use `using` declarations in header files safely if you follow the above rules.
- Forward-declared types need to be declared within their respective namespace.

  - If you don't do this, you will get link errors.

- If you declare a lot of classes or types within a namespace, it can be difficult to use those types in other global-scoped classes (for example, function signatures will need to use explicit namespace when appearing in class declarations).
- You can use `using` declarations to only alias specific variables within a namespace into your scope.

  - For example, using `Foo::FBar`. However, we don't usually do that in Unreal code.

- Macros cannot live in a namespace.

  - They should be prefixed with `UE_` instead of living in a namespace, for example `UE_LOG`.

## Physical Dependencies

- File names should not be prefixed where possible.

  - For example, `Scene.cpp` instead of `UScene.cpp`. This makes it easy to use tools like Workspace Whiz or Visual Assist's Open File in Solution, by reducing the number of letters needed to identify the file you want.

- All headers should protect against multiple includes with the `#pragma once` directive.

  - Note that all compilers we use support `#pragma once`.

```
#pragma once
//<file contents>


```

- Try to minimize physical coupling.

  - In particular, avoid including standard library headers from other headers.

- Forward declarations are preferred to including headers.
- When including a header, be as fine grained as possible.

  - For example, do not include `Core.h`. Instead, you should include the specific headers in Core that you need definitions from.

- Try to include every header you need directly to make fine-grained inclusion easier.
- Don't rely on a header that is included indirectly by another header you include.
- Don't rely on anything being included through another header. Include everything you need.
- Modules have Private and Public source directories.

  - Any definitions that are needed by other modules must be in headers in the Public directory. Everything else should be in the Private directory. In older Unreal modules, these directories may still be called "Src" and "Inc", but those directories are meant to separate private and public code in the same way, and are not meant to separate header files from source files.

- Don't worry about setting up your headers for precompiled header generation.

  - UnrealBuildTool can do a better job of this than you can.

- Split large functions into logical sub-functions.

  - One area of compilers' optimizations is the elimination of common subexpressions. The larger your functions are, the more work the compiler has to do to identify them. This leads to greatly inflated build times.

- Don't use a large number of inline functions.

  - Inline functions force rebuilds even in files which don't use them. Inline functions should only be used for trivial accessors and when profiling shows there is a benefit to doing so.

- Be conservative in your use of `FORCEINLINE`.

  - All code and local variables will be expanded out into the calling function. This will cause the same build time problems as those caused by large functions.

## Encapsulation

Enforce encapsulation with the protection keywords. Class members should almost always be declared private unless they are part of the public/protected interface to the class. Use your best judgment, but always be aware that a lack of accessors makes it hard to refactor later without breaking plugins and existing projects.

If particular fields are only intended to be usable by derived classes, make them private and provide protected accessors.

Use final if your class is not designed to be derived from.

## General Style Issues

- Minimize dependency distance.

  - When code depends on a variable having a certain value, try to set that variable's value right before using it. Initializing a variable at the top of an execution block, and not using it for a hundred lines of code, gives lots of space for someone to accidentally change the value without realizing the dependency. Having it on the next line makes it clear why the variable is initialized the way it is and where it is used.

- Split methods into sub-methods where possible.

  - It is easier for someone to look at a big picture, and then drill down to the interesting details, than it is to start with the details and reconstruct the big picture from them. In the same way, it is easier to understand a simple method, that calls a sequence of several well-named sub-methods, than it is to understand an equivalent method that simply contains all the code in those sub-methods.

- In function declarations or function call sites, do not add a space between the function's name and the parentheses that precede the argument list.
- Address compiler warnings.

  - Compiler warning messages mean something is wrong. Fix what the compiler is warning you about. If you absolutely can't address it, use `#pragma` to suppress the warning, but this should only be done as a last resort.

- Leave a blank line at the end of the file.

  - All `.cpp` and `.h` files should include a blank line, to coordinate with gcc.

- Debug code should either be useful and polished, or not checked in.

  - Debug code that is intermixed with other code makes the other code harder to read.

- Always use the `TEXT()` macro around string literals.

  - Without the `TEXT()` macro, code that constructs `FStrings` from literals will cause an undesirable string conversion process.

- Avoid repeating the same operation redundantly in loops.

  - Move common subexpressions out of loops to avoid redundant calculations. Make use of statics in some cases, to avoid globally-redundant operations across function calls, such as constructing an `FName` from a string literal.

- Be mindful of hot reload.

  - Minimize dependencies to cut down on iteration time. Don't use inlining or templates for functions which are likely to change over a reload. Only use statics for things which are expected to remain constant over a reload.

- Use intermediate variables to simplify complicated expressions.

  - If you have a complicated expression, it can be easier to understand if you split it into sub-expressions, that are assigned to intermediate variables, with names describing the meaning of the sub-expression within the parent expression. For example:

```
if ((Blah->BlahP->WindowExists->Etc && Stuff) &&
    !(bPlayerExists && bGameStarted && bPlayerStillHasPawn &&
    IsTuesday())))
{
    DoSomething();
}


```

Should be replaced with:

```
const bool bIsLegalWindow = Blah->BlahP->WindowExists->Etc && Stuff;
const bool bIsPlayerDead = bPlayerExists && bGameStarted && bPlayerStillHasPawn && IsTuesday();
if (bIsLegalWindow && !bIsPlayerDead)
{
    DoSomething();
}


```

- Pointers and references should only have one space to the right of the pointer or reference.

  - This makes it easy to quickly use **Find in Files** for all pointers or references to a certain type. For example:

```
// Use this
FShaderType* Ptr

// Do not use these:
FShaderType *Ptr
FShaderType * Ptr


```

- Shadowed variables are not allowed.

  - C++ allows variables to be shadowed from an outer scope, but this makes usage ambiguous to a reader. For example, there are three usable `Count` variables in this member function:

```
class FSomeClass
{
public:
    void Func(const int32 Count)
    {
        for (int32 Count = 0; Count != 10; ++Count)
        {
            // Use Count
        }
    }

private:
    int32 Count;
}


```

- Avoid using anonymous literals in function calls.

  - Prefer named constants which describe their meaning. This makes intent more obvious to a casual reader as it avoids the need to look up the function declaration to understand it.

```
// Old style
Trigger(TEXT("Soldier"), 5, true);.

// New style
const FName ObjectName                = TEXT("Soldier");
const float CooldownInSeconds         = 5;
const bool bVulnerableDuringCooldown  = true;
Trigger(ObjectName, CooldownInSeconds, bVulnerableDuringCooldown);


```

- Avoid defining non-trivial static variables in headers.

  - Non-trivial static variables cause an instance to be compiled into in every translation unit that includes that header:

```
// SomeModule.h
static const FString GUsefulNamedString = TEXT("String");

// *Replace the above with:*

// SomeModule.h
extern SOMEMODULE_API const FString GUsefulNamedString;

// SomeModule.cpp
const FString GUsefulNamedString = TEXT("String");


```

- Avoid making extensive changes which do not change the code's behavior (for example: changing whitespace or mass renaming of private variables) as these cause unnecessary noise in source history and are disruptive when merging.

  - If such a change is important, for example fixing broken indentation caused by an automated merge tool, it should be submitted on its own and not mixed with behavioral changes.
  - Prefer to fix whitespace or other minor coding standard violations only when other edits are being made to the same lines or nearby code.

## API Design Guidelines

- Boolean function parameters should be avoided.

  - In particular, boolean parameters should be avoided for flags passed to functions. These have the same anonymous literal problem as mentioned previously, but they also tend to multiply over time as APIs get extended with more behavior. Instead, prefer an enum (see the advice on use of enums as flags in the [Strongly-Typed Enums](https://dev.epicgames.com/documentation/en-us/unreal-engine/epic-cplusplus-coding-standard-for-unreal-engine?application_version=5.4#strongly-typedenums) section):

```
// Old style
FCup* MakeCupOfTea(FTea* Tea, bool bAddSugar = false, bool bAddMilk = false, bool bAddHoney = false, bool bAddLemon = false);
FCup* Cup = MakeCupOfTea(Tea, false, true, true);

// New style
enum class ETeaFlags
{
    None,
    Milk  = 0x01,
    Sugar = 0x02,
    Honey = 0x04,
    Lemon = 0x08
};
ENUM_CLASS_FLAGS(ETeaFlags)

FCup* MakeCupOfTea(FTea* Tea, ETeaFlags Flags = ETeaFlags::None);
FCup* Cup = MakeCupOfTea(Tea, ETeaFlags::Milk | ETeaFlags::Honey);


```

- This form prevents the accidental transposing of flags, avoids accidental conversion from pointer and integer arguments, removes the need to repeat redundant defaults, and is more efficient.
- It is acceptable to use `bools` as arguments when they are the complete state to be passed to a function like a setter, such as `void FWidget::SetEnabled(bool bEnabled)`. Though consider refactoring if this changes.
- Avoid overly-long function parameter lists.

  - If a function takes many parameters then consider passing a dedicated struct instead:

```
// Old style
TUniquePtr<FCup[]> MakeTeaForParty(const FTeaFlags* TeaPreferences, uint32 NumCupsToMake, FKettle* Kettle, ETeaType TeaType = ETeaType::EnglishBreakfast, float BrewingTimeInSeconds = 120.0f);

// New style
struct FTeaPartyParams
{
    const FTeaFlags* TeaPreferences       = nullptr;
    uint32           NumCupsToMake        = 0;
    FKettle*         Kettle               = nullptr;
    ETeaType         TeaType              = ETeaType::EnglishBreakfast;
    float            BrewingTimeInSeconds = 120.0f;
};
TUniquePtr<FCup[]> MakeTeaForParty(const FTeaPartyParams& Params);


```

- Avoid overloading functions by `bool` and `FString`.

  - This can have unexpected behavior:

```
void Func(const FString& String);
void Func(bool bBool);

Func(TEXT("String")); // Calls the bool overload!


```

- Interface classes should always be abstract.

  - Interface classes are prefixed with "I" and must not have member variables. Interfaces are allowed to contain methods that are not pure-virtual, and can contain methods that are non-virtual or static, as long as they are implemented inline.

- Use the `virtual` and `override` keywords when declaring an overriding method.

When declaring a virtual function in a derived class that overrides a virtual function in the parent class, you must use both the `virtual` and the `override` keywords. For example:

```
class A
{
public:
    virtual void F() {}
};

class B : public A
{
public:
    virtual void F() override;
}


```

There is a lot of existing code that doesn't follow this yet, due to the recent addition of the `override` keyword. The `override` keyword should be added to that code when convenient.

- UObjects should be passed around by pointer, not reference. If null is not expected by a function, this should be documented by the API or handled appropriately. For example:

```
// Bad
void AddActorToList(AActor& Obj);

// Good
void AddActorToList(AActor* Obj);


```

## Platform-Specific Code

Platform-specific code should always be abstracted and implemented in platform-specific source files in appropriately named subdirectories, for example:

```
Engine/Platforms/[PLATFORM]/Source/Runtime/Core/Private/[PLATFORM]PlatformMemory.cpp


```

In general, you should avoid adding any uses of `PLATFORM_[PLATFORM]`. For example, avoid adding `PLATFORM_XBOXONE` to code outside of a directory named `[PLATFORM]`. Instead, extend the hardware abstraction layer to add a static function, for example in FPlatformMisc:

```
FORCEINLINE static int32 GetMaxPathLength()
{
    return 128;
}


```

Platforms can then override this function, returning either a platform-specific constant value or even using platform APIs to determine the result. If you force-inline the function it has the same performance characteristics as using a define.

In cases where a define is absolutely necessary, create new `#define` directives that describe particular properties that can apply to a platform, for example `PLATFORM_USE_PTHREADS`. Set the default value in `Platform.h` and override for any platforms which require it in the platform-specific `Platform.h` file.

For example, in `Platform.h` we have:

```
#ifndef PLATFORM_USE_PTHREADS
    #define PLATFORM_USE_PTHREADS 1
#endif


```

`WindowsPlatform.h` has:

```
#define PLATFORM_USE_PTHREADS 0


```

Cross-platform code can then use the define directly without needing to know the platform.

```
#if PLATFORM_USE_PTHREADS
    #include "HAL/PThreadRunnableThread.h"
#endif


```

We centralize the platform-specific details of the engine which allows details to be contained entirely within platform-specific source files. Doing so makes it easier to maintain the engine across multiple platforms, additionally you are able to port code to new platforms without the need to scour the codebase for platform-specific defines.

Keeping platform code in platform-specific folders is also a requirement for NDA platforms such as PlayStation, Xbox and Nintendo Switch.

It is important to ensure the code compiles and runs regardless of whether the `[PLATFORM]` subdirectory is present. In other words, cross-platform code should never be dependent on platform-specific code.
