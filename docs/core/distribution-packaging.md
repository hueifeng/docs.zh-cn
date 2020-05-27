---
title: .NET Core 分发打包
description: 了解如何为 .NET Core 打包、命名并进行版本控制以进行分发。
author: tmds
ms.date: 10/09/2019
ms.openlocfilehash: a345aeded29b3058c6c56abbff439ea26cbc7afb
ms.sourcegitcommit: cdf5084648bf5e77970cbfeaa23f1cab3e6e234e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/01/2020
ms.locfileid: "81386638"
---
# <a name="net-core-distribution-packaging"></a>.NET Core 分发打包

由于 .NET Core 现已可用于更多平台，因此了解如何为其打包、命名并进行版本控制将很有用。 这样，无论用户选择在哪里运行 .NET，包维护人员均可以帮助确保获得一致的体验。 本文对以下用户非常有用：

- 尝试从源生成 .NET Core。
- 想要更改 .NET Core CLI，但更改可能会影响生成的布局或包。

## <a name="disk-layout"></a>磁盘布局

安装时，.NET Core 包含一些组件，这些组件在文件系统中排列如下：

```
{dotnet_root}                                     (*)
├── dotnet                       (1)
├── LICENSE.txt                  (8)
├── ThirdPartyNotices.txt        (8)
├── host                                          (*)
│   └── fxr                                       (*)
│       └── <fxr version>        (2)
├── sdk                                           (*)
│   ├── <sdk version>            (3)
│   └── NuGetFallbackFolder      (4)              (*)
├── packs                                         (*)
│   ├── Microsoft.AspNetCore.App.Ref              (*)
│   │   └── <aspnetcore ref version>     (11)
│   ├── Microsoft.NETCore.App.Ref                 (*)
│   │   └── <netcore ref version>        (12)
│   ├── Microsoft.NETCore.App.Host.<rid>          (*)
│   │   └── <apphost version>            (13)
│   ├── Microsoft.WindowsDesktop.App.Ref          (*)
│   │   └── <desktop ref version>        (14)
│   └── NETStandard.Library.Ref                   (*)
│       └── <netstandard version>        (15)
├── shared                                        (*)
│   ├── Microsoft.NETCore.App                     (*)
│   │   └── <runtime version>     (5)
│   ├── Microsoft.AspNetCore.App                  (*)
│   │   └── <aspnetcore version>  (6)
│   ├── Microsoft.AspNetCore.All                  (*)
│   │   └── <aspnetcore version>  (6)
│   └── Microsoft.WindowsDesktop.App              (*)
│       └── <desktop app version> (7)
└── templates                                     (*)
│   └── <templates version>      (17)
/
├── etc/dotnet
│       └── install_location     (16)
├── usr/share/man/man1
│       └── dotnet.1.gz          (9)
└── usr/bin
        └── dotnet               (10)
```

- (1) **dotnet** 主机（也称为“muxer”）有两个不同角色：激活运行时以启动应用程序，及激活 SDK 以向其分派命令。 主机是本机可执行文件 (`dotnet.exe`)。

主机只有一个，不过大部分的其他组件都在带有版本的目录中（2、3、5 和 6）。 这意味着系统上可存在多个版本，因为它们是并排安装的。

- (2) **host/fxr/\<fxr version>** 包含了主机所使用的框架解析逻辑。 主机采用已安装的最新 hostfxr。 在执行 .NET Core 应用程序时，hostfxr 负责选择合适的运行时。 例如，为 .NET Core 2.0.0 生成的应用程序会使用 2.0.5 运行时（如果可用）。 同样，hostfxr 在开发期间也会选择适当的 SDK。

- (3) **sdk/\<sdk version>** SDK（也称为“工具”）是一组托管工具，可用于编写和生成 .NET Core 库和应用程序。 SDK 包括 .NET Core CLI、托管的语言编译器、MSBuild 及相关生成任务和目标、NuGet、新项目模板等。

