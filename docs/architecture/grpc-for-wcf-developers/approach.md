---
title: GRPC 如何为 WCF 开发人员提供 RPC gRPC
description: 比较 WCF 的主要功能和 gRPC。
ms.date: 09/02/2019
ms.openlocfilehash: b924d2f20b54de2d6476a0b27626d5dd7fd0986f
ms.sourcegitcommit: 5fb5b6520b06d7f5e6131ec2ad854da302a28f2e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/03/2019
ms.locfileid: "74711480"
---
# <a name="how-grpc-approaches-rpc"></a><span data-ttu-id="f0d85-103">gRPC 如何处理 RPC</span><span class="sxs-lookup"><span data-stu-id="f0d85-103">How gRPC approaches RPC</span></span>

<span data-ttu-id="f0d85-104">Windows Communication Foundation （WCF）和 gRPC 都是*远程过程调用*（RPC）模式的两种实现。</span><span class="sxs-lookup"><span data-stu-id="f0d85-104">Windows Communication Foundation (WCF) and gRPC are both implementations of the *Remote Procedure Call* (RPC) pattern.</span></span> <span data-ttu-id="f0d85-105">此模式旨在调用在不同的计算机上或在不同进程中运行的服务，如客户端应用程序中的方法调用。</span><span class="sxs-lookup"><span data-stu-id="f0d85-105">This pattern aims to make calls to services that run on a different machine, or in a different process, work seamlessly, like method calls in the client application.</span></span> <span data-ttu-id="f0d85-106">尽管 WCF 和 gRPC 的目标是相同的，但实现的详细信息是完全不同的。</span><span class="sxs-lookup"><span data-stu-id="f0d85-106">While the aims of WCF and gRPC are the same, the details of the implementation are quite different.</span></span>

<span data-ttu-id="f0d85-107">下表设置了 WCF 的主要功能与 gRPC 的关系，以及在何处可以找到更详细的说明。</span><span class="sxs-lookup"><span data-stu-id="f0d85-107">The following table sets out how the key features of WCF relate to gRPC, and where you can find more detailed explanations.</span></span>

