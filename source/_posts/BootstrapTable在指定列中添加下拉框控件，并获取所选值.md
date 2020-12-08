---
title: BootstrapTable在指定列中添加下拉框控件，并获取所选值
toc: true
english_url: bootstraptable_select_value
date: 2020-12-04 16:05:03
tags: bootstrapTable
categories: bootstrap
excerpt: 最近在使用Bootstrap table ，有一个在某一列添加一个下拉列表，并且通过 “getAllSelections”方法获取所选行的需求，在实现这个功能的时，走了一些弯路，遇到了一些坑。所以今天总结出来，既是自己的学习，也分享给大家，希望能够有些帮助。
---

# 背景 

 最近在使用Bootstrap table ，有一个在某一列添加一个下拉列表，并且通过 “getAllSelections”方法获取所选行的需求，在实现这个功能的时，走了一些弯路，遇到了一些坑。所以今天总结出来，既是自己的学习，也分享给大家，希望能够有些帮助。

# 如何解决

添加这个下拉列表有以下两种方法：

- 利用Column options 中的 formatter 将数据转换成下拉列表的形式
- 使用bootstrap-table拓展中的editable插件

  这次主要介绍第一种，基本的思路为：首先通过 bootstrap-table 的Column 配置项中的formatter，将获取到的数据转换为包含数据的 select 控件。然后根据用户选择项更新对应单元格数据，最后通过getallselection方法获取所选行数据。

> formatter，其配置项为function，有三个参数：(value,row,index)

```
formatter: setSelect
function setSelect(value, row, index) 
{
    var strHtml = "";
    if (value == "Item 1") 
    {
        strHtml = "<select class='ss'><option value='Item 1' selected='selected'>Item 1</option><option value='Item 2'>Item 2</option></select>";
     } 
     else 
     {
        strHtml = "<select class='ss'><option value='Item 1' >Item 1</option><option value='Item 2' selected='selected'>Item 2</option></select>";
     }
     return strHtml;
}
```

到这里，下拉列表已经可以显示出来了，但是如果直接使用 getallselection 方法获取所选内容会有问题：获取到的数据是默认表格初始化加载的内容，并不是重新选择的内容。

  bootstrap-table是一个jquery插件，直接在html上面修改是获取不到的，要修改需要通过它自己的方法。bootstrap-table 在Methods 中提供了一个updateCell的方法。

> updateCell ，包含了三个参数（index,field,value）,在某一行的某一列更新value。

```
$('#table').bootstrapTable('updateCell', {
    index: indexSelected,
    field: 'name',
    value: valueSelected
  })
```

# events

 完成了下拉列表的显示，有了更新单元格值的方法，还需要做的是为下拉列表的选择绑定事件，实现下拉列表选择->改变单元格值。

  我们可以在select元素上绑定onchange事件，或者使用JQuery的change 事件。

```
$(".ss").change(function() {
  var indexSelected = $(this).parent().parent()[0].rowIndex - 1;
  var valueSelected = $(this).children('option:selected').val();
  $('#table').bootstrapTable('updateCell', {
    index: indexSelected,
    field: 'name',
    value: valueSelected
  })
});
```

 但是经过测试，发现$(“.ss”).change（）只是在页面加载后第一次选择可以触发，后来在bootstrap-table的文档中发现了events项，可以监听单元格事件，和formatter 配合着用。

```
events: {'change .ss': function (e, value, row, index) {}};
*//value是当前单元格的值，row是当前行，index是当前行的索引值*
```

- change 传递的是jQuery事件
- .ss 是jQuery的类选择器



完成代码：

```html
<button id="getallselection" class="btn btn-primary" type="button">getallselection</button>
<br>
<br>
<table id="table"></table>
```

```javascript
$('#table').bootstrapTable({
  editable: true,
  columns: [{
    field: 'if',
    title: 'check',
    checkbox: true
  }, {
    field: 'id',
    title: 'Item ID'
  }, {
    field: 'name',
    title: 'Item Name',
    formatter: setSelect,
    events: {'change .ss': function (e, value, row, index) {
                    var valueSelected = $(this).children('option:selected').val();
                        $('#table').bootstrapTable('updateCell', {
                            index: index,
                            field: 'name',
                            value: valueSelected
                        })
                    }
                }
  }],
  data: [{
    id: 1,
    name: 'Item 1',
  }, {
    id: 2,
    name: 'Item 2',
  }]
});

function setSelect(value, row, index) 
{
		var strHtml = "";
		if(value == "Item 1")
    {
    		strHtml = "<select class='ss'><option value='Item 1' selected='selected'>Item 1</option><option value='Item 2'>Item 2</option></select>";
    }
    else
    {
    		strHtml = "<select class='ss'><option value='Item 1' >Item 1</option><option value='Item 2' selected='selected'>Item 2</option></select>";
    }
  	return strHtml;
}



$("#getallselection").click(function() {
  var result = $('#table').bootstrapTable('getAllSelections');
  alert(result[0].name);
});

```

文章来源：[【Bootstrap Table】在指定列中添加下拉框控件，并获取所选值_Just do it!-CSDN博客](https://blog.csdn.net/u013201439/article/details/76290661)

