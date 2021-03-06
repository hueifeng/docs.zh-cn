---
title: 如何：为安全会话创建安全上下文令牌
ms.date: 03/30/2017
dev_langs:
- csharp
- vb
ms.assetid: 640676b6-c75a-4ff7-aea4-b1a1524d71b2
ms.openlocfilehash: 4e91580035d4de23ae90cd0d59a08f321ae70a1c
ms.sourcegitcommit: 927b7ea6b2ea5a440c8f23e3e66503152eb85591
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/16/2020
ms.locfileid: "81464139"
---
# <a name="how-to-create-a-security-context-token-for-a-secure-session"></a>如何：为安全会话创建安全上下文令牌
通过在安全会话中使用有状态安全上下文令牌 (SCT)，可以使该会话避免因为重新使用服务而受到影响。 例如，如果在安全会话中使用了无状态 SCT 并且 Internet 信息服务 (IIS) 被重置，则与该服务相关联的会话数据将丢失。 这些会话数据包括一个 SCT 令牌缓存。 因此，当客户端下一次向该服务发送无状态 SCT 时，将返回错误，这是因为无法检索到与该 SCT 相关联的密钥。 但是，如果使用有状态 SCT，则与该 SCT 相关联的密钥将包含在该 SCT 中。 由于密钥包含在 SCT 中并因而包含在消息中，因此安全会话不会因为重新使用服务而受到影响。 默认情况下，Windows 通信基础 （WCF） 在安全会话中使用无状态 ST。 本主题详细介绍如何在安全会话中使用有状态 SCT。  
  
> [!NOTE]
> 如果安全会话涉及到派生自 <xref:System.ServiceModel.Channels.IDuplexChannel> 的协定，则无法在该安全会话中使用有状态 SCT。  
  
> [!NOTE]
> 对于在安全会话中使用有状态 SCT 的应用程序，服务的线程标识必须是具有关联用户配置文件的用户帐户。 如果服务在不具有用户配置文件的帐户下运行（如 `Local Service`），则可能引发异常。  
  
> [!NOTE]
> 当需要在 Windows XP 上进行模拟时，请不要在安全会话中使用有状态 SCT。 如果在模拟时使用有状态 SCT，则会引发 <xref:System.InvalidOperationException>。 有关详细信息，请参阅[不支持的方案](../../../../docs/framework/wcf/feature-details/unsupported-scenarios.md)。  
  
### <a name="to-use-stateful-scts-in-a-secure-session"></a>在安全会话中使用有状态 SCT  
  
- 创建一个自定义绑定，该绑定指定由使用有状态 SCT 的安全会话来保护 SOAP 消息。  
  
    1. 通过将[\<自定义绑定>](../../../../docs/framework/configure-apps/file-schema/wcf/custombinding.md)添加到服务的配置文件，定义自定义绑定。  
  
        ```xml  
        <customBinding>  
        </customBinding>
        ```  
  
    2. 将[\<绑定>](../../configure-apps/file-schema/wcf/bindings.md)子元素添加到[\<自定义绑定>](../../../../docs/framework/configure-apps/file-schema/wcf/custombinding.md)。  
  
         通过在配置文件中将 `name` 属性设置为一个唯一的名称，指定一个绑定名称。  
  
        ```xml  
        <binding name="StatefulSCTSecureSession">  
        </binding>
        ```  
  
    3. 通过将[\<安全>](../../../../docs/framework/configure-apps/file-schema/wcf/security-of-custombinding.md)子元素添加到[\<自定义绑定>，](../../../../docs/framework/configure-apps/file-schema/wcf/custombinding.md)为发送到和从此服务发送的邮件指定身份验证模式。  
  
         通过将 `authenticationMode` 属性设置为 `SecureConversation`，指定使用安全会话。 通过将 `requireSecurityContextCancellation` 属性设置为 `false`，指定使用有状态 SCT。  
  
        ```xml  
        <security authenticationMode="SecureConversation"  
                  requireSecurityContextCancellation="false">
        </security>
        ```  
  
    4. 通过将[\<安全会话>](../../../../docs/framework/configure-apps/file-schema/wcf/secureconversationbootstrap.md)子元素添加到[\<安全>，](../../../../docs/framework/configure-apps/file-schema/wcf/security-of-custombinding.md)指定在建立安全会话时客户端的身份验证方式。  
  
         通过设置 `authenticationMode` 属性，指定如何对客户端进行身份验证。  
  
        ```xml  
        <secureConversationBootstrap authenticationMode="UserNameForCertificate" />  
        ```  
  
    5. 通过添加编码元素（如[\<文本Messageencode>）](../../../../docs/framework/configure-apps/file-schema/wcf/textmessageencoding.md)指定消息编码。  
  
        ```xml  
        <textMessageEncoding />  
        ```  
  
    6. 通过添加传输元素（如[\<httpTransport>](../../../../docs/framework/configure-apps/file-schema/wcf/httptransport.md)）指定传输。  
  
        ```xml  
        <httpTransport />  
        ```  
  
     下面的代码示例使用配置来指定一个自定义绑定，消息可以将该绑定与安全会话中的有状态 SCT 结合使用。  
  
    ```xml  
    <customBinding>  
      <binding name="StatefulSCTSecureSession">  
        <security authenticationMode="SecureConversation"  
                  requireSecurityContextCancellation="false">  
          <secureConversationBootstrap authenticationMode="UserNameForCertificate" />  
        </security>  
        <textMessageEncoding />  
        <httpTransport />  
      </binding>  
    </customBinding>  
    ```  
  
## <a name="example"></a>示例  
 下面的代码示例创建一个自定义绑定，该绑定使用 <xref:System.ServiceModel.Configuration.AuthenticationMode.MutualCertificate> 身份验证模式启动安全会话。  
  
 [!code-csharp[c_CreateStatefulSCT#2](../../../../samples/snippets/csharp/VS_Snippets_CFX/c_createstatefulsct/cs/secureservice.cs#2)]
 [!code-vb[c_CreateStatefulSCT#2](../../../../samples/snippets/visualbasic/VS_Snippets_CFX/c_createstatefulsct/vb/secureservice.vb#2)]  
  
 当 Windows 身份验证与有状态 SCT 结合使用时，WCF 不会<xref:System.ServiceModel.ServiceSecurityContext.WindowsIdentity%2A>使用实际调用方的标识填充属性，而是将属性设置为匿名。 由于 WCF 安全必须为来自传入 SCT 的每个请求重新创建服务安全上下文的内容，因此服务器不会跟踪内存中的安全会话。 因为不可能将 <xref:System.Security.Principal.WindowsIdentity> 实例序列化为 SCT，所以 <xref:System.ServiceModel.ServiceSecurityContext.WindowsIdentity%2A> 属性返回一个匿名标识。  
  
 下面的配置演示这一行为。  
  
```xml  
<customBinding>  
  <binding name="Cancellation">  
       <textMessageEncoding />  
        <security
            requireSecurityContextCancellation="false">  
              <secureConversationBootstrap />  
        </security>  
    <httpTransport />  
  </binding>  
</customBinding>  
```  
  
## <a name="see-also"></a>请参阅

- [\<自定义绑定>](../../../../docs/framework/configure-apps/file-schema/wcf/custombinding.md)
