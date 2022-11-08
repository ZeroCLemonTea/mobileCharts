# 饼状图

饼状图，带选择框。

## 组件类名（className）

IPieChart

## 组件类 ID（classId）

idm.componet.mobileCharts.pieChart

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

#### 显示标题【isShowTitle】
用于设置title是否显示
- 标识：`是否显示标题【isShowTitle】`
- 默认值：`显示`
#### 标题【title】
用于设置title
- 标识：`标题【title】`
- 默认值：`组件标题`
#### 显示图标【showIcon】
- 标识：`showIcon`
- 默认值：`true`
#### 显示下拉【showSelect】
会在卡片右上角显示一个下拉，能和图表进行联动
- 标识：`showSelect`
- 默认值：`false`
#### 显示表格【showTable】
会在图表下方显示一个表格，开启此项需配置表格属性，见【表格属性设置】分组
- 标识：`showTable`
- 默认值：`false`
#### 下拉来源【columnsType】
下拉可以是静态配置的，或数据源
1. 数据源
2. 静态配置
- 标识：`columnsType`
- 默认值：`数据源`
#### 下拉数据源【columnsDataSource】

用于选择下拉的数据源，获取对应的数据。
入参：

```json
{
  "id": "数据源id"
}
```

接口返回格式示例：

```json
{
  "code": "200",
  "type": "success",
  "message": "操作成功",
  "metadata": null,
  "token": "",
  "data": [
     { "text": "2015年", "value": 2015, "isDefault": false }
    ...
  ]
}
```

- ⚠ <font color="#FF0000">**_注意事项：_**</font>
  <font color="#FF0000">如果格式有差异，请使用自定义函数，并返回正确格式的数据</font>
#### 下拉静态配置【columnsList】
##### 名称【text】
设置此项对应的文字描述
- 标识：`text`
- 默认值：``
##### 取值【value】
设置此项对应的取值
- 标识：`value`
- 默认值：``
##### 是否默认【isDefault】
设置此项是否为默认值
- 标识：`isDefault`
- 默认值：`false`
#### 图表数据源【chartDataSource】

用于选择图表的数据源。
入参：

```json
{
  "id": "数据源id",
  "selectedValue": "下拉选中项的值，下拉中的value字段",
  "selectedItem": "下拉选中项的对象"
}
```

接口返回格式示例。返回的字段名没有特定的要求，格式只需满足最外层的基本格式，data 里可以为其他格式：

```json
{
  "code": "200",
  "type": "success",
  "message": "操作成功",
  "metadata": null,
  "token": "",
  "data": [
    {
      "name": "集中培训", // 请确保字段名一致
      "value": 100, // 请确保字段名一致
      "displayValue": "100h",
      "ratio": "30%",
      "color": "red"
    },
    {
      "name": "论学学时",// 请确保字段名一致
      "value": 110,// 请确保字段名一致
      "displayValue": "100h",
      "ratio": "40%"
    },
    {
      "name": "在线学习",// 请确保字段名一致
      "value": 100,// 请确保字段名一致
      "displayValue": "100h",
      "ratio": "30%",
      "color": "green"
    }
  ]
}
```

- ⚠ <font color="#FF0000">**_注意事项：_**</font>
  <font color="#FF0000">如果格式有差异，请使用自定义函数，并返回正确格式的数据</font>

