---
title: 数据协定序列化程序支持的类型
ms.date: 03/30/2017
helpviewer_keywords:
- serialization [WCF], supported types
ms.assetid: 7381b200-437a-4506-9556-d77bf1bc3f34
ms.openlocfilehash: f3eedda6c9493688680099f4b07810aebf69ff8a
ms.sourcegitcommit: 99b153b93bf94d0fecf7c7bcecb58ac424dfa47c
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/25/2020
ms.locfileid: "80249456"
---
# <a name="types-supported-by-the-data-contract-serializer"></a>数据协定序列化程序支持的类型

Windows 通信基础 （WCF）<xref:System.Runtime.Serialization.DataContractSerializer>使用 作为默认序列化引擎将数据转换为 XML 并将 XML 转换回数据。 <xref:System.Runtime.Serialization.DataContractSerializer> 是专为序列化数据协定 ** 类型而设计的。 但是，它支持许多其他可视为具有隐式数据协定的类型。 以下是可以序列化的类型的完整列表：

- 具有不带参数的构造函数的所有公开可见类型。

- 数据协定类型。 这些是已应用了 <xref:System.Runtime.Serialization.DataContractAttribute> 属性的类型。 表示业务对象的新自定义类型通常应作为数据协定类型创建。 有关详细信息，请参阅[使用数据协定](../../../../docs/framework/wcf/feature-details/using-data-contracts.md)和[可序列化类型](../../../../docs/framework/wcf/feature-details/serializable-types.md)。

- 集合类型。 这些是表示数据列表的类型。 这些类型可以是常规的类型数组或集合类型，例如 <xref:System.Collections.ArrayList> 和 <xref:System.Collections.Generic.Dictionary%602>。 <xref:System.Runtime.Serialization.CollectionDataContractAttribute> 属性可以用于自定义这些类型的序列化，但不是必需的。 有关详细信息，请参阅[数据协定中的收集类型](../../../../docs/framework/wcf/feature-details/collection-types-in-data-contracts.md)。

- 枚举类型。 枚举（包括标志枚举）是可序列化的。 或者，可以使用 <xref:System.Runtime.Serialization.DataContractAttribute> 属性对枚举类型进行标记，在这种情况下，必须使用 <xref:System.Runtime.Serialization.EnumMemberAttribute> 属性对参与序列化的每个成员进行标记。 不序列化未标记的成员。 有关详细信息，请参阅[数据协定中的枚举类型](../../../../docs/framework/wcf/feature-details/enumeration-types-in-data-contracts.md)。

- .NET Framework 基元类型。 集成到 .NET Framework 中的下列类型都可以进行序列化，并可视为基元类型： <xref:System.Byte>、 <xref:System.SByte>、 <xref:System.Int16>、 <xref:System.Int32>、 <xref:System.Int64>、 <xref:System.UInt16>、 <xref:System.UInt32>、 <xref:System.UInt64>、 <xref:System.Single>、 <xref:System.Double>、 <xref:System.Boolean>、 <xref:System.Char>、 <xref:System.Decimal>、 <xref:System.Object>和 <xref:System.String>。

- 其他基元类型。 这些类型不是 .NET Framework 中的基元，但可作为采用序列化的 XML 形式的基元。 这些类型有 <xref:System.DateTime>、 <xref:System.DateTimeOffset>、 <xref:System.TimeSpan>、 <xref:System.Guid>、 <xref:System.Uri>、 <xref:System.Xml.XmlQualifiedName>和 <xref:System.Byte>数组。

  > [!NOTE]
  > 与其他基元类型不同， <xref:System.DateTimeOffset> 默认情况下不是已知类型。 有关详细信息，请参阅[数据协定已知类型](../../../../docs/framework/wcf/feature-details/data-contract-known-types.md)。

- 使用 <xref:System.SerializableAttribute> 属性标记的类型。 .NET 框架基类库中包含的许多类型都属于此类别。 <xref:System.Runtime.Serialization.DataContractSerializer> 完全支持由 .NET Framework 远程处理、 <xref:System.Runtime.Serialization.Formatters.Binary.BinaryFormatter>和 <xref:System.Runtime.Serialization.Formatters.Soap.SoapFormatter>使用的此序列化编程模型，包括支持 <xref:System.Runtime.Serialization.ISerializable> 接口。

