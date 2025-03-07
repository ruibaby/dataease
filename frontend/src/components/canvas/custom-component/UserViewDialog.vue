<template>
  <de-container v-loading="$store.getters.loadingMap[$store.getters.currentPath]">
    <de-main-container v-show="showChartCanvas">
      <div id="chartCanvas" class="canvas-class" :style="customStyle">
        <div class="canvas-class" :style="commonStyle">
          <plugin-com
            v-if="chart.isPlugin"
            :component-name="chart.type + '-view'"
            :obj="{chart: mapChart || chart}"
            :chart="mapChart || chart"
            class="chart-class"
          />
          <chart-component v-else-if="!chart.type.includes('text') && chart.type !== 'label' && !chart.type.includes('table') && renderComponent() === 'echarts'" class="chart-class" :chart="mapChart || chart" />
          <chart-component-g2 v-else-if="!chart.type.includes('text') && chart.type !== 'label' && !chart.type.includes('table') && renderComponent() === 'antv'" class="chart-class" :chart="chart" />
          <chart-component-s2 v-else-if="chart.type === 'table-pivot' && renderComponent() === 'antv'" class="chart-class" :chart="chart" />
          <label-normal v-else-if="chart.type.includes('text')" :chart="chart" class="table-class" />
          <label-normal-text v-else-if="chart.type === 'label'" :chart="chart" class="table-class" />
        </div>
      </div>
    </de-main-container>
    <de-main-container v-show="!showChartCanvas">
      <table-normal :chart="chartTable" :show-summary="false" class="table-class" />
    </de-main-container>
  </de-container>
</template>

<script>

