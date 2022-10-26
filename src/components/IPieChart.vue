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
    class="i-pieChart-outer"
  >
    <div
      key="i-pieChart-header"
      class="i-pieChart-header"
      v-if="propData.isShowTitleBar"
    >
      <div class="i-pieChart-header-main">
        <div class="i-pieChart-header-tit">
          <span v-if="propData.titleIconPosition === 'right'">{{ propData.title }}</span>
          <div class="i-pieChart-header-tit-icon" v-if="propData.showIcon">
            <svg
              v-if="propData.titleIcon && propData.titleIcon.length > 0"
              class="idm_filed_svg_icon"
              aria-hidden="true"
            >
              <use :xlink:href="`#${propData.titleIcon && propData.titleIcon[0]}`" />
            </svg>
            <svg-icon class="idm_filed_svg_icon" v-else icon-class="application-icon" />
          </div>
          <span v-if="propData.titleIconPosition === 'left'">{{ propData.title }}</span>
        </div>
      </div>
      <div
        :class="
          showPicker ? 'i-pieChart-header-select-active' : 'i-pieChart-header-select'
        "
        v-if="propData.showSelect"
        @click="showPicker = true"
      >
        {{ pickerSelectText }}
      </div>
    </div>
    <div key="i-pieChart-content" class="i-pieChart-content">
      <van-loading v-show="isLoading" :size="loadingSize" vertical>{{
        propData.loadingDescription || '加载中...'
      }}</van-loading>
      <div
        v-show="!isLoading && chartData && chartData.length > 0"
        class="i-pieChart-content-wapper"
      >
        <div
          key="i-pieChart-content-chart"
          :ref="`charts_container_${moduleObject.id}`"
          class="i-pieChart-content-chart"
        />
        <div
          key="i-pieChart-content-table"
          class="i-pieChart-content-table"
          v-if="
            propData.showTable &&
            propData.tableFields &&
            propData.tableFields[0] &&
            propData.tableFields[0].name
          "
        >
          <div
            class="i-pieChart-content-table-row"
            v-for="(item, index) in chartData"
            :key="index"
            @click.stop="tableRowCilck(item)"
          >
            <div class="col-legend" :style="{ 'background-color': colors[index] }" />
            <div
              class="col-text"
              v-for="(field, i) in propData.tableFields"
              :key="i"
              :style="{
                'text-align': field.textAlign
                  ? field.textAlign
                  : i == 0
                  ? 'left'
                  : i == propData.tableFields.length - 1
                  ? 'right'
                  : 'center'
              }"
            >
              {{ getExpressData('data', field.name, item) }}
            </div>
          </div>
        </div>
      </div>
      <van-empty
        v-show="!isLoading && (!chartData || chartData.length == 0)"
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
      class="i-pieChart-mask"
      v-if="
        moduleObject.env !== 'production' &&
        ((propData.isShowTitleBar &&
          propData.showSelect &&
          propData.columnsType == 'dataSource' &&
          !propData.columnsDataSource) ||
          !propData.chartDataSource)
      "
    >
      <span>！未绑定数据源</span>
    </div>
    <van-popup
      v-if="propData.isShowTitleBar && propData.showSelect"
      v-model="showPicker"
      round
      position="bottom"
    >
      <van-picker
        show-toolbar
        :title="propData.selectBtnPlaceholder || '请选择'"
        :columns="columnsText"
        @cancel="showPicker = false"
        @confirm="onConfirm"
      />
    </van-popup>
  </div>
</template>

