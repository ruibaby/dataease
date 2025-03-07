<template>
  <div class="bar-main">
    <input id="input" ref="files" type="file" accept="image/*" hidden @click="e => {e.target.value = '';}" @change="handleFileChange">
    <div v-if="linkageAreaShow" style="margin-right: -1px;width: 20px">
      <el-checkbox v-model="linkageInfo.linkageActive" size="medium" />
      <linkage-field v-if="linkageInfo.linkageActive" :element="element" />
    </div>
    <div v-if="batchOptAreaShow" style="margin-right: -1px;width: 20px;z-index: 5">
      <el-checkbox size="medium" @change="batchOptChange" />
    </div>
    <div v-if="normalAreaShow">
      <setting-menu v-if="activeModel==='edit'" style="float: right;height: 24px!important;" @amRemoveItem="amRemoveItem" @linkJumpSet="linkJumpSet" @boardSet="boardSet">
        <span slot="icon" :title="$t('panel.setting')">
          <i class="icon iconfont icon-shezhi" style="margin-top:2px" />
        </span>
      </setting-menu>
      <span :title="$t('panel.edit')">
        <i v-if="activeModel==='edit'&&curComponent&&editFilter.includes(curComponent.type)" class="icon iconfont icon-edit" @click.stop="edit" />
      </span>
      <span :title="$t('panel.matrix')">
        <i v-if="activeModel==='edit'&&curComponent.auxiliaryMatrix" class="icon iconfont icon-shujujuzhen" @click.stop="auxiliaryMatrixChange" />
      </span>
      <span :title="$t('panel.suspension')">
        <i v-if="activeModel==='edit'&&!curComponent.auxiliaryMatrix" class="icon iconfont icon-xuanfuanniu" @click.stop="auxiliaryMatrixChange" />
      </span>
      <span :title="$t('panel.details')">
        <i v-if="curComponent.type==='view'" class="icon iconfont icon-chakan" @click.stop="showViewDetails('details')" />
      </span>
      <span :title="$t('panel.enlarge')">
        <i v-if="curComponent.type==='view'" class="icon iconfont icon-fangda" @click.stop="showViewDetails('enlarge')" />
      </span>
      <span :title="$t('panel.cancel_linkage')">
        <i v-if="curComponent.type==='view'&&existLinkage" class="icon iconfont icon-quxiaoliandong" @click.stop="clearLinkage" />
      </span>
      <span :title="$t('panel.switch_picture')">
        <i v-if="activeModel==='edit'&&curComponent&&curComponent.type==='picture-add'" class="icon iconfont icon-genghuan" @click.stop="goFile" />
      </span>
      <span :title="$t('panel.jump')">
        <a v-if="showJumpFlag" :title="curComponent.hyperlinks.content " :target="curComponent.hyperlinks.openMode " :href="curComponent.hyperlinks.content ">
          <i class="icon iconfont icon-com-jump" />
        </a>
      </span>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import bus from '@/utils/bus'
import SettingMenu from '@/components/canvas/components/Editor/SettingMenu'
import LinkageField from '@/components/canvas/components/Editor/LinkageField'
import toast from '@/components/canvas/utils/toast'