### 图表属性设置
#### 图表高度【chartHeight】
注意当组件外层或内层容器有高度时，图表和表格的高度会按照此值的比例缩放。可使用px、%、vh等单位
- 标识：`chartHeight`
- 默认值：`150px`
#### 图表标题【chartTitle】
会在图表上方显示
- 标识：`chartTitle`
- 默认值：``
#### 圆心标题【centerTitle】
标题是否显示在中心
- 标识：`centerTitle`
- 默认值：`false`
#### 显示图例【showLegend】
- 标识：`showLegend`
- 默认值：`false`
#### 标签取值【labelField】
为空不显示标签，请填写取值字段
- 标识：`labelField`
- 默认值：`value`
#### 图表样式【chartType】
1. 空心
2. 实心
- 标识：`chartType`
- 默认值：`空心`
#### 空隙大小【itemBorderWidth】
设置饼状图中各项间的间隔空隙，为0则没有间隔
- 标识：`itemBorderWidth`
- 默认值：`0`
#### 顶部边距【seriesTop】
图像离容器上边缘的距离，请填入数字或百分比，如果图表标题过大，则可适当增大此值
- 标识：`seriesTop`
- 默认值：`30`
#### 底部边距【seriesBottom】
图像离容器下边缘的距离，请填入数字或百分比，如果图例较多，则可适当增大此值
- 标识：`seriesBottom`
- 默认值：`20`
#### 颜色来源【colorType】
图表的颜色可以由数据源所返回的值决定，请使用hex或hex8等标准格式。也可以配置
1. 静态配置
2. 取值字段
- 标识：`colorType`
- 默认值：`静态配置`
#### 图表颜色【chartColors】
这里可设置图表各项的颜色，最终会按照数据源数组中的顺序加载对应的颜色
##### 颜色【color】
给每项数据配置颜色
#### 颜色字段【chartColorField】
可根据数据里的某字段控制图表颜色
- 标识：`chartColorField`
- 默认值：``
### 表格属性设置
#### 表格高度【tableHeight】
注意当组件外层或内层容器有高度时，表格和图表的高度会按照此值的比例缩放。可使用px、%、vh等单位
- 标识：`tableHeight`
- 默认值：`150px`
#### 字段设置【tableFields】
##### 取值字段【name】
设置此字段的取值
- 标识：`name`
- 默认值：``
##### 对齐方式【textAlign】
设置此扩展字段的对齐方式，不选择时默认中间居中，俩边分别居左或居右
1. 左
2. 中
3. 右
- 标识：`textAlign`
- 默认值：``
### 标题容器样式
#### 图标位置【titleIconPosition】
设置组件标题图标位置
1. 左边
2. 右边
- 标识：`titleIconPosition`
- 默认值：`右边`
#### 自定义图标【titleIcon】
自定义图标
- 标识：`titleIcon`
- 默认值：``
#### 图标颜色【titleIconColor】
图标颜色
- 标识：`titleIconColor`
- 默认值：``
#### 图标大小【titleIconSize】
- 标识：`titleIconSize`
- 默认值：`16`
#### 标题文字【titleFont】
标题文字样式设置
- 标识：`titleFont`
#### 下拉描述【selectBtnPlaceholder】
用于指定下拉无选中时的显示文字，同时也是选择器的标题
- 标识：`selectBtnPlaceholder`
- 默认值：`请选择`
#### 下拉文字【selectFont】
- 标识：`selectFont`
### 高级
#### 图表自定义函数【chartDataCustomFunction】
用于格式化数据源返回的数据，参数为
```json
{
  "urlObject":"url参数",
  "pageId":"页面id",
  "customParam":"自定义的参数",
  "resultData": "原始数据",
  "moduleObject":"组件信息"
}
#### 下拉自定义函数【columnsCustomFunction】
用于格式化数据源返回的数据，参数为
```json
{
  "urlObject":"url参数",
  "pageId":"页面id",
  "customParam":"自定义的参数",
  "resultData": "原始数据",
  "moduleObject":"组件信息"
}
```
#### 标签自定义函数【labelCustomFunction】
用于格式化图表中标签的数据，请返回字符串,参数为
```json
{
  "resultData":"当前项的原数据对象"
}
```
#### 下拉切换自定义函数【customSelectChange】
通过选择器更改下拉的值时调用，参数为
```json
{
  "urlObject":"url参数",
  "pageId":"页面id",
  "customParam":"自定义的参数",
  "moduleObject":"组件信息",
  "selectedValue":"选中的值",
  "selectedItem":"选中的项",
  "chartData":"组件数据"
}
```
#### 表格点击自定义函数【customTableRowCilck】
用于格式化数据源返回的数据，参数为
```json
{
  "urlObject":"url参数",
  "pageId":"页面id",
  "customParam":"自定义的参数",
  "moduleObject":"组件信息",
  "selectedValue":"选中的值",
  "selectedItem":"选中的项",
  "chartData":"组件数据",
  "record":"当前点击的行的数据"
}
```
