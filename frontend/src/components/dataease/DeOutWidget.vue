<template>
  <div ref="myContainer" class="my-container">
    <div ref="conditionMain" :style="outsideStyle" class="condition-main">
      <div v-if="element.options.attrs.title" ref="deTitleContainer" :style="titleStyle" class="condition-title">
        <div class="condition-title-absolute">
          <div class="first-title">
            <div class="span-container">
              <span ref="deTitle">{{ element.options.attrs.title }}</span>
            </div>
          </div>
        </div>
      </div>
      <div
        ref="deContentContainer"
        class="condition-content"
        :class="element.options.attrs.title ? '' : 'condition-content-default'"
      >
        <div class="condition-content-container">
          <div class="first-element">
            <div
              :class="element.component === 'de-select-grid' ? 'first-element-grid-contaner': ''"
              class="first-element-contaner"
            >

              <component
                :is="element.component"
                v-if="element.type==='custom'"
                :id="'component' + element.id"
                class="component-custom"
                :out-style="element.style"
                :element="element"
                :in-draw="inDraw"
                :in-screen="inScreen"
                :size="sizeInfo"
              />
            </div>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'
export default {
  name: 'DeOutWidget',
  props: {
    element: {
      type: Object,
      default: () => {}
    },
    inDraw: {
      type: Boolean,
      default: true
    },
    inScreen: {
      type: Boolean,
      required: false,
      default: true
    },
    h: {
      type: Number,
      default: 50
    },
    editMode: {
      type: String,
      require: false,
      default: 'edit'
    }
  },
  data() {
    return {
      inputMaxSize: 46,
      inputLargeSize: 42,
      inputSmallSize: 38,
      inputMiniSize: 32,
      options: null,
      showNumber: false,
      mainClass: '',
      mainHeight: 75,
      duHeight: 46,
      titleStyle: null,
      outsideStyle: null
    }
  },
  computed: {
    sizeInfo() {
      let size
      if (this.duHeight > this.inputLargeSize) {
        size = 'medium'
      } else if (this.duHeight > this.inputSmallSize) {
        size = 'small'
      } else {
        size = 'mini'
      }
      return size
    },
    ...mapState([
      'curCanvasScale'
    ])
  },
  watch: {
    'element.style': {
      handler(val) {
        this.handlerPositionChange(val)
      },
      deep: true,
      immediate: true
    }
  },
  mounted() {
    // this.watchSize()
  },
  created() {
    const { horizontal, vertical } = this.element.style
    this.$set(this.element.style, 'horizontal', horizontal || 'left')
    this.$set(this.element.style, 'vertical', vertical || 'center')
  },
  methods: {
    handlerPositionChange(val) {
      const { horizontal = 'left', vertical = 'center' } = val
      this.titleStyle = {
        width: '100%',
        textAlign: horizontal
      }
      this.outsideStyle = {
        flexWrap: 'wrap'
      }
      if (vertical !== 'top') {
        this.titleStyle = null
        this.outsideStyle = {
          flexDirection: horizontal === 'right' ? 'row-reverse' : '',
          alignItems: 'center'
        }
      }
    }
  }
}
</script>

<style lang="scss" scoped>
  .my-container {
    position: relative;
    overflow: auto;
    top: 0px;
    right: 0px;
    bottom: 0px;
    left: 0px;
  }
  .ccondition-main {
    position: absolute;
    overflow: auto;
    top: 0px;
    right: 0px;
    bottom: 0px;
    left: 0px;
    display: flex;
  }
  .condition-title {
    height: 2em;
    cursor: -webkit-grab;
    line-height: 2em;
    white-space: nowrap;
  }
  .span-container {
    overflow: hidden auto;
    position: relative;
    padding: 0 5px;
  }
  .condition-content {
    overflow: auto hidden;
    letter-spacing: 0px !important;
    width: 100%;
  }
  .condition-content-container {
    position: relative;
    display: table;
    width: 100%;
    height: 100%;
    white-space: nowrap;
  }
  .first-element {
    position: relative;
    display: table-cell;
    vertical-align: middle;
    margin: 0px;
    padding: 0px;
    height: 100%;
  }
  .first-element-contaner {
    width: calc(100% - 10px);
    background: initial;
    margin: 0 4px;
    div {
      width: 100%;
    }
    display: flex;
    align-items: flex-end;
  }
  .first-element-grid-contaner {
    background: #fff;
    border: 1px solid #d7dae2;
    top: 5px;
  }
  .condition-main-line {
    height: 40px !important;
  }
  .condition-main {
    display: flex;
    padding-top: 5px;
  }
  .condition-content-default {
    inset: 0px 0px 0px !important;
  }
</style>
