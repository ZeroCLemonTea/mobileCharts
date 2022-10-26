<template>
  <!--
    根目录规范(必须不能为空)：
    idm-ctrl：控件类型 drag_container：容器，drag_container_inlieblock：行内容器，idm_module：非容器的组件
    id：使用moduleObject.id，如果id不使用这个将会被idm-ctrl-id属性替换
    idm-ctrl-id：组件的id，这个必须不能为空
  -->
  <div
    idm-ctrl="idm_module"
    :id="moduleObject.id"
    :idm-ctrl-id="moduleObject.id"
    class="i-linebarChart-outer"
  >
    <div key="i-linebarChart-header" class="i-linebarChart-header" v-if="propData.isShowTitleBar">
      <div class="i-linebarChart-header-main">
        <div class="i-linebarChart-header-tit">
          <span v-if="propData.titleIconPosition === 'right'">{{ propData.title }}</span>
          <div class="i-linebarChart-header-tit-icon" v-if="propData.showIcon">
            <svg
              v-if="propData.titleIcon && propData.titleIcon.length > 0"
              class="idm_filed_svg_icon"
              aria-hidden="true"
            >
              <use :xlink:href="`#${propData.titleIcon && propData.titleIcon[0]}`" />
            </svg>
            <svg-icon v-else class="idm_filed_svg_icon" icon-class="application-icon" />
          </div>
          <span v-if="propData.titleIconPosition === 'left'">{{ propData.title }}</span>
        </div>
      </div>
    </div>
    <div key="i-linebarChart-content" class="i-linebarChart-content">
      <van-loading v-show="isLoading" :size="loadingSize" vertical>{{
        propData.loadingDescription || '加载中...'
      }}</van-loading>
      <div
        v-show="!isLoading && chartData && Object.keys(chartData).length > 0"
        class="i-linebarChart-content-wapper"
      >
        <div
          key="i-linebarChart-content-chart"
          :ref="`charts_container_${moduleObject.id}`"
          class="i-linebarChart-content-chart"
        />
      </div>
      <van-empty
        v-show="!isLoading && (!chartData || Object.keys(chartData).length == 0)"
        :image-size="emptyImageSize"
        :description="propData.emptyDescription || '暂无数据'"
      >
        <template #image>
          <van-image
            :src="
              IDM.url.getModuleAssetsWebPath(require('../assets/empty-default.png'), moduleObject)
            "
          />
        </template>
      </van-empty>
    </div>
    <div
      class="i-linebarChart-mask"
      v-if="
        moduleObject.env !== 'production' && !propData.chartDataSource
      "
    >
      <span>！未绑定数据源</span>
    </div>
  </div>
</template>

