---
title: Memory<T>和Span<T>使用准则
description: 本文介绍Memory<T> 和Span<T>，它们是 .NET Core 中可在管道中使用的结构化数据的缓冲区。
ms.date: 10/01/2018
helpviewer_keywords:
- Memory&lt;T&gt; and Span&lt;T&gt; best practices
- using Memory&lt;T&gt; and Span&lt;T&gt;
ms.openlocfilehash: d9a50fa18e027b6df7415438e1a5584003f7a094
ms.sourcegitcommit: 358a28048f36a8dca39a9fe6e6ac1f1913acadd5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/23/2020
ms.locfileid: "85245591"
---
# <a name="memoryt-and-spant-usage-guidelines"></a>Memory\<T>和Span\<T>使用准则

.NET Core 包括多个表示内存的任意连续区域的类型。 .NET Core 2.0 引入了 <xref:System.Span%601> 和 <xref:System.ReadOnlySpan%601>，它们是可由托管或非托管内存提供支持的轻量级内存缓冲区。 由于这些类型只能存储在堆栈上，因此它们不适用于多种方案，包括异步方法调用。 .NET Core 2.1 添加了其他一些类型，包括 <xref:System.Memory%601>、<xref:System.ReadOnlyMemory%601>、<xref:System.Buffers.IMemoryOwner%601> 和 <xref:System.Buffers.MemoryPool%601>。 与 <xref:System.Span%601> 相同，<xref:System.Memory%601> 及其相关类型可以由托管和非托管内存提供支持。 与 <xref:System.Span%601> 不同，<xref:System.Memory%601> 可以存储在托管堆上。

<xref:System.Span%601> 和 <xref:System.Memory%601> 都是可用于管道的结构化数据的缓冲区。 也就是说，它们设计的目的是将某些或所有数据有效地传递到管道中的组件，这些组件可以对其进行处理并（可选）修改缓冲区。 由于 <xref:System.Memory%601> 及其相关类型可由多个组件或多个线程访问，因此开发人员必须遵循一些标准使用准则才能生成可靠的代码。

## <a name="owners-consumers-and-lifetime-management"></a>所有者、使用者和生存期管理

由于可以在各个 API 之间传送缓冲区，以及由于缓冲区有时可以从多个线程进行访问，因此请务必考虑生存期管理。 下面介绍三个核心概念：

- **所有权**。 缓冲区实例的所有者负责生存期管理，包括在不再使用缓冲区时将其销毁。 所有缓冲区都拥有一个所有者。 通常，所有者是创建缓冲区或从工厂接收缓冲区的组件。 所有权也可以转让；组件 A 可以将缓冲区的控制权转让给组件 B，此时组件 A 就无法再使用该缓冲区，组件 B 将负责在不再使用缓冲区时将其销毁。

- **使用**。 允许缓冲区实例的使用者通过从中读取并可能写入其中来使用缓冲区实例。 缓冲区一次可以拥有一个使用者，除非提供了某些外部同步机制。 缓冲区的当前使用者不一定是缓冲区的所有者。

- **租用**。 租用是允许特定组件成为缓冲区使用者的时长。

以下伪代码示例阐释了这三个概念。 它包括实例化类型为 <xref:System.Char> 的 <xref:System.Memory%601> 缓冲区的 `Main` 方法，调用 `WriteInt32ToBuffer` 方法以将整数的字符串表示形式写入缓冲区，然后调用 `DisplayBufferToConsole` 方法以显示缓冲区的值。

```csharp
using System;

class Program
{
    // Write 'value' as a human-readable string to the output buffer.
    void WriteInt32ToBuffer(int value, Buffer buffer);

    // Display the contents of the buffer to the console.
    void DisplayBufferToConsole(Buffer buffer);

    // Application code
    static void Main()
    {
        var buffer = CreateBuffer();
        try
        {
            int value = Int32.Parse(Console.ReadLine());
            WriteInt32ToBuffer(value, buffer);
            DisplayBufferToConsole(buffer);
        }
        finally
        {
            buffer.Destroy();
        }
    }
}
```

`Main` 方法创建缓冲区（在此示例中为 <xref:System.Span%601> 实例），因此它是其所有者。 因此，`Main` 将负责在不再使用缓冲区时将其销毁。 这是通过调用缓冲区的 <xref:System.Span%601.Clear?displayProperty=nameWithType> 方法完成的。 （此处的 <xref:System.Span%601.Clear> 方法实际上会清除缓冲区的内存；<xref:System.Span%601> 结构实际上没有销毁缓冲区的方法。）