<script>
const devChartResult = [
  {
    name: '集中培训',
    value: 100,
    displayValue: '100h',
    ratio: '30%',
    color: 'red'
  },
  {
    name: '论学学时',
    value: 110,
    displayValue: '100h',
    ratio: '40%'
  },
  {
    name: '在线学习',
    value: 100,
    displayValue: '100h',
    ratio: '30%',
    color: 'green'
  }
];
const devColumns = [
  { text: '2015年', value: '2015', isDefault: false },
  { text: '2016年', value: '2016', isDefault: false },
  { text: '2017年', value: '2017', isDefault: false },
  { text: '2018年', value: '2018', isDefault: false },
  { text: '2019年', value: '2019', isDefault: false },
  { text: '2020年', value: '2020', isDefault: false },
  { text: '2021年', value: '2021', isDefault: false },
  { text: '2022年', value: '2022', isDefault: true }
];
export default {
  name: 'IPieChart',
  data() {
    return {
      moduleObject: {},
      propData: this.$root.propData.compositeAttr || {
        // colorType: 'field',
        // chartColorField: '',
        // chartType: 'hollow',
        // columnsType: 'none',
        // showTable: true,
        // showLegend: false,
        // showSelect: false,
        // labelField: 'ratio',
        // chartDataSource: '1',
        // showIcon: true,
        // title: '卡片标题',
        // titleIconPosition: 'right',
        // isShowTitleBar: true,
        // moreBtn: false,
        // centerTitle: true,
        // pickerTitle: '选择年份',
        // itemBorderWidth: 0,
        // seriesTop: '30',
        // seriesBottom: '20',
        // chartTitle: '标题',
        // tableFields: [{ name: 'name' }, { name: 'displayValue', textAlign: 'left' }]
      },
      isLoading: false,
      showPicker: false,
      columns: [],
      pickerSelect: {},
      chartData: [],
      chart: null
    };
  },
  props: {},
  computed: {
    emptyImageSize() {
      return this.getScale() * (this.propData.emptyImageSize || 70);
    },
    loadingSize() {
      return this.getScale() * (this.propData.loadingSize || 24);
    },
    pickerSelectText() {
      if (this.pickerSelect && this.pickerSelect.value) {
        return this.pickerSelect.text;
      }
      return this.propData.selectBtnPlaceholder || '请选择';
    },
    columnsText() {
      if (!this.columns) return [];
      return this.columns.map(item => item.text);
    },
    colors() {
      const defaultColors = [
        '#FFBA00',
        '#FF8500',
        '#3296FA',
        '#0071C1',
        '#67C23B',
        '#2495C7',
        '#5B8FF9',
        '#FF8F00'
      ];
      let chartColors = [];
      if (this.propData.colorType == 'field') {
        chartColors = this.propData.chartColorField
          ? this.chartData.map(item =>
              this.getExpressData('data', this.propData.chartColorField, item)
            )
          : [];
      } else {
        chartColors =
          this.propData.chartColors?.map(item => item.color?.hex8).filter(item => !!item) || [];
      }
      return [...chartColors, ...defaultColors];
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
      this.getColumns();
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
              this.getChartData();
            }
          }
          break;
        case 'linkageReload':
          this.initData();
          break;
        case 'pageResize':
          this.convertAttrToStyleObject(messageObject.message);
          this.drawChart(messageObject.message);
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
    onConfirm(value, index) {
      this.pickerSelect = this.columns && this.columns[index];
      this.showPicker = false;
      if (this.moduleObject.env == 'production') {
        this.customFunctionHandle(this.propData.customSelectChange, {
          selectedValue: this.pickerSelect.value,
          selectedItem: this.pickerSelect,
          chartData: this.chartData
        });
        this.getChartData();
      }
    },
    tableRowCilck(item) {
      this.customFunctionHandle(this.propData.customTableRowCilck, {
        selectedValue: this.pickerSelect.value,
        selectedItem: this.pickerSelect,
        chartData: this.chartData,
        record: item
      });
    },
    labelFormatter({data}) {
      console.log(data)
      if (this.propData.labelCustomFunction && this.propData.labelCustomFunction[0] && this.propData.labelCustomFunction[0].name) {
        return this.customFormat(this.propData.labelCustomFunction, data)
      }
      console.log(this.getExpressData('data', this.propData.labelField, data))
      return this.getExpressData('data', this.propData.labelField, data)
    },
    drawChart(pageSize = {}) {
      this.chart.clear();
      const scale = this.getScale(pageSize.width)
      const legendData = this.chartData?.map(item => item.name);
      const option = {
        color: this.colors,
        legend: {
          show: this.propData.showLegend,
          left: 'center',
          bottom: 0,
          padding: 0,
          itemWidth: this.propData.legendItemSize || 14,
          itemHeight: this.propData.legendItemSize || 14,
          data: legendData,
          itemGap: this.propData.legendItemGap || 20,
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
          top: this.propData.centerTitle ? 'middle' : 0,
          textStyle: {
            color:
              this.propData.chartTitleFontColor && this.propData.chartTitleFontColor.hex8
                ? this.propData.chartTitleFontColor.hex8
                : '#666666',
            fontSize: scale * (this.propData.chartTitleFontSize || 16),
            fontWeight: this.propData.chartTitleFontWeight || 'bolder'
          }
        },
        series: [
          {
            type: 'pie',
            bottom: this.propData.showLegend ? this.propData.seriesBottom : 0,
            top: this.propData.chartTitle && !this.propData.centerTitle ? this.propData.seriesTop : 0,
            radius: this.propData.chartType == 'hollow' ? ['45%', '75%'] : [0, '75%'],
            itemStyle: {
              borderColor: '#fff',
              borderWidth: this.propData.itemBorderWidth || 0
            },
            label: {
              show: this.propData.labelField,
              formatter: this.labelFormatter,
              rich: {
                value: {
                  fontSize: scale * (this.propData.chartLabelFontSize || 12),
                  color:
                    this.propData.chartLabelFontColor && this.propData.chartLabelFontColor.hex8
                      ? this.propData.chartLabelFontColor.hex8
                      : '#666666'
                }
              }
            },
            labelLine: {
              show: this.propData.labelField,
              // smooth: 0.3,
              // length: 10,
              // length2: 15
            },
            emphasis: { disabled: true },
            data: this.chartData
          }
        ]
      };
      this.chart.setOption(option);
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
        this.columns = devColumns;
        const pickerSelect = this.columns.find(item => item.isDefault);
        this.pickerSelect = pickerSelect || {};
        this.drawChart();
        this.$nextTick(() => {
          this.chart.resize();
        });
        return;
      }
      this.getColumns();
    },
    getColumns() {
      if (!this.propData.isShowTitleBar || !this.propData.showSelect) {
        this.getChartData();
      } else if (this.propData.columnsType == 'static' && this.propData.columnsList) {
        if (this.propData.columnsList[0] && !this.propData.columnsList[0].value) {
          this.columns = devColumns;
        } else {
          this.columns = this.propData.columnsList;
        }
        const pickerSelect = this.columns.find(item => item.isDefault);
        this.pickerSelect = pickerSelect || {};
        this.getChartData();
      } else if (this.propData.columnsDataSource) {
        if (!this.moduleObject.env || this.moduleObject.env == 'develop') {
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
              id: this.propData.columnsDataSource && this.propData.columnsDataSource[0]?.id
            },
            {
              headers: {
                'Content-Type': 'application/json;charset=UTF-8'
              }
            }
          )
          .done(res => {
            if (res.type === 'success') {
              const resultData = this.customFormat(this.propData.columnsCustomFunction, res.data);
              this.columns = resultData;
              const pickerSelect = this.columns.find(item => item.isDefault);
              this.pickerSelect = pickerSelect || {};
              this.getChartData();
            } else {
              this.isLoading = false;
            }
          })
          .error(err => {
            console.log(err);
            this.isLoading = false;
          });
      }
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
      const selectParams =
        this.propData.isShowTitleBar && this.propData.showSelect
          ? {
              selectedValue: this.pickerSelect.value,
              selectedItem: this.pickerSelect
            }
          : {};
      IDM.http
        .post(
          url,
          {
            // ...urlObject,
            // ...routerParams,
            id: this.propData.chartDataSource && this.propData.chartDataSource[0]?.id,
            ...selectParams
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
    convertAttrToStyleObject(pageSize = {}) {
      const styleObject = {};
      const titleStyleObject = {};
      const innerCardStyleObject = {};
      const iconStyleObject = {};
      const emptyStyleObject = {};
      const loadingStyleObject = {};
      const chartStyleObject = {};
      const tableStyleObject = {};
      const tableIconStyleObject = {};
      const selectStyleObject = {};

      const scale = this.getScale(pageSize.width);
      styleObject['--i-pieChart-scale'] = scale;

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
            // case 'innerWidth':
            //   innerCardStyleObject['width'] = element;
            //   break;
            // case 'innerHeight':
            //   innerCardStyleObject['height'] = element;
            //   break;
            case 'chartHeight':
              chartStyleObject['height'] = element;
              break;
            case 'tableHeight':
              tableStyleObject['height'] = element;
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
            case 'tableBox':
              if (element.marginTopVal) {
                tableStyleObject['margin-top'] = `${element.marginTopVal}`;
              }
              if (element.marginRightVal) {
                tableStyleObject['margin-right'] = `${element.marginRightVal}`;
              }
              if (element.marginBottomVal) {
                tableStyleObject['margin-bottom'] = `${element.marginBottomVal}`;
              }
              if (element.marginLeftVal) {
                tableStyleObject['margin-left'] = `${element.marginLeftVal}`;
              }
              if (element.paddingTopVal) {
                tableStyleObject['padding-top'] = `${element.paddingTopVal}`;
              }
              if (element.paddingRightVal) {
                tableStyleObject['padding-right'] = `${element.paddingRightVal}`;
              }
              if (element.paddingBottomVal) {
                tableStyleObject['padding-bottom'] = `${element.paddingBottomVal}`;
              }
              if (element.paddingLeftVal) {
                tableStyleObject['padding-left'] = `${element.paddingLeftVal}`;
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
            case 'tableIconSize':
              tableIconStyleObject['width'] = scale * element + 'px';
              tableIconStyleObject['height'] = scale * element + 'px';
              break;
            case 'tableFont':
              IDM.style.setFontStyle(tableStyleObject, element);
              tableStyleObject['font-size'] =
                element.fontSizeUnit === 'px'
                  ? scale * element.fontSize + element.fontSizeUnit
                  : element.fontSize + element.fontSizeUnit;
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
            case 'selectFont':
              IDM.style.setFontStyle(selectStyleObject, element);
              selectStyleObject['font-size'] =
                element.fontSizeUnit === 'px'
                  ? scale * element.fontSize + element.fontSizeUnit
                  : element.fontSize + element.fontSizeUnit;
              break;
          }
        }
      }
      // 内层容器高度适配，若外层有高度，则内层随外层缩放。若外层没有高度，则由内层容器的子元素撑起。前提设置的有外层容器高度属性，若无此值不走此逻辑，取下面style中的预设
      if (styleObject.height && styleObject.height != 'auto') {
        innerCardStyleObject.height = 0;
      } else if (styleObject.height && styleObject.height == 'auto') {
        innerCardStyleObject.height = 'auto';
      }
      window.IDM.setStyleToPageHead(this.moduleObject.id, styleObject);
      window.IDM.setStyleToPageHead(
        this.moduleObject.id + ' .i-pieChart-content',
        innerCardStyleObject
      );
      window.IDM.setStyleToPageHead(
        this.moduleObject.id + ' .i-pieChart-header-tit span',
        titleStyleObject
      );
      window.IDM.setStyleToPageHead(
        this.moduleObject.packageid +
          ' #' +
          this.moduleObject.id +
          ' .i-pieChart-header-tit .i-pieChart-header-tit-icon',
        iconStyleObject
      );
      window.IDM.setStyleToPageHead(
        this.moduleObject.id + ' .i-pieChart-header .i-pieChart-header-select',
        selectStyleObject
      );
      window.IDM.setStyleToPageHead(
        this.moduleObject.id + ' .i-pieChart-header .i-pieChart-header-select-active',
        selectStyleObject
      );
      window.IDM.setStyleToPageHead(
        this.moduleObject.id + ' .i-pieChart-content .i-pieChart-content-chart',
        chartStyleObject
      );
      window.IDM.setStyleToPageHead(
        this.moduleObject.id + ' .i-pieChart-content .i-pieChart-content-table',
        tableStyleObject
      );
      window.IDM.setStyleToPageHead(
        this.moduleObject.id +
          ' .i-pieChart-content .i-pieChart-content-table .col-legend',
        tableIconStyleObject
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

        const selectStyleObject = {
          color: item.mainColor ? IDM.hex8ToRgbaString(item.mainColor.hex8) : ''
        };

        IDM.setStyleToPageHead(
          '.' +
            themeNamePrefix +
            item.key +
            ' #' +
            (this.moduleObject.packageid || 'module_demo') +
            ' .i-pieChart-header .i-pieChart-header-select-active',
          selectStyleObject
        );
        IDM.setStyleToPageHead(
          '.' +
            themeNamePrefix +
            item.key +
            ' #' +
            (this.moduleObject.packageid || 'module_demo') +
            ' .i-pieChart-header-tit .i-pieChart-header-tit-icon',
          titleSvgStyleObject
        );
      }
    }
  }
};
</script>

