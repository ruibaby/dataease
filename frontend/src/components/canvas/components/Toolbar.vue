<template>
  <div>
    <div class="switch-position">
      <el-radio-group v-model="mobileLayoutInitStatus" size="mini" @change="openMobileLayout">
        <el-radio-button :label="false">
          <span style="float: left;">
            <i class="el-icon-monitor" />
          </span>
        </el-radio-button>
        <el-radio-button :label="true">
          <span class="icon iconfont icon-yidongduan" />
        </el-radio-button>
      </el-radio-group>
    </div>
    <div v-show="editControlButton" class="toolbar">
      <span style="float: right;">
        <el-button v-if="mobileLayoutStatus" size="mini" @click="editReset">
          {{ $t('commons.reset') }}
        </el-button>
        <el-button type="primary" size="mini" @click="editSave">
          {{ $t('commons.confirm') }}
        </el-button>
        <el-button size="mini" @click="editCancel">
          {{ $t('commons.cancel') }}
        </el-button>
      </span>
    </div>

    <div v-show="!editControlButton" class="toolbar">
      <div class="panel-info-area">
        <el-tooltip :content="$t('panel.back') ">
          <span class="icon iconfont icon-jiantou insert" @click="closePanelEdit" />
        </el-tooltip>
        <span class="text">
          {{ panelInfo.name }}
        </span>
      </div>
      <el-tooltip :content="$t('panel.undo') ">
        <span class="icon iconfont icon-outline-undo insert" @click="undo" />
      </el-tooltip>
      <el-tooltip :content="$t('panel.redo') ">
        <span class="icon iconfont icon-outline-redo insert" @click="redo" />
      </el-tooltip>
      <el-tooltip :content="$t('panel.fullscreen_preview')">
        <span class="icon iconfont icon-fangda insert" @click="clickPreview" />
      </el-tooltip>
      <el-divider direction="vertical" />

      <span class="button_self">
        <el-dropdown :hide-on-click="false" trigger="click" placement="bottom-start" size="mini">
          <span class="icon iconfont icon-gengduo insert de-icon-base"><span class="icon-font-margin">{{ $t('panel.more') }}</span></span>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item>
              <el-dropdown placement="right-start" size="mini" style="width: 100%">
                <span>
                  <span
                    class="icon iconfont"
                    :class="[canvasStyleData.auxiliaryMatrix?'icon-shujujuzhen':'icon-xuanfuanniu']"
                  />
                  <span class="icon-font-margin" style="font-size: 12px">{{ $t('panel.new_element_distribution') }}</span>
                  <i class="el-icon-arrow-right el-icon--right" />
                </span>
                <el-dropdown-menu slot="dropdown">
                  <el-dropdown-item @click.native="auxiliaryMatrixChange">
                    <span :class="[!canvasStyleData.auxiliaryMatrix?'font-active':'']"> {{ $t('panel.suspension') }} </span>
                    <i v-if="!canvasStyleData.auxiliaryMatrix" class=" font-active el-icon-check" />
                  </el-dropdown-item>
                  <el-dropdown-item @click.native="auxiliaryMatrixChange">
                    <span :class="[canvasStyleData.auxiliaryMatrix?'font-active':'']"> {{ $t('panel.matrix') }} </span>
                    <i v-if="canvasStyleData.auxiliaryMatrix" class=" font-active el-icon-check" />
                  </el-dropdown-item>
                </el-dropdown-menu>
              </el-dropdown>
            </el-dropdown-item>
            <el-dropdown-item>
              <span class="icon iconfont-tb" :class="[canvasStyleData.aidedDesign.showGrid?'icon-wangge-open':'icon-wangge-close']" />
              <span class="icon-font-margin">{{ $t('panel.aided_grid') }}</span>
              <el-switch v-model="showGridSwitch" size="mini" @change="showGridChange" />
            </el-dropdown-item>
            <el-dropdown-item @click.native="openOuterParamsSet">
              <span class="icon iconfont-tb icon-canshu" />
              <span class="icon-font-margin">{{ $t('panel.params_setting') }}</span>
            </el-dropdown-item>
            <el-dropdown-item @click.native="clearCanvas">
              <span class="icon iconfont-tb icon-qingkong" />
              <span class="icon-font-margin">{{ $t('panel.clean_canvas') }}</span>
            </el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </span>
      <span class="icon iconfont icon-magic-line insert" @click="showPanel"><span class="icon-font-margin">{{ $t('panel.panel_style') }}</span></span>
      <span class="icon iconfont icon-piliang-copy insert" @click="batchOption"><span class="icon-font-margin">{{ $t('panel.batch_opt') }}</span></span>
      <span style="float: right;margin-left: 10px">
        <el-button size="mini" type="primary" :disabled="saveButtonDisabled" @click="save(false)">
          {{ $t('commons.save') }}
        </el-button>
      </span>
    </div>

    <!--关闭弹框-->
    <el-dialog
      :visible.sync="closePanelVisible"
      :title="$t('panel.panel_save_tips')"
      :show-close="false"
      width="30%"
      class="dialog-css"
    >
      <el-row style="height: 20px">
        <el-col :span="4">
          <svg-icon icon-class="warn-tre" style="width: 20px;height: 20px;float: right" />
        </el-col>
        <el-col :span="20">
          <span style="font-size: 13px;margin-left: 10px;font-weight: bold;line-height: 20px">{{ $t('panel.panel_save_warn_tips') }}</span>
        </el-col>
      </el-row>
      <div slot="footer" class="dialog-footer">
        <el-button style="float: left" size="mini" @click="closeNotSave()">{{ $t('panel.do_not_save') }}</el-button>
        <el-button size="mini" @click="closePanelVisible=false">{{ $t('panel.cancel') }}</el-button>
        <el-button type="primary" size="mini" @click="save(true)">{{ $t('panel.save') }}
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import generateID from '@/components/canvas/utils/generateID'
import toast from '@/components/canvas/utils/toast'
import { mapState } from 'vuex'
import { commonStyle, commonAttr } from '@/components/canvas/custom-component/component-list'
import eventBus from '@/components/canvas/utils/eventBus'
import { deepCopy, mobile2MainCanvas } from '@/components/canvas/utils/utils'
import { panelUpdate } from '@/api/panel/panel'
import { saveLinkage, getPanelAllLinkageInfo } from '@/api/panel/linkage'
import bus from '@/utils/bus'
import {
  DEFAULT_COMMON_CANVAS_STYLE_STRING
} from '@/views/panel/panel'
import { queryPanelJumpInfo } from '@/api/panel/linkJump'

