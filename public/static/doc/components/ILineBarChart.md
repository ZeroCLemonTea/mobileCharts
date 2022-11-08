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
    "nameList": ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"], // x轴坐标刻度
    "barValueList": [10, 52, 200, 334, 390, 330, 220], // 第一系列的数据
    "lineValueList": [20, 80, 40, 60, 50, 20, 10], // 第二系列的数据
    ... // 其他系列的数据
  }
}
```

- ⚠ <font color="#FF0000">**_注意事项：_**</font>
  <font color="#FF0000">如果格式有差异，请使用自定义函数，并返回正确格式的数据</font>

#### 图表系列设置【seriesSet】
##### 图表类型【type】
用于选择不同类型的图表样式，对应维护不同属性
1. 柱状
2. 折线
- 标识：`type`
- 默认值：``
##### 图表名称【name】
当前图表的名称，若启用了图列，此值还会被图例引用
- 标识：`name`
- 默认值：``
##### 取值字段【dataFiled】
在数据源返回的对象中的取值字段
- 标识：`dataFiled`
- 默认值：``
##### 副Y轴【isMinorYAxis】
当前的图表的值是否由副Y轴展示。比如图表水平排布时，则会在图表右侧显示另一个y轴，当前系列的数值会以此轴为参考显示。
- 标识：`isMinorYAxis`
- 默认值：`false`
##### 柱条宽度【barWidth】
柱条的宽度，不设时自适应。可填百分比或数字
- 标识：`barWidth`
- 默认值：``
##### 折线宽度【lineWidth】
折线的宽度。填写数字
- 标识：`lineWidth`
- 默认值：`2`
##### 折线颜色【color】
- 标识：`color`
- 默认值：``
##### 显示节点【showSymbol】
是否显示折线上的数值节点
- 标识：`showSymbol`
- 默认值：`true`
##### 节点类型【symbolType】
1. 空心
2. 实心
- 标识：`showSymbol`
- 默认值：`空心`
##### 平滑曲线【smooth】
是否更改折线样式为平滑
- 标识：`smooth`
- 默认值：`false`
##### 堆叠标识【stack】
如果想要堆叠效果的柱状图，请维护同一个堆叠标识，如a、1等。相同的堆叠标识会在图表上堆叠显示，请注意堆叠顺序会按照【图标系列配置】中的顺序
- 标识：`stack`
- 默认值：``
##### 颜色渐变【colorGrad】
是否启用颜色渐变，需要维护俩个颜色，此模式下需要维护俩个颜色
- 标识：`colorGrad`
- 默认值：``
##### 填充颜色【fitColor】
柱状的颜色，或折线与x轴间的颜色
- 标识：`fitColor`
- 默认值：``
##### 起点颜色【fitColor1】
柱状的颜色，或折线与x轴间的颜色
- 标识：`fitColor1`
- 默认值：``
##### 终点颜色【fitColor2】
柱状的颜色，或折线与x轴间的颜色
- 标识：`fitColor2`
- 默认值：``
##### 数值标签【showLabel】
显示数值标签，用来展示具体的值
- 标识：`showLabel`
- 默认值：`false`
##### 标签位置【position】
标签会在外部的边缘显示，或在内部中间显示
1. 外部
2. 内部
- 标识：`position`
- 默认值：`外部`
##### 文字大小【chartLabelFontSize】
标签文字大小
- 标识：`chartLabelFontSize`
- 默认值：``
##### 文字颜色【chartLabelFontColor】
标签文字颜色
- 标识：`chartLabelFontColor`
- 默认值：``

### 图表属性设置
#### 柱状方向【chartLayout】
1. 水平
2. 竖直
- 标识：`chartLayout`
- 默认值：`水平`
#### X轴取值【nameField】
- 标识：`nameField`
- 默认值：`nameList`
#### X轴留白【boundaryGap】
会在X轴两侧填充空隙
- 标识：`boundaryGap`
- 默认值：`true`
#### 刻度宽度【axisLabelWidth】
X轴上的所有刻度标签的宽度，请填写数字
- 标识：`axisLabelWidth`
- 默认值：`40`
#### 刻度倾斜【isXRotate】
X轴上的所有刻度标签倾斜
- 标识：`isXRotate`
- 默认值：`false`
#### Y轴单位【yAxisUnit】
会在y轴上每个数值后显示
- 标识：`yAxisUnit`
- 默认值：``
#### 副Y轴单位【minorYAxisUnit】
会在y轴上每个数值后显示
- 标识：`minorYAxisUnit`
- 默认值：``
#### 水平位置【legendXPosition】
调整图例的水平位置
1. 左
2. 右
3. 中
- 标识：`legendXPosition`
- 默认值：``
#### 水平位置【legendYPosition】
调整图例的竖直位置
1. 上
2. 下
- 标识：`legendYPosition`
- 默认值：``
#### 顶部边距【gridTop】
图像离容器上边缘的距离，请填入数字或百分比，给每个柱状上的数字以及图表标题预留空间，可适当增大此值
- 标识：`gridTop`
- 默认值：`25`
#### 底部边距【gridBottom】
图像离容器下边缘的距离，请填入数字或百分比
- 标识：`gridBottom`
- 默认值：`0`
#### 左侧边距【gridLeft】
图像离容器左边缘的距离，请填入数字或百分比
- 标识：`gridLeft`
- 默认值：`0`
#### 右侧边距【gridRight】
图像离容器右边缘的距离，请填入数字或百分比
- 标识：`gridRight`
- 默认值：`0`
#### 图表标题【chartTitle】
会在图表上方显示
- 标识：`chartTitle`
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
```
