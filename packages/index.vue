<template>
  <div class="form-designer">
    <el-container>
      <!-- 左侧字段 -->
      <el-aside :width="leftWidth">
        <div class="fields-list">
          <div v-for="(field, index) in fields"
               :key="index">
            <div v-if="!field.disabled">
              <div class="field-title">{{field.title}}</div>
              <draggable tag="ul"
                         :list="field.list"
                         :group="{ name: 'form', pull: 'clone', put: false }"
                         ghost-class="ghost"
                         :sort="false">
                <li class="field-label"
                    v-for="(item, index) in field.list"
                    :key="index">
                  <a>
                    <i class="icon iconfont"
                       :class="item.icon"></i>
                    <span>{{item.title || item.label}}</span>
                  </a>
                </li>
              </draggable>
            </div>
            <div v-else>
              <div class="field-title">{{field.title}}
                <span class="danger">（开发中）</span>
              </div>
              <ul>
                <li class="field-label-disabled"
                    v-for="(item, index) in field.list"
                    :key="index">
                  <a>
                    <i class="icon iconfont"
                       :class="item.icon"></i>
                    <span>{{item.title || item.label}}</span>
                  </a>
                </li>
              </ul>
            </div>
          </div>
        </div>
      </el-aside>
      <!-- 中间主布局 -->
      <el-container class="widget-container"
                    direction="vertical">
        <el-header class="widget-container-header">
          <el-button v-if="showAvueDoc"
                     type="text"
                     size="medium"
                     icon="el-icon-document"
                     @click="handleAvueDoc">Avue文档</el-button>
          <el-button type="text"
                     size="medium"
                     icon="el-icon-upload2"
                     @click="importJsonVisible = true">导入JSON</el-button>
          <el-button type="text"
                     size="medium"
                     icon="el-icon-download"
                     @click="handleGenerateJson">生成JSON</el-button>
          <el-button type="text"
                     size="medium"
                     icon="el-icon-view"
                     @click="handlePreview">预览</el-button>
          <el-button class="danger"
                     type="text"
                     size="medium"
                     icon="el-icon-delete"
                     @click="handleClear">清空</el-button>
        </el-header>
        <el-main :style="{background: widgetForm.column.length == 0 ? `url(${widgetEmpty}) no-repeat 50%`: ''}">
          <widget-form ref="widgetForm"
                       :data="widgetForm"
                       :select.sync="widgetFormSelect"></widget-form>
        </el-main>
      </el-container>
      <!-- 右侧配置 -->
      <el-aside class="widget-config-container"
                :width="asideRightWidth">
        <el-tabs v-model="configTab"
                 stretch>
          <el-tab-pane label="字段属性"
                       name="widget"
                       style="padding: 0 10px;">
            <widget-config :data="widgetFormSelect"></widget-config>
          </el-tab-pane>
          <el-tab-pane label="表单属性"
                       name="form"
                       lazy
                       style="padding: 0 10px;">
            <form-config :data="widgetForm"></form-config>
          </el-tab-pane>
        </el-tabs>
      </el-aside>
      <!-- 弹窗 -->
      <!-- 导入JSON -->
      <el-drawer title="导入JSON"
                 :visible.sync="importJsonVisible"
                 size="50%"
                 append-to-body
                 destroy-on-close>
        <v-json-editor v-model="importJson"
                       :options="{ mode: 'code' }"
                       height="82vh"></v-json-editor>
        <div class="drawer-foot">
          <el-button size="medium"
                     type="primary"
                     @click="handleImportJsonSubmit">确定</el-button>
          <el-button size="medium"
                     type="danger"
                     @click="importJsonVisible = false">取消</el-button>
        </div>
      </el-drawer>
      <!-- 生成JSON -->
      <el-drawer title="生成JSON"
                 :visible.sync="generateJsonVisible"
                 size="50%"
                 append-to-body
                 destroy-on-close>
        <v-json-editor v-model="widgetFormPreview"
                       :options="{ mode: 'code' }"
                       height="82vh"></v-json-editor>
        <div class="drawer-foot">
          <el-button size="medium"
                     type="primary"
                     @click="handleGenerate">生成</el-button>
          <el-popover placement="top"
                      trigger="hover"
                      popper-class="popper-bo"
                      width="250px">
            <el-button size="medium"
                       type="primary"
                       slot="reference"
                       @click="handleCopy"
                       style="margin-left: 10px;">复制</el-button>
            <div>
              <el-form label-width="180px"
                       label-position="left">
                <el-alert :closable="false">
                  在没有开启美化的情况下，当前编辑器内可见的文本，就是复制得到的内容。<br>
                  如有需要，您可以开启美化，然后选取适合自己的美化配置。
                  <a href="https://www.npmjs.com/package/csvjson-json_beautifier"
                     target="_blank">参考资料</a>
                </el-alert>
                <el-form-item label="是否开启美化">
                  <el-switch v-model="beautifierOptions.enabled" />
                </el-form-item>
                <el-form-item label="缩进长度-空格数量">
                  <el-slider v-model="beautifierOptions.space"
                             show-stops
                             :marks="{ 1: '1', 2: '2', 3: '3', 4: '4' }"
                             :min="1"
                             :max="4"
                             :step="1"></el-slider>
                </el-form-item>
                <el-form-item label="引号类型">
                  <el-switch v-model="beautifierOptions.quoteType"
                             active-value="single"
                             inactive-value="double"
                             active-text="单引号"
                             inactive-text="双引号"></el-switch>
                </el-form-item>
                <el-form-item label="移除key的引号">
                  <el-switch v-model="beautifierOptions.dropQuotesOnKeys"></el-switch>
                </el-form-item>
                <el-form-item label="移除数字字符串的引号">
                  <el-switch v-model="beautifierOptions.dropQuotesOnNumbers"></el-switch>
                </el-form-item>
              </el-form>
            </div>
          </el-popover>
        </div>
      </el-drawer>
      <!-- 预览 -->
      <el-drawer title="预览"
                 :visible.sync="previewVisible"
                 size="60%"
                 append-to-body
                 :before-close="handleBeforeClose">
        <avue-form v-if="previewVisible"
                   ref="form"
                   class="preview-form"
                   :option="widgetFormPreview"
                   v-model="widgetModels"
                   @submit="handlePreviewSubmit"></avue-form>
        <div class="drawer-foot">
          <el-button size="medium"
                     type="primary"
                     @click="handlePreviewSubmit">确定</el-button>
          <el-button size="medium"
                     type="danger"
                     @click="handleBeforeClose">取消</el-button>
        </div>
      </el-drawer>
    </el-container>
  </div>