<script>
import * as echarts from 'echarts';
const devChartResult =
{
  nameList: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'],
  barValueList: [10, 52, 200, 334, 390, 330, 220],
  lineValueList: [20, 80, 40, 60, 50, 20, 10],
};
export default {
  name: 'ILineBarChart',
  data() {
    return {
      pageSizeWidth: 0,
      moduleObject: {},
      propData: this.$root.propData.compositeAttr || {
        // chartDataSource: '1',
        // showIcon: true,
        // title: '卡片标题',
        // titleIconPosition: 'left',
        // isShowTitleBar: false,
        // chartTitle: '标题',
        // // colorList: [{ offset: 0, color: {hex8: '#83bff6'} }],
        // gridTop: 65,
        // axisLabelWidth: '40',
        // chartLayout: 'vertical',
        // nameField: 'nameList',
        // minorYAxisUnit: 'ml',
        // yAxisUnit: 'kg',
        // showLegend: true,
        // boundaryGap: true,
        // legendXPosition: 'center',
        // legendYPosition: 'top',
        // gridBottom: 40,
        // seriesSet: [{
        //   type: 'bar',
        //   dataFiled: 'barValueList',
        //   showLabel: true,
        //   position: 'inside',
        //   name: '柱状',
        //   stack: '1'
        // }, {
        //   type: 'line',
        //   dataFiled: 'lineValueList',
        //   showSymbol: true,
        //   showLabel: true,
        //   position: 'out',
        //   isMinorYAxis: true,
        //   name: '折线',
        //   stack: '2'
        // }],
      },
      isLoading: false,
      chartData: {},
      chart: null
    };
  },
  props: {},
  computed: {
    emptyImageSize() {
      return this.getScale(this.pageSizeWidth) * (this.propData.emptyImageSize || 70);
    },
    loadingSize() {
      return this.getScale(this.pageSizeWidth) * (this.propData.loadingSize || 24);
    }
  },
  created() {
    this.moduleObject = this.$root.moduleObject;
    this.convertAttrToStyleObject();
    this.convertThemeListAttrToStyleObject();
  },
  mounted() {
    this.chart = this.$echarts.init(this.$refs[`charts_container_${this.moduleObject.id}`]);
    this.initData();
  },
  destroyed() {
    this.chart.dispose();
  },
  methods: {
    /**
     * 提供父级组件调用的刷新prop数据组件
     */
    propDataWatchHandle(propData) {
      this.propData = propData.compositeAttr || {};
      this.convertAttrToStyleObject();
      this.convertThemeListAttrToStyleObject();
      this.drawChart();
      this.$nextTick(() => {
        this.chart.resize();
      });
    },
    /**
     * 组件通信：接收消息的方法
     */
    receiveBroadcastMessage(messageObject) {
      switch (messageObject.type) {
        case 'websocket':
          if (this.propData.messageRefreshKey && messageObject.message) {
            const messageData =
              (typeof messageObject.message === 'string' && JSON.parse(messageObject.message)) ||
              messageObject.message;
            const arr = Array.isArray(this.propData.messageRefreshKey)
              ? this.propData.messageRefreshKey
              : [this.propData.messageRefreshKey];
            if (messageData.badgeType && arr.includes(messageData.badgeType)) {
              this.initData();
            }
          }
          break;
        case 'linkageReload':
          this.initData();
          break;
        case 'pageResize':
          this.pageSizeWidth = messageObject.message && messageObject.message.width
          this.convertAttrToStyleObject();
          this.drawChart();
          this.$nextTick(() => {
            this.chart.resize();
          });
          break;
        case 'userCustomFontSizeRatio':
          this.convertAttrToStyleObject();
          this.drawChart();
          this.$nextTick(() => {
            this.chart.resize();
          });
          break;
      }
    },
    /**
     * 适配页面
     */
    getScale(pageWidth) {
      const fontSizeRatio = IDM.user.getUserFontSizeRatio ? IDM.user.getUserFontSizeRatio() || 1 : 1;
      const base = this.propData.baseValue || 414;
      const ratio = this.propData.adaptationRatio || 1.2;
      const width = this.moduleObject.env === 'production' ? window.innerWidth : pageWidth || 414;
      return ((width / base - 1) * (ratio - 1) + 1) * fontSizeRatio;
    },
    drawChart() {
      this.chart.clear();
      const scale = this.getScale(this.pageSizeWidth)
      const nameList = this.getExpressData('data', this.propData.nameField, this.chartData);
      const xAxis = [
        {
          type: 'category',
          data: nameList,
          boundaryGap: this.propData.boundaryGap,
          axisTick: {
            show: false,
            alignWithLabel: true
          },
          // axisLine: {
          //   show: false
          // },
          axisLabel: {
            interval: 0,
            width: scale * this.propData.axisLabelWidth,
            overflow: 'break',
            rotate: this.propData.isXRotate ? 30 : 0,
            fontSize: scale * (this.propData.xAxisLabelFontSize || 12),
            color:
              this.propData.xAxisLabelFontColor && this.propData.xAxisLabelFontColor.hex8
                ? this.propData.xAxisLabelFontColor.hex8
                : '#666666',
            fontWeight: this.propData.xAxisLabelFontWeight || 'normal'
          }
        },
        // {
        //   type: 'category',
        //   data: valueList,
        //   show: this.propData.showExtraXAxis,
        //   axisTick: {
        //     show: false,
        //     alignWithLabel: true
        //   },
        //   axisLine: {
        //     show: false
        //   },
        //   axisLabel: {
        //     fontSize: scale * (this.propData.extraXAxisLabelFontSize || 12),
        //     color:
        //       this.propData.extraXAxisLabelFontColor && this.propData.extraXAxisLabelFontColor.hex8
        //         ? this.propData.extraXAxisLabelFontColor.hex8
        //         : '#666666'
        //   }
        // }
      ];
      const yAxis = [
        {
          type: 'value',
          axisTick: {
            show: false,
            alignWithLabel: true
          },
          axisLine: {
            show: false
          },
          axisLabel: {
            formatter: this.propData.yAxisUnit ? `{value} ${this.propData.yAxisUnit}` : '{value}',
            fontSize: scale * (this.propData.yAxisLabelFontSize || 12),
            color:
              this.propData.yAxisLabelFontColor && this.propData.yAxisLabelFontColor.hex8
                ? this.propData.yAxisLabelFontColor.hex8
                : '#666666'
          }
        },
        {
          type: 'value',
          axisTick: {
            show: false,
            alignWithLabel: true
          },
          axisLine: {
            show: false
          },
          axisLabel: {
            formatter: this.propData.minorYAxisUnit ? `{value} ${this.propData.minorYAxisUnit}` : '{value}',
            fontSize: scale * (this.propData.yAxisLabelFontSize || 12),
            color:
              this.propData.yAxisLabelFontColor && this.propData.yAxisLabelFontColor.hex8
                ? this.propData.yAxisLabelFontColor.hex8
                : '#666666'
          }
        }
      ];
      const option = {
        legend: {
          show: this.propData.showLegend,
          left: this.propData.legendXPosition,
          top: this.propData.legendYPosition == 'top' ? this.propData.chartTitle ? scale * ((this.propData.chartTitleFontSize || 16) + 6) : 'top' : 'bottom',
          textStyle: {
            color:
              this.propData.legendFontColor && this.propData.legendFontColor.hex8
                ? this.propData.legendFontColor.hex8
                : '#666666',
            fontSize: scale * (this.propData.legendFontSize || 14)
          }
        },
        title: {
          show: !!this.propData.chartTitle,
          text: this.propData.chartTitle,
          left: 'center',
          top: 0,
          textStyle: {
            color:
              this.propData.chartTitleFontColor && this.propData.chartTitleFontColor.hex8
                ? this.propData.chartTitleFontColor.hex8
                : '#666666',
            fontSize: scale * (this.propData.chartTitleFontSize || 16),
            fontWeight: this.propData.chartTitleFontWeight || 'bolder'
          }
        },
        emphasis: { disabled: true },
        grid: {
          left: this.propData.gridLeft,
          right: this.propData.gridRight,
          bottom: this.propData.gridBottom,
          top: this.propData.gridTop,
          containLabel: true
        },
        xAxis: this.propData.chartLayout == 'vertical' ? xAxis : yAxis,
        yAxis: this.propData.chartLayout == 'vertical' ? yAxis : xAxis,
        series: this.propData.seriesSet && this.propData.seriesSet.length > 0 ?
        this.propData.seriesSet.map(item => {
          if(item.type === 'bar') {
            return this.getBarSeries(item)
          } else if (item.type === 'line') {
            return this.getLineSeries(item)
          }
        }) : []
      };
      console.log(option)
      this.chart.setOption(option);
    },
    getBarSeries(propData){
      const scale = this.getScale(this.pageSizeWidth)
      const valueList = this.getExpressData('data', propData.dataFiled, this.chartData);
      const colorList =
        propData.colorGrad
        ? propData.fitColor1 && propData.fitColor1.hex8 && propData.fitColor2 && propData.fitColor2.hex8 ? [
            { offset: 0, color: propData.fitColor1.hex8 },
            { offset: 1, color: propData.fitColor2.hex8 }
          ] : [
            { offset: 0, color: '#188df0' },
            { offset: 1, color: '#188df0' }
          ]
        : propData.fitColor && propData.fitColor.hex8 ? [
            { offset: 1, color: propData.fitColor.hex8 }
          ] : [
            { offset: 1, color: '#188df0' }
          ];
      return {
        type: 'bar',
        stack: propData.stack,
        yAxisIndex: propData.isMinorYAxis && this.propData.chartLayout == 'vertical' ? 1 : 0,
        xAxisIndex: propData.isMinorYAxis && this.propData.chartLayout != 'vertical' ? 1 : 0,
        name: propData.name,
        barWidth: propData.barWidth,
        label: {
          show: propData.showLabel,
          position: propData.position == 'inside' ? this.propData.chartLayout == 'vertical' ? 'insideTop' : 'insideRight' : this.propData.chartLayout == 'vertical' ? 'top' : 'right',
          fontSize: scale * (propData.chartLabelFontSize || 12),
          color:
            propData.chartLabelFontColor && propData.chartLabelFontColor.hex8
              ? propData.chartLabelFontColor.hex8
              : '#666666'
        },
        itemStyle: {
          color: new echarts.graphic.LinearGradient(
            0,
            this.propData.chartLayout == 'vertical' ? 1 : 0,
            this.propData.chartLayout != 'vertical' ? 1 : 0,
            0,
            colorList
          )
        },
        data: valueList
      }
    },
    getLineSeries(propData){
      const scale = this.getScale(this.pageSizeWidth)
      const valueList = this.getExpressData('data', propData.dataFiled, this.chartData);
      const colorList =
        propData.colorGrad
        ? propData.fitColor1 && propData.fitColor1.hex8 && propData.fitColor2 && propData.fitColor2.hex8 ? [
            { offset: 0, color: propData.fitColor1.hex8 },
            { offset: 1, color: propData.fitColor2.hex8 }
          ] : []
        : propData.fitColor && propData.fitColor.hex8 ? [
            { offset: 1, color: propData.fitColor.hex8 }
          ] : [];
      return {
        type: 'line',
        stack: propData.stack,
        yAxisIndex: propData.isMinorYAxis && this.propData.chartLayout == 'vertical' ? 1 : 0,
        xAxisIndex: propData.isMinorYAxis && this.propData.chartLayout != 'vertical' ? 1 : 0,
        name: propData.name,
        smooth: propData.smooth,
        showSymbol: propData.showSymbol,
        symbol: propData.symbolType,
        symbolSize: propData.symbolType == 'emptyCircle' ? 8 : 6,
        lineStyle: {
          width: propData.lineWidth || 2,
          color: propData.color && propData.color.hex8 ? propData.color.hex8 : '#188df0'
        },
        label: {
          show: propData.showLabel,
          position: propData.position == 'inside' ? this.propData.chartLayout == 'vertical' ? 'insideTop' : 'insideRight' : this.propData.chartLayout == 'vertical' ? 'top' : 'right',
          fontSize: scale * (propData.chartLabelFontSize || 12),
          color:
            propData.chartLabelFontColor && propData.chartLabelFontColor.hex8
              ? propData.chartLabelFontColor.hex8
              : '#666666'
        },
        itemStyle: {
          color: propData.color && propData.color.hex8 ? propData.color.hex8 : '#188df0'
        },
        areaStyle: {
          color: new echarts.graphic.LinearGradient(
            0,
            this.propData.chartLayout == 'vertical' ? 1 : 0,
            this.propData.chartLayout != 'vertical' ? 1 : 0,
            0,
            colorList
          )
        },
        data: valueList
      }
    },
    /**
     * 通用的获取表达式匹配后的结果
     */
    getExpressData(dataName, dataFiled, resultData) {
      //给defaultValue设置dataFiled的值
      var _defaultVal = undefined;
      if (dataFiled) {
        var filedExp = dataFiled;
        filedExp = dataName + (filedExp.startsWiths('[') ? '' : '.') + filedExp;
        var dataObject = { IDM: window.IDM };
        dataObject[dataName] = resultData;
        _defaultVal = window.IDM.express.replace.call(this, '@[' + filedExp + ']', dataObject);
      }
      return _defaultVal;
    },
    customFormat(customFunction, resultData) {
      if (customFunction && customFunction[0] && customFunction[0].name) {
        resultData =
          window[customFunction[0].name] &&
          window[customFunction[0].name].call(this, {
            customParam: customFunction[0].param,
            moduleObject: this.moduleObject,
            resultData: resultData,
            urlObject: IDM.url.queryObject(),
            pageId:
              window.IDM.broadcast && window.IDM.broadcast.pageModule
                ? window.IDM.broadcast.pageModule.id
                : ''
          });
      }
      return resultData;
    },
    customFunctionHandle(customFunction, param = {}) {
      if (customFunction && customFunction[0] && customFunction[0].name) {
        window[customFunction[0].name] &&
          window[customFunction[0].name].call(this, {
            customParam: customFunction[0].param,
            moduleObject: this.moduleObject,
            urlObject: IDM.url.queryObject(),
            pageId:
              window.IDM.broadcast && window.IDM.broadcast.pageModule
                ? window.IDM.broadcast.pageModule.id
                : '',
            ...param
          });
      }
    },
    /**
     * 请求数据
     */
    initData() {
      if (!this.moduleObject.env || this.moduleObject.env == 'develop') {
        this.chartData = devChartResult;
        this.drawChart();
        this.$nextTick(() => {
          this.chart.resize();
        });
        return;
      }
      this.getChartData();
    },
    getChartData() {
      if (
        !this.moduleObject.env ||
        this.moduleObject.env == 'develop' ||
        !this.propData.chartDataSource
      ) {
        return false;
      }
      const url = `ctrl/dataSource/getDatas`;
      // const urlObject = IDM.url.queryObject();
      // const routerParams = this.moduleObject.routerId
      //   ? IDM.router.getParam(this.moduleObject.routerId)
      //   : {};
      this.isLoading = true;
      IDM.http
        .post(
          url,
          {
            // ...urlObject,
            // ...routerParams,
            id: this.propData.chartDataSource && this.propData.chartDataSource[0]?.id
          },
          {
            headers: {
              'Content-Type': 'application/json;charset=UTF-8'
            }
          }
        )
        .done(res => {
          this.isLoading = false;
          if (res.type === 'success') {
            const resultData = this.customFormat(this.propData.chartDataCustomFunction, res.data);
            this.chartData = resultData;
            this.drawChart();
            this.$nextTick(() => {
              this.chart.resize();
            });
          }
        })
        .error(err => {
          console.log(err);
          this.isLoading = false;
        });
    },
    /**
     * 把属性转换成样式对象
     */
    convertAttrToStyleObject() {
      const styleObject = {};
      const titleStyleObject = {};
      const innerCardStyleObject = {};
      const iconStyleObject = {};
      const emptyStyleObject = {};
      const loadingStyleObject = {};

      const scale = this.getScale(this.pageSizeWidth);
      styleObject['--i-linebarChart-scale'] = scale;

      if (this.propData.bgSize && this.propData.bgSize == 'custom') {
        styleObject['background-size'] =
          (this.propData.bgSizeWidth
            ? this.propData.bgSizeWidth.inputVal + this.propData.bgSizeWidth.selectVal
            : 'auto') +
          ' ' +
          (this.propData.bgSizeHeight
            ? this.propData.bgSizeHeight.inputVal + this.propData.bgSizeHeight.selectVal
            : 'auto');
      } else if (this.propData.bgSize) {
        styleObject['background-size'] = this.propData.bgSize;
      }
      if (this.propData.innerBgSize && this.propData.innerBgSize == 'custom') {
        innerCardStyleObject['background-size'] =
          (this.propData.innerBgSizeWidth
            ? this.propData.innerBgSizeWidth.inputVal + this.propData.innerBgSizeWidth.selectVal
            : 'auto') +
          ' ' +
          (this.propData.innerBgSizeHeight
            ? this.propData.innerBgSizeHeight.inputVal + this.propData.innerBgSizeHeight.selectVal
            : 'auto');
      } else if (this.propData.innerBgSize) {
        innerCardStyleObject['background-size'] = this.propData.innerBgSize;
      }

      if (this.propData.positionX && this.propData.positionX.inputVal) {
        styleObject['background-position-x'] =
          this.propData.positionX.inputVal + this.propData.positionX.selectVal;
      }
      if (this.propData.innerPositionX && this.propData.innerPositionX.inputVal) {
        innerCardStyleObject['background-position-x'] =
          this.propData.innerPositionX.inputVal + this.propData.innerPositionX.selectVal;
      }

      if (this.propData.positionY && this.propData.positionY.inputVal) {
        styleObject['background-position-y'] =
          this.propData.positionY.inputVal + this.propData.positionY.selectVal;
      }
      if (this.propData.innerPositionY && this.propData.innerPositionY.inputVal) {
        innerCardStyleObject['background-position-y'] =
          this.propData.innerPositionY.inputVal + this.propData.innerPositionY.selectVal;
      }
      for (const key in this.propData) {
        if (this.propData.hasOwnProperty.call(this.propData, key)) {
          const element = this.propData[key];
          if (!element && element !== false && element != 0) {
            continue;
          }
          switch (key) {
            case 'width':
            case 'height':
              styleObject[key] = element;
              break;
            case 'bgColor':
              if (element && element.hex8) {
                styleObject['background-color'] = IDM.hex8ToRgbaString(element.hex8);
              }
              break;
            case 'innerBgColor':
              if (element && element.hex8) {
                innerCardStyleObject['background-color'] = IDM.hex8ToRgbaString(element.hex8);
              }
              break;
            case 'boxShadow':
              styleObject['box-shadow'] = element;
              break;
            case 'box':
              if (element.marginTopVal) {
                styleObject['margin-top'] = `${element.marginTopVal}`;
              }
              if (element.marginRightVal) {
                styleObject['margin-right'] = `${element.marginRightVal}`;
              }
              if (element.marginBottomVal) {
                styleObject['margin-bottom'] = `${element.marginBottomVal}`;
              }
              if (element.marginLeftVal) {
                styleObject['margin-left'] = `${element.marginLeftVal}`;
              }
              if (element.paddingTopVal) {
                styleObject['padding-top'] = `${element.paddingTopVal}`;
              }
              if (element.paddingRightVal) {
                styleObject['padding-right'] = `${element.paddingRightVal}`;
              }
              if (element.paddingBottomVal) {
                styleObject['padding-bottom'] = `${element.paddingBottomVal}`;
              }
              if (element.paddingLeftVal) {
                styleObject['padding-left'] = `${element.paddingLeftVal}`;
              }
              break;
            case 'innerBox':
              if (element.marginTopVal) {
                innerCardStyleObject['margin-top'] = `${element.marginTopVal}`;
              }
              if (element.marginRightVal) {
                innerCardStyleObject['margin-right'] = `${element.marginRightVal}`;
              }
              if (element.marginBottomVal) {
                innerCardStyleObject['margin-bottom'] = `${element.marginBottomVal}`;
              }
              if (element.marginLeftVal) {
                innerCardStyleObject['margin-left'] = `${element.marginLeftVal}`;
              }
              if (element.paddingTopVal) {
                innerCardStyleObject['padding-top'] = `${element.paddingTopVal}`;
              }
              if (element.paddingRightVal) {
                innerCardStyleObject['padding-right'] = `${element.paddingRightVal}`;
              }
              if (element.paddingBottomVal) {
                innerCardStyleObject['padding-bottom'] = `${element.paddingBottomVal}`;
              }
              if (element.paddingLeftVal) {
                innerCardStyleObject['padding-left'] = `${element.paddingLeftVal}`;
              }
              break;
            case 'bgImgUrl':
              styleObject['background-image'] = `url(${window.IDM.url.getWebPath(element)})`;
              break;
            case 'innerBgImgUrl':
              innerCardStyleObject['background-image'] = `url(${window.IDM.url.getWebPath(
                element
              )})`;
              break;
            case 'positionX':
              //背景横向偏移
              break;
            case 'innerPositionX':
              //背景横向偏移
              break;
            case 'positionY':
              //背景纵向偏移
              break;
            case 'innerPositionY':
              //背景纵向偏移
              break;
            case 'bgRepeat':
              //平铺模式
              styleObject['background-repeat'] = element;
              break;
            case 'innerBgRepeat':
              //平铺模式
              innerCardStyleObject['background-repeat'] = element;
              break;
            case 'bgAttachment':
              //背景模式
              styleObject['background-attachment'] = element;
              break;
            case 'innerBgAttachment':
              //背景模式
              innerCardStyleObject['background-attachment'] = element;
              break;
            case 'border':
              if (element.border.top.width > 0) {
                styleObject['border-top-width'] =
                  element.border.top.width + element.border.top.widthUnit;
                styleObject['border-top-style'] = element.border.top.style;
                if (element.border.top.colors.hex8) {
                  styleObject['border-top-color'] = IDM.hex8ToRgbaString(
                    element.border.top.colors.hex8
                  );
                }
              }
              if (element.border.right.width > 0) {
                styleObject['border-right-width'] =
                  element.border.right.width + element.border.right.widthUnit;
                styleObject['border-right-style'] = element.border.right.style;
                if (element.border.right.colors.hex8) {
                  styleObject['border-right-color'] = IDM.hex8ToRgbaString(
                    element.border.right.colors.hex8
                  );
                }
              }
              if (element.border.bottom.width > 0) {
                styleObject['border-bottom-width'] =
                  element.border.bottom.width + element.border.bottom.widthUnit;
                styleObject['border-bottom-style'] = element.border.bottom.style;
                if (element.border.bottom.colors.hex8) {
                  styleObject['border-bottom-color'] = IDM.hex8ToRgbaString(
                    element.border.bottom.colors.hex8
                  );
                }
              }
              if (element.border.left.width > 0) {
                styleObject['border-left-width'] =
                  element.border.left.width + element.border.left.widthUnit;
                styleObject['border-left-style'] = element.border.left.style;
                if (element.border.left.colors.hex8) {
                  styleObject['border-left-color'] = IDM.hex8ToRgbaString(
                    element.border.left.colors.hex8
                  );
                }
              }

              styleObject['border-top-left-radius'] =
                element.radius.leftTop.radius + element.radius.leftTop.radiusUnit;
              styleObject['border-top-right-radius'] =
                element.radius.rightTop.radius + element.radius.rightTop.radiusUnit;
              styleObject['border-bottom-left-radius'] =
                element.radius.leftBottom.radius + element.radius.leftBottom.radiusUnit;
              styleObject['border-bottom-right-radius'] =
                element.radius.rightBottom.radius + element.radius.rightBottom.radiusUnit;
              break;
            case 'innerBorder':
              if (element.border.top.width > 0) {
                innerCardStyleObject['border-top-width'] =
                  element.border.top.width + element.border.top.widthUnit;
                innerCardStyleObject['border-top-style'] = element.border.top.style;
                if (element.border.top.colors.hex8) {
                  innerCardStyleObject['border-top-color'] = IDM.hex8ToRgbaString(
                    element.border.top.colors.hex8
                  );
                }
              }
              if (element.border.right.width > 0) {
                innerCardStyleObject['border-right-width'] =
                  element.border.right.width + element.border.right.widthUnit;
                innerCardStyleObject['border-right-style'] = element.border.right.style;
                if (element.border.right.colors.hex8) {
                  innerCardStyleObject['border-right-color'] = IDM.hex8ToRgbaString(
                    element.border.right.colors.hex8
                  );
                }
              }
              if (element.border.bottom.width > 0) {
                innerCardStyleObject['border-bottom-width'] =
                  element.border.bottom.width + element.border.bottom.widthUnit;
                innerCardStyleObject['border-bottom-style'] = element.border.bottom.style;
                if (element.border.bottom.colors.hex8) {
                  innerCardStyleObject['border-bottom-color'] = IDM.hex8ToRgbaString(
                    element.border.bottom.colors.hex8
                  );
                }
              }
              if (element.border.left.width > 0) {
                innerCardStyleObject['border-left-width'] =
                  element.border.left.width + element.border.left.widthUnit;
                innerCardStyleObject['border-left-style'] = element.border.left.style;
                if (element.border.left.colors.hex8) {
                  innerCardStyleObject['border-left-color'] = IDM.hex8ToRgbaString(
                    element.border.left.colors.hex8
                  );
                }
              }
              innerCardStyleObject['border-top-left-radius'] =
                element.radius.leftTop.radius + element.radius.leftTop.radiusUnit;
              innerCardStyleObject['border-top-right-radius'] =
                element.radius.rightTop.radius + element.radius.rightTop.radiusUnit;
              innerCardStyleObject['border-bottom-left-radius'] =
                element.radius.leftBottom.radius + element.radius.leftBottom.radiusUnit;
              innerCardStyleObject['border-bottom-right-radius'] =
                element.radius.rightBottom.radius + element.radius.rightBottom.radiusUnit;
              break;
            case 'titleFont':
              IDM.style.setFontStyle(titleStyleObject, element);
              titleStyleObject['font-size'] =
                element.fontSizeUnit === 'px'
                  ? scale * element.fontSize + element.fontSizeUnit
                  : element.fontSize + element.fontSizeUnit;
              break;
            case 'titleIconColor':
              iconStyleObject['color'] = IDM.hex8ToRgbaString(element.hex8);
              break;
            case 'titleIconSize':
              iconStyleObject['font-size'] = scale * element + 'px';
              break;
            case 'loadingFont':
              IDM.style.setFontStyle(loadingStyleObject, element);
              loadingStyleObject['font-size'] =
                element.fontSizeUnit === 'px'
                  ? scale * element.fontSize + element.fontSizeUnit
                  : element.fontSize + element.fontSizeUnit;
              break;
            case 'emptyFont':
              IDM.style.setFontStyle(emptyStyleObject, element);
              emptyStyleObject['font-size'] =
                element.fontSizeUnit === 'px'
                  ? scale * element.fontSize + element.fontSizeUnit
                  : element.fontSize + element.fontSizeUnit;
              break;
          }
        }
      }
      // 内层容器高度适配，若外层有高度，则内层随外层缩放。若外层没有高度，则由内层容器的子元素撑起。前提设置的有外层容器高度属性，若无此值不走此逻辑，取下面style中的预设
      // if (styleObject.height && styleObject.height != 'auto') {
      //   innerCardStyleObject.height = 0;
      // } else if (styleObject.height && styleObject.height == 'auto') {
      //   innerCardStyleObject.height = 'auto';
      // }
      window.IDM.setStyleToPageHead(this.moduleObject.id, styleObject);
      window.IDM.setStyleToPageHead(
        this.moduleObject.id + ' .i-linebarChart-content',
        innerCardStyleObject
      );
      window.IDM.setStyleToPageHead(
        this.moduleObject.id + ' .i-linebarChart-header-tit span',
        titleStyleObject
      );
      window.IDM.setStyleToPageHead(
        this.moduleObject.packageid +
          ' #' +
          this.moduleObject.id +
          ' .i-linebarChart-header-tit .i-linebarChart-header-tit-icon',
        iconStyleObject
      );
      window.IDM.setStyleToPageHead(this.moduleObject.id + ' .van-empty', emptyStyleObject);
      window.IDM.setStyleToPageHead(this.moduleObject.id + ' .van-loading', loadingStyleObject);
    },
    /**
     * 主题颜色
     */
    convertThemeListAttrToStyleObject() {
      const themeList = this.propData.themeList;
      if (!themeList) {
        return;
      }
      const themeNamePrefix =
        IDM.setting && IDM.setting.applications && IDM.setting.applications.themeNamePrefix
          ? IDM.setting.applications.themeNamePrefix
          : 'idm-theme-';
      for (var i = 0; i < themeList.length; i++) {
        const item = themeList[i];

        const titleSvgStyleObject = {
          color: item.mainColor ? IDM.hex8ToRgbaString(item.mainColor.hex8) : ''
        };

        IDM.setStyleToPageHead(
          '.' +
            themeNamePrefix +
            item.key +
            ' #' +
            (this.moduleObject.packageid || 'module_demo') +
            ' .i-linebarChart-header-tit .i-linebarChart-header-tit-icon',
          titleSvgStyleObject
        );
      }
    }
  }
};
</script>