- 表示原始 XML 的类型或表示关系数据ADO.NET的类型。 支持 <xref:System.Xml.XmlElement> 和 <xref:System.Xml.XmlNode> 类型的数组作为一种直接表示 XML 的方式。 另外，支持实现 <xref:System.Xml.Serialization.IXmlSerializable> 接口的类型，包括相关的 <xref:System.Xml.Serialization.XmlSchemaProviderAttribute> 属性、 <xref:System.Xml.Linq.XDocument> 和 <xref:System.Xml.Linq.XElement> 类型。 ADO.NET<xref:System.Data.DataTable>类型和<xref:System.Data.DataSet>类型（以及其键入的派生类）都实现<xref:System.Xml.Serialization.IXmlSerializable>接口，因此适合此类别。 有关详细信息，请参阅[数据协定 中的 XML 和ADO.NET类型](../../../../docs/framework/wcf/feature-details/xml-and-ado-net-types-in-data-contracts.md)。

## <a name="limitations-of-using-certain-types-in-partial-trust-mode"></a>在部分信任模式中使用某些类型的限制

以下是在部分信任模式方案中使用某些类型时的限制列表：

- 若要使用 <xref:System.Runtime.Serialization.ISerializable> 在部分受信任的代码中序列化或反序列化实现 <xref:System.Runtime.Serialization.DataContractSerializer> 的类型，则需要 <xref:System.Security.Permissions.SecurityPermissionAttribute.SerializationFormatter%2A> 和 <xref:System.Security.Permissions.SecurityPermissionAttribute.UnmanagedCode%2A> 权限。

- 在[部分信任](../../../../docs/framework/wcf/feature-details/partial-trust.md)模式下运行 WCF 代码时，不支持字段（和`readonly``public``private`） 的序列化和反序列化。 这是因为生成的 IL 无法验证，所以需要提升权限。

- 在部分信任环境中支持 <xref:System.Runtime.Serialization.DataContractSerializer> 和 <xref:System.Xml.Serialization.XmlSerializer> 。 但是，使用 <xref:System.Runtime.Serialization.DataContractSerializer> 时需要遵循以下条件：

  - 所有可序列化的 `[DataContract]` 类型必须为 Public。

  - `[DataMember]` 类型中的所有可序列化的 `[DataContract]` 字段或属性必须是公共字段或属性并且可以读取/写入。 在部分受信任的应用程序中运行 WCF`readonly`时，不支持字段的序列化和反序列化。

  - 支持 `[Serializable]`/`ISerializable]` 编程模型。

  - 必须在代码或计算机级别配置 (`Machine.config`) 中指定已知类型。 出于安全方面的原因，不能在应用程序级配置中指定已知类型。

- 实现 <xref:System.Runtime.Serialization.IObjectReference> 的类型会在部分受信任的环境中引发异常，原因是 <xref:System.Runtime.Serialization.IObjectReference.GetRealObject%2A> 方法需要安全权限 `[SecurityPermission(SecurityAction.LinkDemand, Flags=SecurityPermissionFlag.SerializationFormatter)]`。

## <a name="additional-notes-on-serialization"></a>有关序列化的其他说明

下面的规则也适用于数据协定序列化程序支持的类型：

- 数据协定序列化程序完全支持泛型类型。

- 数据协定序列化器完全支持空值类型。

- 将接口类型视为 <xref:System.Object> 或集合类型（对于集合接口）。

- 支持结构和类。

- <xref:System.Runtime.Serialization.DataContractSerializer>不支持 和 ASP.NET Web 服务使用的<xref:System.Xml.Serialization.XmlSerializer>编程模型。 具体而言，它不支持类似于 <xref:System.Xml.Serialization.XmlElementAttribute> 和 <xref:System.Xml.Serialization.XmlAttributeAttribute>的属性。 要启用对此编程模型的支持，必须将 WCF 切换到使用<xref:System.Xml.Serialization.XmlSerializer>而不是 。 <xref:System.Runtime.Serialization.DataContractSerializer>

- 以特殊方式处理 <xref:System.DBNull> 类型。 此类型是一个单一类型，在反序列化时，反序列化程序遵循单一约束并将所有 `DBNull` 引用指向单一实例。 因为 `DBNull` 是可序列化的类型，所以它需要 <xref:System.Security.Permissions.SecurityPermissionAttribute.SerializationFormatter%2A> 权限。

## <a name="see-also"></a>请参阅

- [数据协定中的 XML 和 ADO.NET 类型](../../../../docs/framework/wcf/feature-details/xml-and-ado-net-types-in-data-contracts.md)
- [使用数据协定](../../../../docs/framework/wcf/feature-details/using-data-contracts.md)
- [可序列化类型](../../../../docs/framework/wcf/feature-details/serializable-types.md)
- [数据协定中的集合类型](../../../../docs/framework/wcf/feature-details/collection-types-in-data-contracts.md)
- [数据协定中的枚举类型](../../../../docs/framework/wcf/feature-details/enumeration-types-in-data-contracts.md)