| <span data-ttu-id="f0d85-108">特征</span><span class="sxs-lookup"><span data-stu-id="f0d85-108">Features</span></span> | <span data-ttu-id="f0d85-109">WCF</span><span class="sxs-lookup"><span data-stu-id="f0d85-109">WCF</span></span> | <span data-ttu-id="f0d85-110">gRPC</span><span class="sxs-lookup"><span data-stu-id="f0d85-110">gRPC</span></span> |
| -------- | --- | ---- |
| <span data-ttu-id="f0d85-111">目标</span><span class="sxs-lookup"><span data-stu-id="f0d85-111">Objective</span></span> | <span data-ttu-id="f0d85-112">独立于网络实现的业务代码。</span><span class="sxs-lookup"><span data-stu-id="f0d85-112">Separate business code from networking implementation.</span></span> | <span data-ttu-id="f0d85-113">将业务代码与接口定义和网络实现分离。</span><span class="sxs-lookup"><span data-stu-id="f0d85-113">Separate business code from interface definition and networking implementation.</span></span> |
| <span data-ttu-id="f0d85-114">定义服务和消息（章节3-4）</span><span class="sxs-lookup"><span data-stu-id="f0d85-114">Define services and messages (chapters 3-4)</span></span>  | <span data-ttu-id="f0d85-115">服务协定、操作协定和数据协定。</span><span class="sxs-lookup"><span data-stu-id="f0d85-115">Service Contract, Operation Contract, and Data Contract.</span></span> | <span data-ttu-id="f0d85-116">使用 proto 文件声明服务和消息。</span><span class="sxs-lookup"><span data-stu-id="f0d85-116">Uses proto file to declare services and messages.</span></span> |
| <span data-ttu-id="f0d85-117">语言（章节3-5）</span><span class="sxs-lookup"><span data-stu-id="f0d85-117">Language (chapters 3-5)</span></span> | <span data-ttu-id="f0d85-118">用或 Visual Basic C#编写的协定。</span><span class="sxs-lookup"><span data-stu-id="f0d85-118">Contracts written in C# or Visual Basic.</span></span> | <span data-ttu-id="f0d85-119">协议缓冲区语言。</span><span class="sxs-lookup"><span data-stu-id="f0d85-119">Protocol Buffer language.</span></span> |
| <span data-ttu-id="f0d85-120">线路格式（第3章）</span><span class="sxs-lookup"><span data-stu-id="f0d85-120">Wire format (chapter 3)</span></span> | <span data-ttu-id="f0d85-121">可配置，包括 SOAP/XML、纯 XML、JSON 和 .NET Binary。</span><span class="sxs-lookup"><span data-stu-id="f0d85-121">Configurable, including SOAP/XML, Plain XML, JSON, and .NET Binary.</span></span> | <span data-ttu-id="f0d85-122">协议缓冲区二进制格式（尽管可以使用其他格式）。</span><span class="sxs-lookup"><span data-stu-id="f0d85-122">Protocol Buffer binary format (although it's possible to use other formats).</span></span>
| <span data-ttu-id="f0d85-123">互操作性（第4章）</span><span class="sxs-lookup"><span data-stu-id="f0d85-123">Interoperability (chapter 4)</span></span> | <span data-ttu-id="f0d85-124">使用 SOAP over HTTP 时。</span><span class="sxs-lookup"><span data-stu-id="f0d85-124">When using SOAP over HTTP.</span></span> | <span data-ttu-id="f0d85-125">官方支持： .NET、Java、Python、JavaScript、C/C++、中转、Rust、Ruby、Swift、DART、PHP。</span><span class="sxs-lookup"><span data-stu-id="f0d85-125">Official support: .NET, Java, Python, JavaScript, C/C++, Go, Rust, Ruby, Swift, Dart, PHP.</span></span> <span data-ttu-id="f0d85-126">来自社区的其他语言的非正式支持。</span><span class="sxs-lookup"><span data-stu-id="f0d85-126">Unofficial support for other languages from the community.</span></span> |
| <span data-ttu-id="f0d85-127">网络（第4章）</span><span class="sxs-lookup"><span data-stu-id="f0d85-127">Networking (chapter 4)</span></span> | <span data-ttu-id="f0d85-128">在运行时配置。</span><span class="sxs-lookup"><span data-stu-id="f0d85-128">Configured at runtime.</span></span> <span data-ttu-id="f0d85-129">在 Wcf-nettcp、HTTP 和 MSMQ 之间切换。</span><span class="sxs-lookup"><span data-stu-id="f0d85-129">Switch between NetTCP, HTTP, and MSMQ.</span></span> | <span data-ttu-id="f0d85-130">HTTP/2，当前通过 TCP ASP.NET Core gRPC。</span><span class="sxs-lookup"><span data-stu-id="f0d85-130">HTTP/2, currently over TCP only with ASP.NET Core gRPC.</span></span> |
| <span data-ttu-id="f0d85-131">方法（第4章）</span><span class="sxs-lookup"><span data-stu-id="f0d85-131">Approach (chapter 4)</span></span> | <span data-ttu-id="f0d85-132">基类中序列化、反序列化和网络代码的运行时生成。</span><span class="sxs-lookup"><span data-stu-id="f0d85-132">Runtime generation of serialization, deserialization, and networking code in base classes.</span></span> | <span data-ttu-id="f0d85-133">生成时生成序列化、反序列化和基类中的网络代码。</span><span class="sxs-lookup"><span data-stu-id="f0d85-133">Build-time generation of serialization, deserialization, and networking code in base classes.</span></span> |
| <span data-ttu-id="f0d85-134">安全性（第6章）</span><span class="sxs-lookup"><span data-stu-id="f0d85-134">Security (chapter 6)</span></span> | <span data-ttu-id="f0d85-135">身份验证、WS 安全和消息加密。</span><span class="sxs-lookup"><span data-stu-id="f0d85-135">Authentication, WS-Security, message encryption.</span></span> | <span data-ttu-id="f0d85-136">凭据、ASP.NET Core 安全性、TLS 网络。</span><span class="sxs-lookup"><span data-stu-id="f0d85-136">Credentials, ASP.NET Core security, TLS networking.</span></span> |

>[!div class="step-by-step"]
><span data-ttu-id="f0d85-137">[上一页](grpc-overview.md)
>[下一页](interface-definition-language.md)</span><span class="sxs-lookup"><span data-stu-id="f0d85-137">[Previous](grpc-overview.md)
[Next](interface-definition-language.md)</span></span>