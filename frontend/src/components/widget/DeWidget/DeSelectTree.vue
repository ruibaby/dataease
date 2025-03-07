<template>

  <el-tree-select
    v-if="element.options!== null && element.options.attrs!==null && show"
    ref="deSelectTree"
    v-model="value"
    popover-class="test-class-wrap"
    :data="datas"
    :select-params="selectParams"
    :tree-params="treeParams"
    :filter-node-method="_filterFun"
    :tree-render-fun="_renderFun"
    @searchFun="_searchFun"
    @node-click="changeNode"
    @removeTag="changeNodeIds"
    @check="changeCheckNode"
    @select-clear="selectClear"
  />

</template>

<script>
import { mappingFieldValues, linkMappingFieldValues } from '@/api/dataset/dataset'
import bus from '@/utils/bus'
import { getLinkToken, getToken } from '@/utils/auth'
import ElTreeSelect from '@/components/ElTreeSelect'
export default {
  components: { ElTreeSelect },
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
    size: String
  },
  data() {
    return {
      show: true,
      selectOptionWidth: 0,
      datas: [],
      value: this.isSingle ? '' : [],
      selectParams: {
        clearable: true,
        placeholder: this.$t(this.element.options.attrs.placeholder)
      },
      treeParams: {
        showParent: true,
        clickParent: true,
        filterable: true,
        // 只想要子节点，不需要父节点
        leafOnly: false,
        includeHalfChecked: false,
        'check-strictly': false,
        'default-expand-all': false,
        'expand-on-click-node': false,
        'render-content': this._renderFun,
        data: [],
        props: {
          children: 'children',
          label: 'text',
          rootId: 'root',
          disabled: 'disabled',
          parentId: 'pid',
          value: 'id'
        }
      }
    }
  },
  computed: {
    operator() {
      return this.element.options.attrs.multiple ? 'in' : 'eq'
    },
    defaultValueStr() {
      if (!this.element || !this.element.options || !this.element.options.value) return ''
      return this.element.options.value.toString()
    },
    viewIds() {
      if (!this.element || !this.element.options || !this.element.options.attrs.viewIds) return ''
      return this.element.options.attrs.viewIds.toString()
    },
    manualModify() {
      return !!this.element.options.manualModify
    },
    panelInfo() {
      return this.$store.state.panel.panelInfo
    },
    isSingle() {
      return this.element.options.attrs.multiple
    }
  },

  watch: {
    'viewIds': function(value, old) {
      if (typeof value === 'undefined' || value === old) return
      this.setCondition()
    },
    'defaultValueStr': function(value, old) {
      if (value === old) return
      this.value = this.fillValueDerfault()
      this.changeValue(value)
    },
    'element.options.attrs.fieldId': function(value, old) {
      if (value === null || typeof value === 'undefined' || value === old) return
      this.datas = []

      let method = mappingFieldValues
      const token = this.$store.getters.token || getToken()
      const linkToken = this.$store.getters.linkToken || getLinkToken()
      if (!token && linkToken) {
        method = linkMappingFieldValues
      }
      const param = { fieldIds: this.element.options.attrs.fieldId.split(','), sort: this.element.options.attrs.sort }
      if (this.panelInfo.proxy) {
        param.userId = this.panelInfo.proxy
      }
      this.element.options.attrs.fieldId &&
      this.element.options.attrs.fieldId.length > 0 &&
      method(param).then(res => {
        this.datas = this.optionDatas(res.data)
        this.$nextTick(() => {
          this.$refs.deSelectTree && this.$refs.deSelectTree.treeDataUpdateFun(this.datas)
        })
      })
      this.element.options.value = ''
    },
    'element.options.attrs.multiple': function(value, old) {
      if (typeof old === 'undefined' || value === old) return

      if (!this.inDraw) {
        this.value = value ? [] : null
        this.element.options.value = ''
      }
      this.show = false
      this.$nextTick(() => {
        // this.value = value ? [] : null

        this.show = true
        this.$nextTick(() => {
          const defaultV = this.element.options.value === null ? '' : this.element.options.value.toString()
          if (value) {
            if (defaultV === null || typeof defaultV === 'undefined' || defaultV === '' || defaultV === '[object Object]') {
              this.value = []
            } else {
              this.value = defaultV.split(',')
            }
          } else {
            if (defaultV === null || typeof defaultV === 'undefined' || defaultV === '' || defaultV === '[object Object]') {
              this.value = ''
            } else {
              this.value = defaultV.split(',')[0]
            }
          }
          this.$refs.deSelectTree && this.$refs.deSelectTree.treeDataUpdateFun(this.datas)
        })
      })
    },
    'element.options.attrs.sort': function(value, old) {
      if (value === null || typeof value === 'undefined' || value === old) return
      this.datas = []

      let method = mappingFieldValues
      const token = this.$store.getters.token || getToken()
      const linkToken = this.$store.getters.linkToken || getLinkToken()
      if (!token && linkToken) {
        method = linkMappingFieldValues
      }
      const param = { fieldIds: this.element.options.attrs.fieldId.split(','), sort: this.element.options.attrs.sort }
      if (this.panelInfo.proxy) {
        param.userId = this.panelInfo.proxy
      }
      this.element.options.attrs.fieldId &&
      this.element.options.attrs.fieldId.length > 0 &&
      method(param).then(res => {
        this.datas = this.optionDatas(res.data)
        this.$nextTick(() => {
          this.$refs.deSelectTree && this.$refs.deSelectTree.treeDataUpdateFun(this.datas)
        })
      })
      this.element.options.value = ''
    }

  },
  created() {
    if (!this.element.options.attrs.sort) {
      this.element.options.attrs.sort = {}
    }
    this.initLoad()
  },
  mounted() {
    bus.$on('onScroll', () => {

    })
    bus.$on('reset-default-value', id => {
      if (this.inDraw && this.manualModify && this.element.id === id) {
        this.value = this.fillValueDerfault()
        this.changeValue(this.value)
      }
    })
  },

  methods: {
    selectClear() {
      this.changeValue(this.value)
    },
    changeNode(data, node) {
      this.changeValue(this.value)
    },
    changeCheckNode(data, obj) {
      const { checkedKeys } = obj
      if (checkedKeys) this.value = checkedKeys
      this.changeValue(this.value)
    },
    changeNodeIds(ids) {
      this.value = ids
      this.changeValue(this.value)
    },
    initLoad() {
      this.value = this.fillValueDerfault()
      this.datas = []
      if (this.element.options.attrs.fieldId) {
        let method = mappingFieldValues
        const token = this.$store.getters.token || getToken()
        const linkToken = this.$store.getters.linkToken || getLinkToken()
        if (!token && linkToken) {
          method = linkMappingFieldValues
        }
        method({ fieldIds: this.element.options.attrs.fieldId.split(','), sort: this.element.options.attrs.sort }).then(res => {
          this.datas = this.optionDatas(res.data)
          this.$nextTick(() => {
            this.$refs.deSelectTree && this.$refs.deSelectTree.treeDataUpdateFun(this.datas)
          })
        })
      }
      if (this.element.options.value) {
        this.value = this.fillValueDerfault()
        this.changeValue(this.value)
      }
    },
    changeValue(value) {
      if (!this.inDraw) {
        if (value === null) {
          this.element.options.value = ''
        } else {
          this.element.options.value = Array.isArray(value) ? value.join() : value
        }
        this.element.options.manualModify = false
      } else {
        this.element.options.manualModify = true
      }
      this.setCondition()
    },

    setCondition() {
      const val = this.formatFilterValue()

      const param = {
        component: this.element,
        value: val,
        operator: this.operator,
        isTree: true
      }
      this.inDraw && this.$store.commit('addViewFilter', param)
    },
    formatFilterValue() {
      const SEPARATOR = '-de-'
      if (this.value === null) return []
      if (Array.isArray(this.value)) {
        const results = []
        const duplicateMap = {}
        this.value.forEach(item => {
          const links = item.split(SEPARATOR)
          let temp = ''
          for (let index = 0; index < links.length; index++) {
            const isLast = index === (links.length - 1)
            const isFirst = index === 0
            const node = links[index]

            temp += ((isFirst ? '' : SEPARATOR) + node)
            if (duplicateMap[temp] && !isLast) {
              delete duplicateMap[temp]
            }
          }

          duplicateMap[item] = true
        })
        for (const key in duplicateMap) {
          if (Object.hasOwnProperty.call(duplicateMap, key) && duplicateMap[key]) {
            const node = key.replaceAll(SEPARATOR, ',')
            results.push(node)
          }
        }
        return results
        // return this.value
      }
      const result = this.value.split(',').map(v => v.replaceAll(SEPARATOR, ','))
      return result
    },

    fillValueDerfault() {
      const defaultV = this.element.options.value === null ? '' : this.element.options.value.toString()
      if (this.element.options.attrs.multiple) {
        if (defaultV === null || typeof defaultV === 'undefined' || defaultV === '' || defaultV === '[object Object]') return []
        return defaultV.split(',')
      } else {
        if (defaultV === null || typeof defaultV === 'undefined' || defaultV === '' || defaultV === '[object Object]') return null
        return defaultV.split(',')[0]
      }
    },
    optionDatas(datas) {
      if (!datas) return null

      return datas.filter(item => !!item)
    },

    /* 下面是树的渲染方法 */

    _filterFun(value, data, node) {
      if (!value) return true
      return data.id.toString().indexOf(value.toString()) !== -1
    },
    // 树点击
    _nodeClickFun(data, node, vm) {
      console.log('this _nodeClickFun', this.value, data, node)
    },
    // 树过滤
    _searchFun(value) {
      console.log(value, '<--_searchFun')
      // 自行判断 是走后台查询，还是前端过滤
      this.$refs.deSelectTree.filterFun(value)
      // 后台查询
      // this.$refs.deSelectTree.treeDataUpdateFun(treeData);
    },
    // 自定义render
    _renderFun(h, { node, data, store }) {
      const { props, clickParent } = this.treeParams
      return (
        <span class={['custom-tree-node', !clickParent && data[props.children] && data[props.children].length ? 'disabled' : null]}>
          <span>{node.label}</span>
        </span>
      )
    }

  }

}
</script>

<style lang="scss" scoped>

</style>