- (4) **sdk/NuGetFallbackFolder** 包含 SDK 在还原操作期间使用的 NuGet 包的缓存，例如在运行 `dotnet restore` 或 `dotnet build` 时。 此文件夹仅在 .NET Core 3.0 之前使用。 不能从源生成它，因为它包含来自 `nuget.org` 的预构建二进制资产。

 **shared** 文件夹包含框架。共享框架在中心位置提供一组库，使得不同的应用程序可以使用它们。

- (5) **shared/Microsoft.NETCore.App/\<runtime version>** 此框架包含.NET Core 运行时和支持托管库。

- (6) **shared/Microsoft.AspNetCore.{App,All}/\<aspnetcore version>** 包含 ASP.NET Core 库。 已开发且支持 `Microsoft.AspNetCore.App` 下的库（作为 .NET Core 项目的一部分）。 `Microsoft.AspNetCore.All` 下的库是一个超集，其中还包含第三方库。

- (7) **shared/Microsoft.Desktop.App/\<desktop app version>** 包含 Windows 桌面库。 在非 Windows 平台上不包含此项。

- (8) **LICENSE.txt,ThirdPartyNotices.txt** 分别是 .NET Core 许可证和 .NET Core 中使用的第三方库的许可证。

- (9,10) **dotnet.1.gz, dotnet** `dotnet.1.gz` 是 dotnet 手册页。 `dotnet` 是指向 dotnet 主机 (1) 的符号链接。 这些文件安装在已知位置用于系统集成。

- (11,12) **Microsoft.NETCore.App.Ref,Microsoft.AspNetCore.App.Ref** 分别描述了 `x.y` 版本 .NET Core 和 ASP.NET Core 的 API。 针对这些目标版本进行编译时，将使用这些包。

- (13) **Microsoft.NETCore.App.Host.\<rid>** 包含平台 `rid` 的本机二进制文件。 将 .NET Core 应用程序编译为适用于该平台的本机二进制文件时，将使用此二进制文件作为模板。

- (14) **Microsoft.WindowsDesktop.App.Ref** 介绍 Windows 桌面应用程序 `x.y` 版本的 API。 在针对该目标进行编译时，将使用这些文件。 在非 Windows 平台上不提供此项。

- (15) **NETStandard.Library.Ref** 描述了 netstandard `x.y` API。 在针对该目标进行编译时，将使用这些文件。

- (16)“/etc/dotnet/install_location”是一个包含 `{dotnet_root}` 完整路径的文件  。 该路径可能以换行符结尾。 根路径为 `/usr/share/dotnet` 时无需添加此文件。

- (17) templates  包含 SDK 使用的模板。 例如，`dotnet new` 在此处查找项目模板。

标记为 `(*)` 的文件夹被多个包使用。 某些包格式（例如，`rpm`）需要对此类文件夹进行特殊处理。 包维护人员必须处理这个问题。

## <a name="recommended-packages"></a>推荐的包

.NET Core 版本控制基于运行时组件 `[major].[minor]` 版本号。
SDK 版本采用相同的 `[major].[minor]`，并有一个独立的 `[patch]`，它为 SDK 合并了功能和修补语义。
例如：SDK 版本 2.2.302 是支持 2.2 运行时的 SDK 的第 3 个功能版本的第 2 个补丁版本。 有关版本控制的工作原理的详细信息，请参阅 [.NET Core 版本控制概述](./versions/index.md)。

一些包在自己的名称中就包含一部分版本号。 这允许你安装特定版本。
版本名称中不包含版本的剩余部分。 这允许 OS 包管理器更新这些包（例如，自动安装安全修补程序）。 支持的包管理器特定于 Linux。

下面列出了推荐的包：

- `dotnet-sdk-[major].[minor]` - 安装特定运行时的最新 SDK
  - **版本：** \<运行时版本>
  - **示例：** dotnet-sdk-2.1
  - **包含：** (3),(4)
  - **依赖项：** `dotnet-runtime-[major].[minor]`、`aspnetcore-runtime-[major].[minor]`、`dotnet-targeting-pack-[major].[minor]`、`aspnetcore-targeting-pack-[major].[minor]`、`netstandard-targeting-pack-[netstandard_major].[netstandard_minor]`、`dotnet-apphost-pack-[major].[minor]`、`dotnet-templates-[major].[minor]`