export default {
  name: 'Toolbar',
  props: {
    styleButtonActive: Boolean,
    aidedButtonActive: Boolean
  },
  data() {
    return {
      showGridSwitch: false,
      mobileLayoutInitStatus: false,
      isShowPreview: false,
      needToChange: [
        'top',
        'left',
        'width',
        'height',
        'fontSize',
        'borderWidth'
      ],
      scale: '100%',
      timer: null,
      changes: 0,
      closePanelVisible: false
    }
  },
  computed: {
    panelInfo() {
      return this.$store.state.panel.panelInfo
    },
    saveButtonDisabled() {
      return this.changeTimes === 0 || this.snapshotIndex === this.lastSaveSnapshotIndex
    },
    editControlButton() {
      return this.linkageSettingStatus || this.mobileLayoutStatus
    },
    ...mapState([
      'componentData',
      'canvasStyleData',
      'areaData',
      'curComponent',
      'changeTimes',
      'snapshotIndex',
      'lastSaveSnapshotIndex',
      'linkageSettingStatus',
      'curLinkageView',
      'targetLinkageInfo',
      'mobileLayoutStatus',
      'mobileComponentData',
      'componentDataCache',
      'batchOptStatus'
    ])
  },
  created() {
    eventBus.$on('preview', this.preview)
    eventBus.$on('save', this.save)
    eventBus.$on('clearCanvas', this.clearCanvas)
    this.scale = this.canvasStyleData.scale
    this.mobileLayoutInitStatus = this.mobileLayoutStatus
    this.showGridSwitch = this.canvasStyleData.aidedDesign.showGrid
  },
  methods: {
    close() {
      // 关闭页面清理缓存
      this.clearCanvas()
      this.$emit('close-left-panel')
      this.$nextTick(() => {
        bus.$emit('PanelSwitchComponent', { name: 'PanelMain' })
      })
    },
    closePanelEdit() {
      if (this.changeTimes === 0 || this.snapshotIndex === this.lastSaveSnapshotIndex) { // 已保存
        this.close()
      } else {
        this.closePanelVisible = true
      }
    },
    goFile() {
      this.$refs.files.click()
    },
    format(value) {
      const scale = this.scale
      return value * scale / 100
    },
    getOriginStyle(value) {
      const scale = this.canvasStyleData.scale
      const result = value / (scale / 100)
      return result
    },
    handleScaleChange() {
      clearTimeout(this.timer)
      setTimeout(() => {
        const componentData = deepCopy(this.componentData)
        componentData.forEach(component => {
          Object.keys(component.style).forEach(key => {
            if (this.needToChange.includes(key)) {
              // 根据原来的比例获取样式原来的尺寸
              // 再用原来的尺寸 * 现在的比例得出新的尺寸
              component.style[key] = this.format(this.getOriginStyle(component.style[key]))
            }
          })
        })
        this.$store.commit('setComponentData', componentData)
        this.$store.commit('setCanvasStyle', {
          ...this.canvasStyleData,
          scale: this.scale
        })
      }, 1000)
    },

    lock() {
      this.$store.commit('lock')
    },

    unlock() {
      this.$store.commit('unlock')
    },

    compose() {
      this.$store.commit('compose')
      this.$store.commit('recordSnapshot', 'compose')
    },

    decompose() {
      this.$store.commit('decompose')
      this.$store.commit('recordSnapshot', 'decompose')
    },

    undo() {
      this.$store.commit('undo')
    },

    redo() {
      this.$store.commit('redo')
    },

    showPanel() {
      this.$emit('showPanel', 2)
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
        const img = new Image()
        img.onload = () => {
          const scaleWith = img.width / 400
          const scaleHeight = img.height / 200
          let scale = scaleWith > scaleHeight ? scaleWith : scaleHeight
          scale = scale > 1 ? scale : 1
          this.$store.commit('addComponent', {
            component: {
              ...commonAttr,
              id: generateID(),
              component: 'Picture',
              label: '图片',
              icon: '',
              propValue: fileResult,
              style: {
                ...commonStyle,
                top: 0,
                left: 0,
                width: img.width / scale,
                height: img.height / scale
              }
            }
          })
          this.$store.commit('recordSnapshot', 'handleFileChange')
        }
        img.src = fileResult
      }

      reader.readAsDataURL(file)
    },

    preview() {
      this.isShowPreview = true
      this.$store.commit('setEditMode', 'preview')
    },

    save(withClose) {
      // 清理联动信息
      this.$store.commit('clearPanelLinkageInfo')
      // 保存到数据库
      const requestInfo = {
        id: this.panelInfo.id,
        panelStyle: JSON.stringify(this.canvasStyleData),
        panelData: JSON.stringify(this.componentData)
      }
      const components = deepCopy(this.componentData)
      components.forEach(view => {
        if (view.DetailAreaCode) {
          view.DetailAreaCode = null
        }
        if (view.filters && view.filters.length > 0) {
          view.filters = []
        }
        if (view.type === 'de-tabs') {
          view.options.tabList && view.options.tabList.length > 0 && view.options.tabList.forEach(tab => {
            if (tab.content && tab.content.filters && tab.content.filters.length > 0) {
              tab.content.filters = []
            }
          })
        }
      })
      // 无需保存条件
      requestInfo.panelData = JSON.stringify(components)
      panelUpdate(requestInfo).then(response => {
        this.$store.commit('refreshSaveStatus')
        this.$message({
          message: this.$t('commons.save_success'),
          type: 'success',
          showClose: true
        })
        if (withClose) {
          this.close()
        }
      })
    },
    clearCanvas() {
      this.$store.commit('setComponentData', [])
      this.$store.commit('setCanvasStyle', DEFAULT_COMMON_CANVAS_STYLE_STRING)
      this.$store.commit('recordSnapshot', 'clearCanvas')
      this.$store.commit('setInEditorStatus', false)
    },

    handlePreviewChange() {
      this.$store.commit('setEditMode', 'edit')
    },

    clickPreview() {
      this.$emit('previewFullScreen')
    },
    openOuterParamsSet() {
      this.$emit('outerParamsSetVisibleChange', true)
    },
    changeAidedDesign() {
      this.$emit('changeAidedDesign')
    },
    closeNotSave() {
      this.close()
    },
    saveLinkage() {
      // 字段检查
      for (const key in this.targetLinkageInfo) {
        let subCheckCount = 0
        const linkageInfo = this.targetLinkageInfo[key]
        const linkageFields = linkageInfo['linkageFields']
        if (linkageFields) {
          linkageFields.forEach(function(linkage) {
            if (!(linkage.sourceField && linkage.targetField)) {
              subCheckCount++
            }
          })
        }

        if (subCheckCount > 0) {
          this.$message({
            message: this.$t('chart.datalist') + '【' + linkageInfo.targetViewName + '】' + this.$t('panel.exit_un_march_linkage_field'),
            type: 'error',
            showClose: true
          })
          return
        }
      }
      const request = {
        panelId: this.panelInfo.id,
        sourceViewId: this.curLinkageView.propValue.viewId,
        linkageInfo: this.targetLinkageInfo
      }
      saveLinkage(request).then(rsp => {
        // 刷新联动信息
        getPanelAllLinkageInfo(this.panelInfo.id).then(rsp => {
          this.$store.commit('setNowPanelTrackInfo', rsp.data)
        })
        this.cancelLinkageSettingStatus()
        // 刷新跳转信息
        queryPanelJumpInfo(this.panelInfo.id).then(rsp => {
          this.$store.commit('setNowPanelJumpInfo', rsp.data)
        })
      })
    },
    cancelMobileLayoutStatue(sourceComponentData) {
      this.$store.commit('setComponentData', sourceComponentData)
      this.$store.commit('setMobileLayoutStatus', false)
      this.mobileLayoutInitStatus = false
    },
    cancelLinkage() {
      this.cancelLinkageSettingStatus()
    },
    cancelLinkageSettingStatus() {
      this.$store.commit('clearLinkageSettingInfo')
    },
    auxiliaryMatrixChange() {
      this.canvasStyleData.auxiliaryMatrix = !this.canvasStyleData.auxiliaryMatrix
    },
    showGridChange() {
      this.$store.state.styleChangeTimes++
      this.canvasStyleData.aidedDesign.showGrid = !this.canvasStyleData.aidedDesign.showGrid
    },
    // batch option
    batchOption() {
      bus.$emit('change_panel_right_draw', !this.batchOptStatus)
      this.$store.commit('setBatchOptStatus', !this.batchOptStatus)
    },
    // 启用移动端布局
    openMobileLayout(switchVal) {
      if (switchVal) {
        this.$store.commit('openMobileLayout')
      } else {
        this.mobileLayoutSave()
      }
    },
    editSave() {
      if (this.mobileLayoutStatus) {
        this.mobileLayoutSave()
      } else {
        this.saveLinkage()
      }
    },
    editReset() {
      this.cancelMobileLayoutStatue(JSON.parse(this.componentDataCache))
      this.$store.commit('openMobileLayout')
    },
    editCancel() {
      if (this.mobileLayoutStatus) {
        this.cancelMobileLayoutStatue(JSON.parse(this.componentDataCache))
      } else {
        this.cancelLinkageSettingStatus()
      }
    },
    // 移动端布局保存
    mobileLayoutSave() {
      this.$store.state.styleChangeTimes++
      const mobileDataObj = {}
      this.componentData.forEach(item => {
        mobileDataObj[item.id] = item
      })
      const sourceComponentData = JSON.parse(this.componentDataCache)
      this.$store.commit('setComponentDataCache', null)
      sourceComponentData.forEach(item => {
        if (mobileDataObj[item.id]) {
          mobile2MainCanvas(item, mobileDataObj[item.id])
        } else {
          item.mobileSelected = false
        }
      })
      this.cancelMobileLayoutStatue(sourceComponentData)
    }
  }
}
</script>

