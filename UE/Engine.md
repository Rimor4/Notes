# Engine Structure

## 文件管理

1. 自定义引擎版本需在**注册表**`HKEY_CURRENT_USER\SOFTWARE\Epic Games\UnrealEngine\Builds`中注册。同时也要在
   uproject 文件等同步
2. 可以在`Saved`文件夹中恢复已保存的内容到`Contents`。
3. 只应该同步` Config``Content``Source``Uproject``Binaries `文件（夹），不应同步`Saved`和`Intermediate`（这两个只和本地项目设置有关），如果出现问题但是远程协作者正常，可以尝试删除这两个文件夹。

## 编译系统

- **UnrealBuildTool**（UBT，C#）：UE4 的自定义工具，来编译 UE4 的逐个模块并处理依赖等。我们编写的 Target.cs，Build.cs 都是为这个工具服务的。
- **UnrealHeaderTool**（UHT，C++）：UE4 的 C++代码解析生成工具，我们在代码里写的那些宏 UCLASS 等和#include "\*.generated.h"都为 UHT 提供了信息来生成相应的 C++反射代码。
