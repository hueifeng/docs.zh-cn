---
title: 如何：使用模板来设置使用 GridView 的 ListView 的样式
ms.date: 03/30/2017
helpviewer_keywords:
- ListView controls [WPF], styling
ms.assetid: 94bf964b-96c8-4bdf-a0c3-f5271b7cb565
ms.openlocfilehash: 1caa652c4a2a3a7d0a8d40fe703df7a3e8038c9b
ms.sourcegitcommit: 9b552addadfb57fab0b9e7852ed4f1f1b8a42f8e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/23/2019
ms.locfileid: "61699118"
---
# <a name="how-to-use-templates-to-style-a-listview-that-uses-gridview"></a>如何：使用模板来设置使用 GridView 的 ListView 的样式
此示例演示如何使用<xref:System.Windows.DataTemplate>并<xref:System.Windows.Style>对象来指定的外观<xref:System.Windows.Controls.ListView>控件，它使用<xref:System.Windows.Controls.GridView>视图模式。  
  
## <a name="example"></a>示例  
 下面的示例演示<xref:System.Windows.Style>并<xref:System.Windows.DataTemplate>对象的自定义的列标题的外观<xref:System.Windows.Controls.GridViewColumn>。  
  
 [!code-xaml[ListViewTemplate#GridViewHeaderStyle](~/samples/snippets/csharp/VS_Snippets_Wpf/ListViewTemplate/CS/window1.xaml#gridviewheaderstyle)]  
  
 [!code-xaml[ListViewTemplate#GridViewHeaderTemplate](~/samples/snippets/csharp/VS_Snippets_Wpf/ListViewTemplate/CS/window1.xaml#gridviewheadertemplate)]  
  
 下面的示例演示如何使用这些<xref:System.Windows.Style>并<xref:System.Windows.DataTemplate>对象来设置<xref:System.Windows.Controls.GridViewColumn.HeaderContainerStyle%2A>并<xref:System.Windows.Controls.GridViewColumn.HeaderTemplate%2A>的属性<xref:System.Windows.Controls.GridViewColumn>。 <xref:System.Windows.Controls.GridViewColumn.DisplayMemberBinding%2A>属性定义的列单元格的内容。  
  
 [!code-xaml[ListViewTemplate#GridViewColumnTemplate](~/samples/snippets/csharp/VS_Snippets_Wpf/ListViewTemplate/CS/window1.xaml#gridviewcolumntemplate)]  
  
 <xref:System.Windows.Controls.GridViewColumn.HeaderContainerStyle%2A>并<xref:System.Windows.Controls.GridViewColumn.HeaderTemplate%2A>只有两种可用于自定义列标题外观的多个属性<xref:System.Windows.Controls.GridView>控件。 有关详细信息，请参阅 [GridView 列标题的样式和模板概述](gridview-column-header-styles-and-templates-overview.md)。  
  
 下面的示例演示如何定义<xref:System.Windows.DataTemplate>的自定义中的单元格的外观<xref:System.Windows.Controls.GridViewColumn>。  
  
 [!code-xaml[ListViewTemplate#GridViewCellTemplate](~/samples/snippets/csharp/VS_Snippets_Wpf/ListViewTemplate/CS/window1.xaml#gridviewcelltemplate)]  
  
 下面的示例演示如何使用此<xref:System.Windows.DataTemplate>定义的内容<xref:System.Windows.Controls.GridViewColumn>单元格。 而不是使用此模板<xref:System.Windows.Controls.GridViewColumn.DisplayMemberBinding%2A>在以前显示的属性<xref:System.Windows.Controls.GridViewColumn>示例。  
  
 [!code-xaml[ListViewTemplate#CellTemplateProperty](~/samples/snippets/csharp/VS_Snippets_Wpf/ListViewTemplate/CS/window1.xaml#celltemplateproperty)]  
  
## <a name="see-also"></a>请参阅

- <xref:System.Windows.Controls.ListView>
- <xref:System.Windows.Controls.GridView>
- [GridView 概述](gridview-overview.md)
- [帮助主题](listview-how-to-topics.md)
- [ListView 概述](listview-overview.md)