- `aspnetcore-runtime-[major].[minor]` - 安装特定 ASP.NET Core 运行时
  - **版本：** \<aspnetcore 运行时版本>
  - **示例：** aspnetcore-runtime-2.1
  - **包含：** (6)
  - **依赖项：** `dotnet-runtime-[major].[minor]`

- `dotnet-runtime-deps-[major].[minor]` _（可选）_ - 安装运行自包含应用程序的依赖项
  - **版本：** \<运行时版本>
  - **示例：** dotnet-runtime-deps-2.1
  - **依赖项：** _特定于分发的依赖项_

- `dotnet-runtime-[major].[minor]` - 安装特定运行时
  - **版本：** \<运行时版本>
  - **示例：** dotnet-runtime-2.1
  - **包含：** (5)
  - **依赖项：** `dotnet-hostfxr-[major].[minor]`、`dotnet-runtime-deps-[major].[minor]`

- `dotnet-hostfxr-[major].[minor]` - 依赖项
  - **版本：** \<运行时版本>
  - **示例：** dotnet-hostfxr-3.0
  - **包含：** (2)
  - **依赖项：** `dotnet-host`

- `dotnet-host` - 依赖项
  - **版本：** \<运行时版本>
  - **示例：** dotnet-host
  - **包含：** (1),(8),(9),(10),(16)

- `dotnet-apphost-pack-[major].[minor]` - 依赖项
  - **版本：** \<运行时版本>
  - **包含：** (13)

- `dotnet-targeting-pack-[major].[minor]` - 允许面向非最新的运行时
  - **版本：** \<运行时版本>
  - **包含：** (12)

- `aspnetcore-targeting-pack-[major].[minor]` - 允许面向非最新的运行时
  - **版本：** \<aspnetcore 运行时版本>
  - **包含：** (11)

- `netstandard-targeting-pack-[netstandard_major].[netstandard_minor]` - 允许面向 netstandard 版本
  - **版本：** \<sdk 版本>
  - **包含：** (15)

- `dotnet-templates-[major].[minor]`
  - **版本：** \<sdk 版本>
  - **包含：** (15)

`dotnet-runtime-deps-[major].[minor]` 需要了解发行版特定依赖项  。 因为发行版生成系统可能能够自动派生包，所以包是可选的，如果选择，会将这些依赖项直接添加到 `dotnet-runtime-[major].[minor]` 包中。

当包内容位于受版本控制的文件夹下时，包名称 `[major].[minor]` 与受版本控制的文件夹名称匹配。 对于所有包（除 `netstandard-targeting-pack-[netstandard_major].[netstandard_minor]` 外），这也与 .NET Core 版本匹配。

包间的依赖关系应使用“等于或大于”版本要求  。 例如，`dotnet-sdk-2.2:2.2.401` 要求 `aspnetcore-runtime-2.2 >= 2.2.6`。 这使用户可以通过根包（例如 `dnf update dotnet-sdk-2.2`）升级其安装。

大多数分发都需要从源中构建所有项目。 这对包有一些影响：

- 不能简单地从源生成 `shared/Microsoft.AspNetCore.All` 下的第三方库。 因此 `aspnetcore-runtime` 包中省略了该文件夹。

- 使用 `nuget.org` 中的二进制项目填充了 `NuGetFallbackFolder`。 它应保留为空。

多个 `dotnet-sdk` 包可能会为 `NuGetFallbackFolder` 提供同样的文件。 若要避免包管理器出现问题，这些文件应完全相同（包括校验和、修改日期等等）。

## <a name="building-packages"></a>生成包

[dotnet/source-build](https://github.com/dotnet/source-build) 存储库中说明了如何生成 .NET Core SDK 的源 tarball 及其所有组件。 源版本存储库中的输出内容符合本文第一部分中所描述的布局。