import ChartComponent from '@/views/chart/components/ChartComponent.vue'
import TableNormal from '@/views/chart/components/table/TableNormal'
import LabelNormal from '@/views/chart/components/normal/LabelNormal'
import DeMainContainer from '@/components/dataease/DeMainContainer'
import DeContainer from '@/components/dataease/DeContainer'
import DeAsideContainer from '@/components/dataease/DeAsideContainer'
import { mapState } from 'vuex'
import ChartComponentG2 from '@/views/chart/components/ChartComponentG2'
import PluginCom from '@/views/system/plugin/PluginCom'
import ChartComponentS2 from '@/views/chart/components/ChartComponentS2'
import LabelNormalText from '@/views/chart/components/normal/LabelNormalText'
import { exportDetails } from '@/api/panel/panel'
import html2canvas from 'html2canvasde'
import { hexColorToRGBA } from '@/views/chart/chart/util'
import { deepCopy, exportImg } from '@/components/canvas/utils/utils'
export default {
  name: 'UserViewDialog',
  components: { LabelNormalText, ChartComponentS2, ChartComponentG2, DeMainContainer, DeContainer, DeAsideContainer, ChartComponent, TableNormal, LabelNormal, PluginCom },
  props: {
    chart: {
      type: Object,
      default: null
    },
    chartTable: {
      type: Object,
      default: null
    },
    openType: {
      type: String,
      default: 'details'
    }
  },
  data() {
    return {
      refId: null,
      element: {},
      lastMapChart: null
    }
  },
  computed: {

    showChartCanvas() {
      return this.openType === 'enlarge'
    },
    isOnlyDetails() {
      return this.chart.type === 'table-normal' || this.chart.type === 'table-info'
    },
    customStyle() {
      let style = {
      }
      if (this.canvasStyleData.openCommonStyle) {
        if (this.canvasStyleData.panel.backgroundType === 'image' && this.canvasStyleData.panel.imageUrl) {
          style = {
            background: `url(${this.canvasStyleData.panel.imageUrl}) no-repeat`,
            ...style
          }
        } else if (this.canvasStyleData.panel.backgroundType === 'color') {
          style = {
            background: this.canvasStyleData.panel.color,
            ...style
          }
        }
      }
      if (!style.background) {
        style.background = '#FFFFFF'
      }
      return style
    },
    commonStyle() {
      const style = {}
      if (this.element && this.element.commonBackground) {
        style['padding'] = (this.element.commonBackground.innerPadding || 0) + 'px'
        style['border-radius'] = (this.element.commonBackground.borderRadius || 0) + 'px'
        if (this.element.commonBackground.enable) {
          if (this.element.commonBackground.backgroundType === 'innerImage') {
            const innerImage = this.element.commonBackground.innerImage.replace('svg', 'png')
            style['background'] = `url(${innerImage}) no-repeat`
          } else if (this.element.commonBackground.backgroundType === 'outerImage') {
            style['background'] = `url(${this.element.commonBackground.outerImage}) no-repeat`
          } else if (this.element.commonBackground.backgroundType === 'color') {
            style['background-color'] = hexColorToRGBA(this.element.commonBackground.color, this.element.commonBackground.alpha)
          }
        }
        style['overflow'] = 'hidden'
      }
      return style
    },
    ...mapState([
      'isClickComponent',
      'curComponent',
      'componentData',
      'canvasStyleData'
    ]),
    mapChart() {
      if (this.chart.type && (this.chart.type === 'map' || this.chart.type === 'buddle-map')) {
        const temp = JSON.parse(JSON.stringify(this.chart))
        let DetailAreaCode = null
        if (this.curComponent && this.curComponent.DetailAreaCode && this.curComponent.DetailAreaCode.length) {
          DetailAreaCode = this.curComponent.DetailAreaCode
        }
        if (!this.curComponent && this.lastMapChart) {
          return this.lastMapChart
        }
        if (this.curComponent && this.curComponent.options && this.curComponent.options.tabList && this.curComponent.options.tabList.length) {
          const tabList = JSON.parse(JSON.stringify(this.curComponent.options.tabList))
          tabList.forEach(tab => {
            if (tab.content &&
              tab.content.propValue &&
              tab.content.propValue.viewId &&
              tab.content.propValue.viewId === this.chart.id &&
              tab.content.DetailAreaCode &&
              tab.content.DetailAreaCode.length) {
              DetailAreaCode = tab.content.DetailAreaCode
            }
          })
        }
        const result = { ...temp, ...{ DetailAreaCode: DetailAreaCode }}
        this.setLastMapChart(result)
        return result
      }

      return null
    }
  },
  mounted() {
    this.element = deepCopy(this.curComponent)
  },
  methods: {
    exportExcel() {
      const _this = this
      if (this.isOnlyDetails) {
        _this.exportExcelDownload()
      } else {
        if (this.showChartCanvas) {
          html2canvas(document.getElementById('chartCanvas')).then(canvas => {
            const snapshot = canvas.toDataURL('image/jpeg', 1)
            _this.exportExcelDownload(snapshot, canvas.width, canvas.height)
          })
        } else {
          _this.exportExcelDownload()
        }
      }
    },
    exportViewImg() {
      exportImg(this.chart.name)
    },
    setLastMapChart(data) {
      this.lastMapChart = JSON.parse(JSON.stringify(data))
    },
    exportExcelDownload(snapshot, width, height) {
      const excelHeader = JSON.parse(JSON.stringify(this.chart.data.fields)).map(item => item.name)
      const excelHeaderKeys = JSON.parse(JSON.stringify(this.chart.data.fields)).map(item => item.dataeaseName)
      const excelData = JSON.parse(JSON.stringify(this.chart.data.tableRow)).map(item => excelHeaderKeys.map(i => item[i]))
      const excelName = this.chart.name
      const request = {
        viewName: excelName,
        header: excelHeader,
        details: excelData,
        snapshot: snapshot,
        snapshotWidth: width,
        snapshotHeight: height
      }
      exportDetails(request).then((res) => {
        const blob = new Blob([res], { type: 'application/vnd.ms-excel' })
        const link = document.createElement('a')
        link.style.display = 'none'
        link.href = URL.createObjectURL(blob)
        link.download = excelName + '.xls' // 下载的文件名
        document.body.appendChild(link)
        link.click()
        document.body.removeChild(link)
      })
    },

    renderComponent() {
      return this.chart.render
    }
  }
}
</script>

<style lang="scss" scoped>
  .ms-aside-container {
    height: 70vh;
    min-width: 400px;
    max-width: 400px;
    padding: 0 0;
  }
  .ms-main-container {
    height: 70vh;
    border: 1px solid #E6E6E6;
  }
  .chart-class{
    height: 100%;
  }
  .table-class{
    height: 100%;
  }
  .canvas-class{
    width: 100%;
    height: 100%;
    background-size: 100% 100% !important;
  }
</style>
