# 柱状图

折线和柱状图。

## 组件类名（className）

ILineBarChart

## 组件类 ID（classId）

idm.componet.mobileCharts.lineBarChart

## 组件开发语言（comLangue）

vue

## 组件类型（comType）

common

## 所在代码包版本

mobileCharts@1.0.0

## 组件属性

此章节主要介绍该组件的每个属性的含义以及如何使用说明

### 唯一标识【ctrlId】

只读属性，不可修改，作为每个组件实例的一个唯一标识

- 标识：`ctrlId`
- 默认值：`@[packageid]`

### 基本属性

<font color="#CCCCCC">此章节主要用于存放设置组件所需要的一些基本信息的属性，以达到组件具备使用的基础条件</font>

#### 图表数据源【chartDataSource】

用于选择图表的数据源。
入参：

```json
{
  "id": "数据源id"
}
```

接口返回格式示例。返回的字段名没有特定的要求：

```json
{
  "code": "200",
  "type": "success",
  "message": "操作成功",
  "metadata": null,
  "token": "",
  "data": {
    "nameList": ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
    "barValueList": [10, 52, 200, 334, 390, 330, 220],
    "lineValueList": [20, 80, 40, 60, 50, 20, 10],
  }
}
```

- ⚠ <font color="#FF0000">**_注意事项：_**</font>
  <font color="#FF0000">如果格式有差异，请使用自定义函数，并返回正确格式的数据</font>