<style lang="scss" scoped>
  .toolbar {
    float: right;
    height: 56px;
    line-height: 56px;
    min-width: 400px;

    .canvas-config {
      display: inline-block;
      margin-left: 10px;
      font-size: 14px;

      input {
        width: 50px;
        margin-left: 10px;
        outline: none;
        padding: 0 5px;
        border: 1px solid #ddd;
      }

      span {
        margin-left: 10px;
      }
    }

    .insert {
      display: inline-block;
      font-weight: 400 !important;
      font-size: 14px !important;
      font-family: PingFang SC;
      line-height: 1;
      white-space: nowrap;
      cursor: pointer;
      color: var(--TextPrimary, #606266);
      -webkit-appearance: none;
      text-align: center;
      box-sizing: border-box;
      outline: 0;
      margin: 0;
      transition: .1s;
      padding: 5px 5px;
      border-radius: 3px;
      margin-left: 5px;

      &:active {
        color: #000;
        border-color: #3a8ee6;
        background-color: red;
        outline: 0;
      }

      &:hover {
        background-color: #ecf5ff;
        color: #3a8ee6;
      }
    }
  }

  .button-show {
    background-color: #ebf2fe !important;
  }

  .button-closed {
    background-color: #ffffff !important;
  }

  ::v-deep .el-switch__core {
    width: 30px !important;
    height: 15px;
  }

  /*设置圆*/
  ::v-deep .el-switch__core::after {
    width: 14px;
    height: 14px;
    margin-top: -1px;
    margin-bottom: 2px;
  }

  .iconfont-tb {
    font-family: "iconfont" !important;
    font-size: 12px;
    font-style: normal;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

  .switch-position {
    position: absolute;
    top: 13px;
    right: 50%;
    width: 100px;
  }

  .button_self {
    margin-right: 5px;
  }

  .button_self ::v-deep .el-button--mini {
    padding: 7px 7px !important;
  }

  .font-active {
    font-color: #3a8ee6 !important;
  }

  .icon-active {
    color: #3a8ee6;
  }

  .icon-unactivated {
    display: none;
  }

  .panel-info-area {
    position: absolute;
    left: 10px;

    .text {
      margin-left: 15px;
      font-size: 16px;
      font-weight: 500;
      color: var(--TextPrimary, #606266);
    }
  ;

    .icon-back {
      font-size: 20px;
      font-weight: bold;
      color: var(--MenuActiveBG, #409EFF);
    }

  }

  .icon-font-margin{
    margin-left: 2px;
  }

</style>