</template>

<script>
import fields from './fieldsConfig.js'
import { stringify } from './utils'
import beautifier from './utils/json-beautifier'
import widgetEmpty from './assets/widget-empty.png'

import Draggable from 'vuedraggable'
import VJsonEditor from 'v-jsoneditor'

import WidgetForm from './WidgetForm'
import FormConfig from './FormConfig'
import WidgetConfig from './WidgetConfig'

export default {
  name: "FormDesign",
  components: { Draggable, VJsonEditor, WidgetForm, FormConfig, WidgetConfig },
  props: {
    options: {
      type: Object,
      default: () => {
        return {
          column: []
        }
      }
    },
    storage: {
      type: Boolean,
      default: false
    },
    asideLeftWidth: {
      type: [String, Number],
      default: '270px'
    },
    asideRightWidth: {
      type: [String, Number],
      default: '380px'
    },
    showAvueDoc: {
      type: Boolean,
      default: false
    }
  },
  watch: {
    widgetForm: {
      handler (val) {
        if (this.storage) {
          if (val.column && val.column.length > 0) localStorage.setItem('avue-form', JSON.stringify(val))
          else localStorage.removeItem('avue-form')
        }
      },
      deep: true
    },
    beautifierOptions: {
      handler (val) {
        if (this.storage) {
          localStorage.setItem('avue-form-beautifier-options', JSON.stringify(val))
        }
      },
      deep: true
    },
    options: {
      handler (val) {
        this.transAvueOptionsToFormDesigner(val).then(res => {
          this.widgetForm = { ...this.widgetForm, ...res }
        })
      },
      deep: true
    }
  },
  computed: {
    leftWidth () {
      if (typeof this.asideLeftWidth == 'string') {
        return this.asideLeftWidth
      } else {
        return `${this.asideLeftWidth}px`
      }
    },
    rightWidth () {
      if (typeof this.asideRightWidth == 'string') {
        return this.asideRightWidth
      } else {
        return `${this.asideRightWidth}px`
      }
    }
  },
  data () {
    return {
      widgetEmpty,
      fields,
      widgetForm: {
        column: [],
        labelPosition: 'left',
        labelSuffix: '：',
        labelWidth: 120,
        gutter: 0,
        menuBtn: true,
        submitBtn: true,
        submitText: '提交',
        emptyBtn: true,
        emptyText: '清空',
        menuPosition: 'center'
      },
      widgetFormPreview: {},
      configTab: 'widget',
      widgetFormSelect: {},
      previewVisible: false,
      generateJsonVisible: false,
      importJsonVisible: false,
      importJson: {},
      widgetModels: {},
      configOption: {},
      beautifierOptions: {
        enabled: false,
        space: 2,
        quoteType: 'single',
        dropQuotesOnKeys: true,
        dropQuotesOnNumbers: false
      }
    }
  },
  mounted () {
    this.handleLoadStorage()
    this.loadBeautifierOptions()
    this.handleLoadCss()
  },
  methods: {
    // 组件初始化时加载本地存储中的options(需开启storage),若不存在则读取用户配置的options
    handleLoadStorage () {
      if (this.storage) {
        const form = localStorage.getItem('avue-form')
        if (form) this.transAvueOptionsToFormDesigner(JSON.parse(form)).then(data => this.widgetForm = data)
      } else this.transAvueOptionsToFormDesigner({ ...this.widgetForm, ...this.options }).then(data => this.widgetForm = data)
    },
    // 获取JSON格式化属性
    loadBeautifierOptions () {
      const bo = localStorage.getItem('avue-form-beautifier-options')
      if (bo) this.beautifierOptions = JSON.parse(bo)
    },
    handleLoadCss () {
      const url = '//at.alicdn.com/t/font_1254447_x280zepmf6.css'
      const link = document.createElement('link');
      link.rel = 'stylesheet';
      link.href = url;
      window.document.head.appendChild(link)
    },
    // Avue文档链接
    handleAvueDoc () {
      window.open('https://avuejs.com/doc/form/form-doc', '_blank')
    },
    // 预览 - 弹窗
    handlePreview () {
      if (!this.widgetForm.column || this.widgetForm.column.length == 0) this.$message.error("没有需要展示的内容")
      else {
        this.transformToAvueOptions(this.widgetForm).then(data => {
          this.widgetFormPreview = data
          this.previewVisible = true
        })
      }
    },
    // 导入JSON - 弹窗 - 确定
    handleImportJsonSubmit () {
      try {
        this.transAvueOptionsToFormDesigner(this.importJson).then(res => {
          this.widgetForm = res
          this.importJsonVisible = false
        })
      } catch (e) {
        this.$message.error(e.message)
      }
    },
    // 生成JSON - 弹窗
    handleGenerateJson () {
      this.transformToAvueOptions(this.widgetForm).then(data => {
        this.widgetFormPreview = data
        this.generateJsonVisible = true
      })
    },
    // 生成JSON - 弹窗 - 确定
    handleGenerate () {
      this.transformToAvueOptions(this.widgetForm).then(data => {
        this.$emit('submit', data)
      })
    },
    // 生成JSON - 弹窗 - 拷贝
    async handleCopy () {
      this.transformToAvueOptions(this.widgetForm).then(data => {
        let text;
        if (this.beautifierOptions.enabled) text = beautifier(data, this.beautifierOptions)
        else text = stringify(data)

        this.$Clipboard({
          text
        }).then(() => {
          this.$message.success('复制成功')
        }).catch(() => {
          this.$message.error('复制失败')
        });
      })
    },
    // 预览 - 弹窗 - 确定
    handlePreviewSubmit (form, done) {
      if (done) {
        this.$alert(this.widgetModels).then(() => {
          done()
        }).catch(() => {
        })
      } else {
        this.$refs.form.validate((valid, done) => {
          if (valid) this.$alert(this.widgetModels).then(() => {
            done()
          }).catch(() => {
          })
        })
      }
    },
    // 预览 - 弹窗 - 关闭前
    handleBeforeClose () {
      this.$refs.form.resetForm()
      this.$nextTick(() => this.previewVisible = false)
    },
    // 清空
    handleClear () {
      if (this.widgetForm && this.widgetForm.column && this.widgetForm.column.length > 0) {
        this.$confirm('确定要清空吗？', '警告', {
          type: 'warning'
        }).then(() => {
          this.$set(this.widgetForm, 'column', [])
          this.$set(this, 'widgetModels', {})
          this.$set(this, 'widgetFormSelect', {})
        }).catch(() => {
        })
      } else this.$message.error("没有需要清空的内容")
    },
    // 表单设计器配置项 转化为 Avue配置项
    transformToAvueOptions (obj) {
      return new Promise((resolve, reject) => {
        try {
          const data = this.deepClone(obj)
          for (let i = 0; i < data.column.length; i++) {
            const col = data.column[i]
            if (col.type == 'dynamic' && col.children && col.children.column && col.children.column.length > 0) {
              const c = col.children.column;
              c.forEach(item => {
                delete item.subfield
              })
              this.transformToAvueOptions(col.children).then(res => {
                col.children = res
              })
            } else if (col.type == 'group') {
              if (!data.group) data.group = []

              const group = {
                label: col.label,
                icon: col.icon,
                prop: col.prop,
                arrow: col.arrow,
                collapse: col.collapse,
                display: col.display
              }
              this.transformToAvueOptions(col.children).then(res => {
                group.column = res.column
                data.group.push(group)
              })
              data.column.splice(i, 1)
              i--
            } else if (['checkbox', 'radio', 'tree', 'cascader', 'select'].includes(col.type)) {
              if (col.dicOption == 'static') {
                delete col.dicUrl
                delete col.dicMethod
                delete col.dicQuery
              } else if (col.dicOption == 'remote') {
                delete col.dicData
                if (col.dicQuery && col.dicQuery.length > 0) {
                  const query = {}
                  col.dicQuery.forEach(q => {
                    if (q.key && q.value) query[q.key] = q.value
                  })
                  col.dicQuery = query
                } else delete col.dicQuery
              }
              delete col.dicOption
            } else if (['upload'].includes(col.type)) {
              if (col.headersConfig && col.headersConfig.length > 0) {
                const headers = {}
                col.headersConfig.forEach(h => {
                  if (h.key && h.value) headers[h.key] = h.value
                })
                col.headers = headers
              } else delete col.headers
              delete col.headersConfig

              if (col.dataConfig && col.dataConfig.length > 0) {
                const data = {}
                col.dataConfig.forEach(h => {
                  if (h.key && h.value) data[h.key] = h.value
                })
                col.data = data
              } else delete col.data
              delete col.dataConfig
            }
            if (col.change) col.change = eval(col.change)
            else delete col.change

            if (col.click) col.click = eval(col.click)
            else delete col.click

            if (col.focus) col.focus = eval(col.focus)
            else delete col.focus

            if (col.blur) col.blur = eval(col.blur)
            else delete col.blur
          }
          resolve(data)
        } catch (e) {
          reject(e)
        }
      })
    },
    // Avue配置项 转化为 表单设计器配置项
    transAvueOptionsToFormDesigner (obj) {
      const data = this.deepClone(obj)
      return new Promise((resolve, reject) => {
        try {
          if (data.column && data.column.length > 0) {
            data.column.forEach(col => {
              if (col.type == 'dynamic' && col.children && col.children.column && col.children.column.length > 0) {
                const c = col.children.column;
                c.forEach(item => {
                  item.subfield = true
                })
                this.transAvueOptionsToFormDesigner(col.children).then(res => {
                  col.children = res
                })
              } else if (['checkbox', 'radio', 'tree', 'cascader', 'select'].includes(col.type)) {
                if (!col.dicData && col.dicQuery && typeof col.dicQuery == 'object') {
                  const arr = []
                  for (let key in col.dicQuery) {
                    arr.push({
                      key,
                      value: col.dicQuery[key],
                      $cellEdit: true
                    })
                  }
                  col.dicQuery = arr
                }
                if (col.dicUrl) col.dicOption = 'remote'
                else col.dicOption = 'static'
                if (!col.dicData) col.dicData = []
              } else if (['upload'].includes(col.type)) {
                if (col.headers && typeof col.headers == 'object') {
                  const arr = []
                  for (let key in col.headers) {
                    arr.push({
                      key,
                      value: col.headers[key],
                      $cellEdit: true
                    })
                  }
                  col.headersConfig = arr
                } else col.headersConfig = []

                if (col.data && typeof col.data == 'object') {
                  const arr = []
                  for (let key in col.data) {
                    arr.push({
                      key,
                      value: col.data[key],
                      $cellEdit: true
                    })
                  }
                  col.dataConfig = arr
                } else col.dataConfig = []
              }
            })
          }
          if (data.group && data.group.length > 0) {
            for (let i = 0; i < data.group.length; i++) {
              if (!data.column) data.column = []
              const col = data.group[i]

              const group = {
                type: 'group',
                label: col.label,
                icon: col.icon,
                prop: col.prop,
                arrow: col.arrow,
                collapse: col.collapse,
                display: col.display
              }
              this.transAvueOptionsToFormDesigner(col).then(res => {
                group.children = res
                data.column.push(group)
              })
            }
            delete data.group
          }
          resolve(data)
        } catch (e) {
          reject(e)
        }
      })
    }
  }
}
</script>

<style lang="scss">
@import "./styles/index.scss";
</style>