<style scoped lang="scss">
$scale: var(--i-linebarChart-scale);
.i-linebarChart-outer {
  width: auto;
  box-sizing: border-box;
  padding: 10px;
  font-family: PingFangSC-Regular;
  color: #333333;
  background-color: #fff;
  position: relative;
  font-size: calc(14px * #{$scale});
  display: flex;
  flex-direction: column;
  align-items: stretch;
  height: 300px;
  overflow: hidden;
  *{
    font-family: inherit;
  }

  .i-linebarChart-header {
    display: flex;
    // height: calc(40px * #{ $scale });
    // line-height: calc(40px * #{ $scale });
    justify-content: space-between;
    align-items: center;

    .i-linebarChart-header-main {
      // width: 86%;
      flex-grow: 1;
      display: flex;
      align-items: center;

      .i-linebarChart-header-tit {
        font-family: PingFangSC-Regular;
        color: #333333;
        font-style: normal;
        text-decoration: none;
        font-size: calc(16px * #{$scale});
        max-width: 60%;
        display: flex;
        align-items: center;

        .i-linebarChart-header-tit-icon {
          margin-right: 5px;
          display: inline-block;
          flex: 1;
        }

        span {
          margin-right: 5px;
          width: 90%;
          overflow: hidden;
          flex: 8;
        }

        .idm_filed_svg_icon {
          font-size: 1em;
          width: 1em;
          height: 1em;
          fill: currentColor;
          vertical-align: -0.15em;
          outline: none;
        }
      }
    }

    .i-linebarChart-header-more {
      // opacity: 0.5;
      margin: 0 5px;
      position: relative;
    }
  }

  .i-linebarChart-content {
    /* border-radius: calc(10px * #{$scale});
    padding: calc(10px * #{$scale}) calc(14px * #{$scale}); */
    flex-grow: 1;
    flex-shrink: 1;
    height: 0;

    ::v-deep .van-loading {
      height: 100%;
      justify-content: center;
    }

    ::v-deep .van-empty {
      padding: 0;
      height: 100%;
    }

    ::v-deep .van-empty__description {
      font-size: inherit;
    }

    .i-linebarChart-content-wapper {
      height: 100%;
      display: flex;
      flex-direction: column;
      align-items: stretch;
      .i-linebarChart-content-chart {
        width: 100%;
        height: 100%;
        flex-grow: 1;
        flex-shrink: 1;
      }
    }
  }

  .i-linebarChart-mask {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 1;
    background: rgba(0, 0, 0, 0.3);
    display: flex;
    justify-content: center;
    align-items: center;
    span {
      padding: 6px 20px;
      color: #e6a23c;
      background: #fdf6ec;
      border: 1px solid #f5dab1;
      border-radius: 4px;
    }
  }
}
</style>
