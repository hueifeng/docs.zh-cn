---
title: ISymUnmanagedWriter::CloseScope 方法
ms.date: 03/30/2017
api_name:
- ISymUnmanagedWriter.CloseScope
api_location:
- diasymreader.dll
api_type:
- COM
f1_keywords:
- ISymUnmanagedWriter::CloseScope
helpviewer_keywords:
- CloseScope method [.NET Framework debugging]
- ISymUnmanagedWriter::CloseScope method [.NET Framework debugging]
ms.assetid: 6dade525-7770-4cb4-bafd-4bb995ad0d87
topic_type:
- apiref
ms.openlocfilehash: 4d8790dc68bc063deed4c58ba0df8e9ea258b9d7
ms.sourcegitcommit: 27db07ffb26f76912feefba7b884313547410db5
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/19/2020
ms.locfileid: "83610075"
---
# <a name="isymunmanagedwriterclosescope-method"></a>ISymUnmanagedWriter::CloseScope 方法
关闭当前词法范围。  
  
## <a name="syntax"></a>语法  
  
```cpp  
HRESULT CloseScope(  
    [in] ULONG32 endOffset);  
```  
  
## <a name="parameters"></a>参数  
 `endOffset`  
 中词法范围中最后一个指令结束时点的方法开头的偏移量（以字节为单位）。  
  
## <a name="return-value"></a>返回值  
 如果该方法成功，则 S_OK;否则，E_FAIL 或其他一些错误代码。  
  
## <a name="remarks"></a>备注  
 范围结束后，不能再定义更多变量。  
  
 [ISymUnmanagedWriter：： OpenScope](../../../../docs/framework/unmanaged-api/diagnostics/isymunmanagedwriter-openscope-method.md)返回一个不透明的范围标识符，该标识符可与[ISymUnmanagedWriter：： SetScopeRange](isymunmanagedwriter-setscoperange-method.md)一起使用，以便稍后定义作用域的起始和结束偏移量。 在这种情况下，忽略传递到 `ISymUnmanagedWriter::OpenScope` 和 `ISymUnmanagedWriter::CloseScope` 的偏移量。 范围标识符只在当前方法中有效。  
  
## <a name="requirements"></a>要求  
 **标头：** CorSym，CorSym  
  
## <a name="see-also"></a>另请参阅

- [ISymUnmanagedWriter 接口](isymunmanagedwriter-interface.md)