缓冲区有两个使用者：`WriteInt32ToBuffer` 和 `DisplayBufferToConsole`。 一次只能有一个使用者（先是 `WriteInt32ToBuffer`，然后是 `DisplayBufferToConsole`），这两个使用者都不拥有缓冲区。 另请注意，此上下文中的“使用者”并不意味着以只读形式查看缓冲区；如果提供了以读/写形式查看缓冲区的权限，则使用者可以像 `WriteInt32ToBuffer` 那样修改缓冲区的内容。

`WriteInt32ToBuffer` 方法在方法调用的开始时间和方法返回的时间之间会租用（可以使用）缓冲区。 同样，`DisplayBufferToConsole` 在执行时会租用缓冲区，回退该方法时将解除租用。 （没有用于租用管理的 API；“租用”是概念性内容。）

## <a name="memoryt-and-the-ownerconsumer-model"></a>Memory\<T> 和所有者/使用者模型

如[所有者、使用者和生存期管理](#owners-consumers-and-lifetime-management)部分中指出，缓冲区始终都有一个所有者。 .NET Core 支持以下两种所有权模型：

- 支持单个所有权的模型。 缓冲区在其整个生存期内拥有单个所有者。

- 支持所有权转让的模型。 缓冲区的所有权可以从其原始所有者（其创建者）转让给其他组件，该组件随后将负责缓冲区的生存期管理。 该所有者可以反过来将所有权转让给其他组件等。

使用 <xref:System.Buffers.IMemoryOwner%601?displayProperty=nameWithType> 接口显式管理缓冲区的所有权。 <xref:System.Buffers.IMemoryOwner%601> 支持两种所有权模型。 具有 <xref:System.Buffers.IMemoryOwner%601> 引用的组件拥有缓冲区。 以下示例使用 <xref:System.Buffers.IMemoryOwner%601?> 实例反映 <xref:System.Memory%601> 缓冲区的所有权。

[!code-csharp[ownership](~/samples/snippets/standard/buffers/memory-t/owner/owner.cs)]

也可以使用 [`using`](../../csharp/language-reference/keywords/using-statement.md) 编写此示例：

[!code-csharp[ownership-using](~/samples/snippets/standard/buffers/memory-t/owner-using/owner-using.cs)]

在此代码中：

- `Main` 方法保留对 <xref:System.Buffers.IMemoryOwner%601> 实例的引用，因此 `Main` 方法是缓冲区的所有者。

- `WriteInt32ToBuffer` 和 `DisplayBufferToConsole` 方法接受 <xref:System.Memory%601> 作为公共 API。 因此，它们是缓冲区的使用者。 并且它们一次仅使用一个。

尽管 `WriteInt32ToBuffer` 方法用于将值写入缓冲区，但 `DisplayBufferToConsole` 方法并不如此。 若要反映此情况，可以接受类型为 <xref:System.ReadOnlyMemory%601> 的参数。 有关 <xref:System.ReadOnlyMemory%601> 的详细信息，请参阅[规则 2：如果缓冲区应为只读，则使用 ReadOnlySpan\<T> 或 ReadOnlyMemory\<T>](#rule-2)。

### <a name="ownerless-memoryt-instances"></a>“无所有者”Memory\<T> 实例

无需使用 <xref:System.Buffers.IMemoryOwner%601> 即可创建 <xref:System.Memory%601> 实例。 在这种情况下，缓冲区的所有权是隐式的而不是显式的，并且仅支持单所有者模型。 可以通过以下方式达到此目的：

- 直接调用 <xref:System.Memory%601> 构造函数之一，传入 `T[]`，如下面的示例所示。

- 调用 [String.AsMemory](xref:System.MemoryExtensions.AsMemory(System.String)) 扩展方法以生成 `ReadOnlyMemory<char>` 实例。

[!code-csharp[ownerless-memory](~/samples/snippets/standard/buffers/memory-t/ownerless/ownerless.cs)]

最初创建 <xref:System.Memory%601> 实例的方法是缓冲区的隐式所有者。 无法将所有权转让给任何其他组件，因为没有 <xref:System.Buffers.IMemoryOwner%601> 实例可用于进行转让。 （或者，也可以假设运行时的垃圾回收器拥有缓冲区，而所有方法仅使用缓冲区。）

## <a name="usage-guidelines"></a>使用准则

由于拥有内存块，但打算传递到多个组件，因此其中一些组件可能会同时在特定的内存块上运行，请务必建立使用 <xref:System.Memory%601> 和 <xref:System.Span%601> 的准则。  准则是必需的，因为：

- 组件可以在内存块的所有者发布内存块之后保留对该内存块的引用。

- 组件可以同时在运行其他组件的缓冲区上运行，在该过程中会损坏缓冲区中的数据。

- 虽然 <xref:System.Span%601> 的堆栈分配特性优化了性能并使 <xref:System.Span%601> 成为在内存块上运行的首选类型，但它也使 <xref:System.Span%601> 受一些主要限制的约束。 请务必了解何时使用 <xref:System.Span%601> 以及何时使用 <xref:System.Memory%601>。

下面介绍成功使用 <xref:System.Memory%601> 及其相关类型的建议。 除非另有明确说明，否则适用于 <xref:System.Memory%601> 和 <xref:System.Span%601> 的指南也适用于 <xref:System.ReadOnlyMemory%601> 和 <xref:System.ReadOnlySpan%601>。

**规则 1：对于同步 API，如有可能，请使用 Span\<T>（而不是 Memory\<T>）作为参数。**

<xref:System.Span%601> 比 <xref:System.Memory%601> 更通用，可以表示更多种类的连续内存缓冲区。 <xref:System.Span%601> 还提供比 <xref:System.Memory%601> 更好的性能。 最后，尽管无法进行 Span\<T> 到 Memory\<T> 的转换，但可以使用 <xref:System.Memory%601.Span?displayProperty=nameWithType> 属性将 <xref:System.Memory%601> 实例转换为 <xref:System.Span%601>。 因此，如果调用方恰好具有 <xref:System.Memory%601> 实例，则它们不管怎样都可以使用 <xref:System.Span%601> 参数调用你的方法。

使用类型为 <xref:System.Span%601>（而不是类型为 <xref:System.Memory%601>）的参数还可以帮助你编写正确的使用方法实现。 你将自动进行编译时检查，以确保不尝试访问方法租用之外的缓冲区（后续部分将对此进行详细介绍）。

有时，必须使用 <xref:System.Memory%601> 参数（而不是 <xref:System.Span%601> 参数），即使完全同步也是如此。 所依赖的 API 可能仅接受 <xref:System.Memory%601> 参数。 这没有问题，但应注意同步使用 <xref:System.Memory%601> 时所涉及的权衡取舍。

<a name="rule-2"></a>

**规则 2：如果缓冲区应为只读，则使用 ReadOnlySpan\<T> 或 ReadOnlyMemory\<T>。**

在前面的示例中，`DisplayBufferToConsole` 方法仅从缓冲区读取；它不会修改缓冲区的内容。 方法签名应进行如下更改。

```csharp
void DisplayBufferToConsole(ReadOnlyMemory<char> buffer);
```

事实上，如果我们结合使用此规则和规则 1，我们可以做得更好，并按如下所示重写方法签名：

```csharp
void DisplayBufferToConsole(ReadOnlySpan<char> buffer);
```

`DisplayBufferToConsole` 方法现在几乎适用于每一个能够想到的缓冲区类型：`T[]`，使用 [stackalloc](../../csharp/language-reference/operators/stackalloc.md) 分配的存储等。 甚至可以向其直接传递 <xref:System.String>！

**规则 3：如果方法接受 Memory\<T> 并返回 `void`，则在方法返回之后不得使用 Memory\<T> 实例。**

这与前面提到的“租用”概念相关。 返回 void 的方法对 <xref:System.Memory%601> 实例的租用将在进入该方法时开始，并在退出该方法时结束。 请考虑以下示例，该示例会基于控制台中的输入在循环中调用 `Log`。

[!code-csharp[void-returning](~/samples/snippets/standard/buffers/memory-t/void-returning/void-returning.cs#1)]

如果 `Log` 是完全同步的方法，则此代码将按预期运行，因为在任何给定时间只有一个活动的内存实例使用者。
但请假设 `Log` 具有此实现。

```csharp
// !!! INCORRECT IMPLEMENTATION !!!
static void Log(ReadOnlyMemory<char> message)
{
    // Run in background so that we don't block the main thread while performing IO.
    Task.Run(() =>
    {
        StreamWriter sw = File.AppendText(@".\input-numbers.dat");
        sw.WriteLine(message);
    });
}
```

在此实现中，`Log` 违反了其租用，因为它在返回原始方法之后仍尝试在后台使用 <xref:System.Memory%601> 示例。 `Main` 方法可能会在 `Log` 尝试从缓冲区进行读取时更改缓冲区，这可能导致数据损坏。

有多种方法可解决此问题：

- `Log` 方法可以按 `Log` 方法的以下实现所示返回 <xref:System.Threading.Tasks.Task>，而不是 `void`。

   [!code-csharp[task-returning](~/samples/snippets/standard/buffers/memory-t/task-returning2/task-returning2.cs#1)]

- 可以改为按如下所示实现 `Log`：

   [!code-csharp[defensive-copy](~/samples/snippets/standard/buffers/memory-t/task-returning/task-returning.cs#1)]

**规则 4：如果方法接受 Memory\<T> 并返回某个任务，则在该任务转换为终止状态之后不得使用 Memory\<T> 实例。**

这只是规则 3 的异步变体。 可以按如下所示编写前面示例中的 `Log` 方法以遵守此规则：

[!code-csharp[task-returning-async](~/samples/snippets/standard/buffers/memory-t/void-returning-async/void-returning-async.cs#1)]

此处的“终止状态”表示任务转换为已完成、已出错或已取消状态。 换而言之，“终止状态”表示“导致等待引发或继续执行的任何状态”。

此指南适用于返回 <xref:System.Threading.Tasks.Task>、<xref:System.Threading.Tasks.Task%601>、<xref:System.Threading.Tasks.ValueTask%601> 或任何类似类型的方法。

**规则 5：如果构造函数接受 Memory\<T> 作为参数，则假定构造对象上的实例方法是 Memory\<T> 实例的使用者。**

请看下面的示例：

```csharp
class OddValueExtractor
{
    public OddValueExtractor(ReadOnlyMemory<int> input);
    public bool TryReadNextOddValue(out int value);
}

void PrintAllOddValues(ReadOnlyMemory<int> input)
{
    var extractor = new OddValueExtractor(input);
    while (extractor.TryReadNextOddValue(out int value))
    {
      Console.WriteLine(value);
    }
}
```

此处的 `OddValueExtractor` 构造函数接受 `ReadOnlyMemory<int>` 作为构造函数参数，因此构造函数本身是 `ReadOnlyMemory<int>` 实例的使用者，并且返回值上的所有实例方法也是原始 `ReadOnlyMemory<int>` 实例的使用者。 这意味着 `TryReadNextOddValue` 使用 `ReadOnlyMemory<int>` 实例，即使该实例未直接传递到 `TryReadNextOddValue` 方法也是如此。

**规则 6：如果类型具有可设置的 Memory\<T> 类型的属性（或等效实例方法），则假定该对象上的实例方法是 Memory\<T> 实例的使用者。**

这实际上只是规则 5 的变体。 存在此规则的原因是假定属性 setter 或等效方法捕获并保留其输入，因此同一对象上的实例方法可能会使用已捕获状态。

下面的示例将触发以下规则：

```csharp
class Person
{
    // Settable property.
    public Memory<char> FirstName { get; set; }

    // alternatively, equivalent "setter" method
    public SetFirstName(Memory<char> value);

    // alternatively, a public settable field
    public Memory<char> FirstName;
}
```

**规则 7：如果具有 IMemoryOwner\<T> 引用，则必须在某些时候对其进行处理或转让其所有权（但不同时执行两个操作）。**

由于 <xref:System.Memory%601> 实例可能由托管或非托管内存提供支持，因此在对 <xref:System.Memory%601> 实例执行的工作完成之后，所有者必须调用 <xref:System.Buffers.MemoryPool%601.Dispose%2A?displayProperty=nameWithType>。 此外，所有者可能会将 <xref:System.Buffers.IMemoryOwner%601> 实例的所有权转让给其他组件，同时获取组件将负责在适当时间调用 <xref:System.Buffers.MemoryPool%601.Dispose%2A?displayProperty=nameWithType>（稍后将对此进行详细介绍）。

调用 <xref:System.Buffers.MemoryPool%601.Dispose%2A> 方法失败可能会导致非托管内存泄漏或其他性能降低。

此规则也适用于调用工厂方法（如 <xref:System.Buffers.MemoryPool%601.Rent%2A?displayProperty=nameWithType>）的代码。 调用方将成为返回的 <xref:System.Buffers.IMemoryOwner%601> 的所有者，并负责在完成后处理实例。

**规则 8：如果 API 接口中具有 IMemoryOwner\<T> 参数，即表示你接受该实例的所有权。**

接受此类型的实例表示组件打算获取此实例的所有权。 组件将负责根据规则 7 进行正确处理。

在方法调用完成后，将 <xref:System.Buffers.IMemoryOwner%601> 实例的所有权转让给其他组件的任何组件应不再使用该实例。

> [!IMPORTANT]
> 如果构造函数接受 <xref:System.Buffers.IMemoryOwner%601> 作为参数，则其类型应实现 <xref:System.IDisposable>，并且 <xref:System.IDisposable.Dispose%2A> 方法应调用 <xref:System.Buffers.MemoryPool%601.Dispose%2A?displayProperty=nameWithType>。

**规则 9：如果正在包装同步 P/Invoke 方法，则 API 应接受 Span\<T> 作为参数。**

根据规则 1，<xref:System.Span%601> 通常是用于同步 API 的正确类型。 可以通过 [`fixed`](../../csharp/language-reference/keywords/fixed-statement.md) 关键字固定 <xref:System.Span%601> 实例，如下面的示例所示。

```csharp
using System.Runtime.InteropServices;

[DllImport(...)]
private static extern unsafe int ExportedMethod(byte* pbData, int cbData);

public unsafe int ManagedWrapper(Span<byte> data)
{
    fixed (byte* pbData = &MemoryMarshal.GetReference(data))
    {
        int retVal = ExportedMethod(pbData, data.Length);

        /* error checking retVal goes here */

        return retVal;
    }
}
```

在上一示例中，如果输入跨度为空，则 `pbData` 可以为 Null。 如果导出的方法绝对需要 `pbData` 为非 Null，即使 `cbData` 为 0，则可以按如下所示实现该方法：

```csharp
public unsafe int ManagedWrapper(Span<byte> data)
{
    fixed (byte* pbData = &MemoryMarshal.GetReference(data))
    {
        byte dummy = 0;
        int retVal = ExportedMethod((pbData != null) ? pbData : &dummy, data.Length);

        /* error checking retVal goes here */

        return retVal;
    }
}
```

**规则 10：如果正在包装异步 P/Invoke 方法，则 API 应接受 Memory\<T> 作为参数。**

由于不能在异步操作中使用 [`fixed`](../../csharp/language-reference/keywords/fixed-statement.md) 关键字，因此使用 <xref:System.Memory%601.Pin%2A?displayProperty=nameWithType> 方法固定 <xref:System.Memory%601> 实例，无论该实例表示的连续内存类型为何。 下面的示例演示了如何使用此 API 执行异步 P/Invoke 调用。

```csharp
using System.Runtime.InteropServices;

[UnmanagedFunctionPointer(...)]
private delegate void OnCompletedCallback(IntPtr state, int result);

[DllImport(...)]
private static extern unsafe int ExportedAsyncMethod(byte* pbData, int cbData, IntPtr pState, IntPtr lpfnOnCompletedCallback);

private static readonly IntPtr _callbackPtr = GetCompletionCallbackPointer();

public unsafe Task<int> ManagedWrapperAsync(Memory<byte> data)
{
    // setup
    var tcs = new TaskCompletionSource<int>();
    var state = new MyCompletedCallbackState
    {
        Tcs = tcs
    };
    var pState = (IntPtr)GCHandle.Alloc(state);

    var memoryHandle = data.Pin();
    state.MemoryHandle = memoryHandle;

    // make the call
    int result;
    try
    {
        result = ExportedAsyncMethod((byte*)memoryHandle.Pointer, data.Length, pState, _callbackPtr);
    }
    catch
    {
        ((GCHandle)pState).Free(); // cleanup since callback won't be invoked
        memoryHandle.Dispose();
        throw;
    }

    if (result != PENDING)
    {
        // Operation completed synchronously; invoke callback manually
        // for result processing and cleanup.
        MyCompletedCallbackImplementation(pState, result);
    }

    return tcs.Task;
}

private static void MyCompletedCallbackImplementation(IntPtr state, int result)
{
    GCHandle handle = (GCHandle)state;
    var actualState = (MyCompletedCallbackState)(handle.Target);
    handle.Free();
    actualState.MemoryHandle.Dispose();

    /* error checking result goes here */

    if (error)
    {
        actualState.Tcs.SetException(...);
    }
    else
    {
        actualState.Tcs.SetResult(result);
    }
}

private static IntPtr GetCompletionCallbackPointer()
{
    OnCompletedCallback callback = MyCompletedCallbackImplementation;
    GCHandle.Alloc(callback); // keep alive for lifetime of application
    return Marshal.GetFunctionPointerForDelegate(callback);
}

private class MyCompletedCallbackState
{
    public TaskCompletionSource<int> Tcs;
    public MemoryHandle MemoryHandle;
}
```

## <a name="see-also"></a>请参阅

- <xref:System.Memory%601?displayProperty=nameWithType>
- <xref:System.Buffers.IMemoryOwner%601?displayProperty=nameWithType>
- <xref:System.Span%601?displayProperty=nameWithType>