<style scoped lang="scss">
$scale: var(--i-pieChart-scale);
.i-pieChart-outer {
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
  height: auto;
  overflow: hidden;
  *{
    font-family: inherit;
  }

  .i-pieChart-header {
    display: flex;
    // height: calc(40px * #{ $scale });
    // line-height: calc(40px * #{ $scale });
    justify-content: space-between;
    align-items: center;

    .i-pieChart-header-main {
      // width: 86%;
      flex-grow: 1;
      display: flex;
      align-items: center;

      .i-pieChart-header-tit {
        font-family: PingFangSC-Regular;
        color: #333333;
        font-style: normal;
        text-decoration: none;
        font-size: calc(16px * #{$scale});
        max-width: 60%;
        display: flex;
        align-items: center;

        .i-pieChart-header-tit-icon {
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

    .i-pieChart-header-more,
    .i-pieChart-header-select,
    .i-pieChart-header-select-active {
      // opacity: 0.5;
      margin: 0 5px;
      position: relative;
    }

    .i-pieChart-header-select,
    .i-pieChart-header-select-active {
      padding-right: 13px;
    }
    .i-pieChart-header-select-active {
      color: red;
    }
    .i-pieChart-header-select::after {
      position: absolute;
      top: 50%;
      right: 0px;
      margin-top: -5px;
      border: 3px solid;
      border-color: transparent transparent currentColor currentColor;
      -webkit-transform: rotate(-45deg);
      transform: rotate(-45deg);
      opacity: 1;
      content: '';
    }
    .i-pieChart-header-select-active::after {
      position: absolute;
      top: 50%;
      right: 0px;
      margin-top: -1px;
      -webkit-transform: rotate(135deg);
      transform: rotate(135deg);
      border: 3px solid;
      border-color: transparent transparent currentColor currentColor;
      opacity: 1;
      content: '';
    }
  }

  .i-pieChart-content {
    /* border-radius: calc(10px * #{$scale});
    padding: calc(10px * #{$scale}) calc(14px * #{$scale}); */
    flex-grow: 1;
    flex-shrink: 1;
    height: auto;

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

    .i-pieChart-content-wapper {
      height: 100%;
      display: flex;
      flex-direction: column;
      align-items: stretch;
      .i-pieChart-content-chart {
        width: 100%;
        flex-grow: 1;
        flex-shrink: 1;
        height: 150px;
      }

      .i-pieChart-content-table {
        flex-grow: 1;
        flex-shrink: 1;
        overflow-y: auto;
        overflow-x: hidden;
        height: 150px;
        padding: 0 12%;
        .i-pieChart-content-table-row {
          display: flex;
          align-items: center;
          padding: 8px 0;
          line-height: 1;
          .col-legend {
            // width: calc(14px * #{$scale});
            // height: calc(14px * #{$scale});
            width: calc(14px * #{$scale});
            height: calc(14px * #{$scale});
            margin-right: calc(12px * #{$scale});
            border-radius: 3px;
          }
          .col-text {
            flex-grow: 1;
          }
        }
      }
    }
  }

  .i-pieChart-mask {
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