export default {
  components: { SettingMenu, LinkageField },

  props: {
    element: {
      type: Object,
      required: true
    },
    active: {
      type: Boolean,
      required: false,
      default: false
    },
    // 当前模式 preview 预览 edit 编辑，
    activeModel: {
      type: String,
      required: false,
      default: 'preview'
    },
    previewVisible: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      componentType: null,
      linkageActiveStatus: false,
      editFilter: [
        'view',
        'custom'
      ],
      timer: null
    }
  },
  mounted() {
  },
  computed: {
    showJumpFlag() {
      return this.curComponent && this.curComponent.hyperlinks && this.curComponent.hyperlinks.enable
    },
    // batch operation area
    batchOptAreaShow() {
      return this.batchOptStatus && this.element.type === 'view' && !this.element.isPlugin
    },
    // 联动区域按钮显示
    linkageAreaShow() {
      return this.linkageSettingStatus && this.element !== this.curLinkageView && this.element.type === 'view'
    },
    // 编辑或预览区域显示
    normalAreaShow() {
      return !this.linkageSettingStatus && !this.batchOptStatus
    },
    existLinkage() {
      let linkageFiltersCount = 0
      this.componentData.forEach(item => {
        if (item.linkageFilters && item.linkageFilters.length > 0) {
          item.linkageFilters.forEach(linkage => {
            if (this.element.propValue.viewId === linkage.sourceViewId) {
              linkageFiltersCount++
            }
          })
        }
      })
      return linkageFiltersCount
    },
    linkageInfo() {
      return this.targetLinkageInfo[this.element.propValue.viewId]
    },
    miniHeight() {
      let miniHeight = this.curComponent.miniSizey || 1
      if (this.element.component === 'de-number-range') {
        miniHeight = this.curComponent.miniSizey || 2
      }
      return miniHeight
    },
    miniWidth() {
      return this.curComponent.miniSizex || 1
    },
    ...mapState([
      'menuTop',
      'menuLeft',
      'menuShow',
      'curComponent',
      'componentData',
      'canvasStyleData',
      'linkageSettingStatus',
      'targetLinkageInfo',
      'curLinkageView',
      'curCanvasScale',
      'batchOptStatus',
      'curBatchOptComponents'
    ])
  },
  beforeDestroy() {
  },
  methods: {
    closePreview() {
      this.$emit('closePreview')
    },
    destroyTimer() {
      if (this.timer) {
        clearInterval(this.timer)
        this.timer = null
      }
    },
    showViewDetails(openType = 'details') {
      this.$emit('showViewDetails', { openType: openType })
    },
    auxiliaryMatrixChange() {
      if (this.curComponent.auxiliaryMatrix) {
        this.curComponent.auxiliaryMatrix = false
        this.$emit('amRemoveItem')
      } else {
        this.curComponent.x = Math.round(this.curComponent.style.left / this.curCanvasScale.matrixStyleOriginWidth) + 1
        this.curComponent.y = Math.round(this.curComponent.style.top / this.curCanvasScale.matrixStyleOriginHeight) + 1
        this.curComponent.sizex = Math.round(this.curComponent.style.width / this.curCanvasScale.matrixStyleOriginWidth)
        this.curComponent.sizey = Math.round(this.curComponent.style.height / this.curCanvasScale.matrixStyleOriginHeight)
        this.curComponent.sizey = this.curComponent.sizey > this.miniHeight ? this.curComponent.sizey : this.miniHeight
        this.curComponent.sizex = this.curComponent.sizex > this.miniWidth ? this.curComponent.sizex : this.miniWidth
        this.curComponent.auxiliaryMatrix = true
        this.$emit('amAddItem')
      }
      setTimeout(() => {
        this.recordMatrixCurShadowStyle()
      }, 50)
      this.$store.state.styleChangeTimes++
      bus.$emit('auxiliaryMatrixChange')
    },
    // 记录当前样式 跟随阴影位置 矩阵处理
    recordMatrixCurShadowStyle() {
      const left = (this.curComponent.x - 1) * this.curCanvasScale.matrixStyleWidth
      const top = (this.curComponent.y - 1) * this.curCanvasScale.matrixStyleHeight
      const width = this.curComponent.sizex * this.curCanvasScale.matrixStyleWidth
      const height = this.curComponent.sizey * this.curCanvasScale.matrixStyleHeight
      const style = {
        left: left,
        top: top,
        width: width,
        height: height
      }
      this.$store.commit('setShapeStyle', style)
      // resize
      this.$emit('resizeView')
    },
    // edit() {
    //   // 编辑时临时保存 当前修改的画布
    //   this.$store.dispatch('panel/setComponentDataTemp', JSON.stringify(this.componentData))
    //   this.$store.dispatch('panel/setCanvasStyleDataTemp', JSON.stringify(this.canvasStyleData))
    //   if (this.curComponent.type === 'view') {
    //     this.$store.dispatch('chart/setViewId', null)
    //     this.$store.dispatch('chart/setViewId', this.curComponent.propValue.viewId)
    //     bus.$emit('PanelSwitchComponent', { name: 'ChartEdit', param: { 'id': this.curComponent.propValue.viewId, 'optType': 'edit' }})
    //   }
    //   if (this.curComponent.type === 'custom') {
    //     bus.$emit('component-dialog-edit')
    //   }
    //   // 编辑样式组件
    //   if (this.curComponent.type === 'v-text' || this.curComponent.type === 'rect-shape') {
    //     bus.$emit('component-dialog-style')
    //   }
    // },
    edit() {
      if (this.curComponent.type === 'custom') {
        bus.$emit('component-dialog-edit')
      } else if (this.curComponent.type === 'v-text' || this.curComponent.type === 'de-rich-text' || this.curComponent.type === 'rect-shape') {
        bus.$emit('component-dialog-style')
      } else { bus.$emit('change_panel_right_draw', true) }
    },
    linkageEdit() {

    },
    amRemoveItem() {
      this.$emit('amRemoveItem')
    },
    // 清除相同sourceViewId 的 联动条件
    clearLinkage() {
      this.componentData.forEach(item => {
        if (item.linkageFilters && item.linkageFilters.length > 0) {
          const newList = item.linkageFilters.filter(linkage => linkage.sourceViewId !== this.element.propValue.viewId)
          item.linkageFilters.splice(0, item.linkageFilters.length)
          // 重新push 可保证数组指针不变 可以watch到
          if (newList.length > 0) {
            newList.forEach(newLinkage => {
              item.linkageFilters.push(newLinkage)
            })
          }
        }
      })
    },
    linkJumpSet() {
      this.$emit('linkJumpSet')
    },
    goFile() {
      this.$refs.files.click()
    },
    handleFileChange(e) {
      const file = e.target.files[0]
      if (!file.type.includes('image')) {
        toast('只能插入图片')
        return
      }
      const reader = new FileReader()
      reader.onload = (res) => {
        const fileResult = res.target.result
        this.curComponent.propValue = fileResult
        this.$store.commit('recordSnapshot', 'handleFileChange')
      }

      reader.readAsDataURL(file)
    },
    boardSet() {
      this.$emit('boardSet')
    },
    batchOptChange(val) {
      if (val) {
        // push
        this.$store.commit('addCurBatchComponent', this.element.propValue.viewId)
      } else {
        // remove
        this.$store.commit('removeCurBatchComponentWithId', this.element.propValue.viewId)
      }
    }
  }
}
</script>

<style lang="scss" scoped>
  .bar-main{
    position: absolute;
    right: 0px;
    float:right;
    z-index: 2;
    border-radius:2px;
    padding-left: 3px;
    padding-right: 0px;
    cursor:pointer!important;
    background-color: rgba(10,123,224, 1);
  }
  .bar-main i{
    color: white;
    float: right;
    margin-right: 3px;
  }

  .bar-main ::v-deep .el-checkbox__inner{
    width: 16px;
    height: 16px;
  }

  .bar-main ::v-deep .el-checkbox__inner::after{
    width: 4.5px;
  }

</style>
