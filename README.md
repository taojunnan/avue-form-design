## 简介

本项目是一款基于 <a href="https://avuejs.com/" target="_blank">Avue</a> 的表单设计器，拖拽式操作让你快速构建一个表单。
[在线预览](https://form.beta.kim/)

如果有任何使用上的问题，QQ群 1083980136

## 更新日志
[更新日志](https://github.com/sscfaith/avue-form-design/blob/master/CHANGELOG.md)

## 依赖

element-ui 2.13.2+

```sh
$ npm i element-ui
```

@smallwei/avue 2.6.11+

```sh
$ npm i @smallwei/avue
```

或自行引入cdn

## 安装

### 组件

```sh
$ npm i @sscfaith/avue-form-design
或
$ yarn add @sscfaith/avue-form-design
```

### 源码

<a href="https://github.com/sscfaith/avue-form-design" target="_blank">github</a>

```sh
$ yarn
$ yarn serve
```

## 使用

```
import AvueFormDesign from '@sscfaith/avue-form-design'

Vue.use(AvueFormDesign)
```

```
<avue-form-design style="height: 86vh;"
                  :options="options"
                  :aside-left-width="270"
                  :aside-right-width="380"
                  storage
                  @submit="handleSubmit"></avue-form-design>
```

### 属性

| 参数 | 说明 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| options | 字段配置 | Object | { column: [] } |
| storage | 开启本地存储功能，防止浏览器刷新丢失json | Boolean | false |
| asideLeftWidth | 左工具栏宽度 | String/Number | '270px' |
| asideRightWidth | 右工具栏宽度 | String/Number | '380px' |
| showAvueDoc | 是否显示Avue文档 | Boolean | false |

### options字段配置

<a href="https://avuejs.com/doc/form/form-doc" target="_blank">Avue文档</a>

| 属性 | 说明 | 类型 | 可选值 | 默认值 |
| ------| ------ | ------ | ------ | ------ |
| column | Avue字段 | Array | - | [] |
| labelPosition | 字段位置 |  String | 'left'/'center'/'right' | 'left' |
| labelWidth | 字段宽度 | Number | - | 120 |
| gutter | 字段间隔 | Number | - | 0 |
| menuBtn | 表单按钮 | Boolean | true/false | false |
| submitBtn | 显示提交按钮 | Boolean | true/false | false |
| submitText | 提交按钮文本 | String | - | '提交' |
| emptyBtn | 显示清空按钮 | Boolean | true/false | false |
| emptyText | 清空按钮文本 | String | - | '清空' |
| tabs | 多分组转标签 | Boolean | true/false | false |
| detail | 详情模式 | Boolean | true/false | false |
| readonly | 全局只读 | Boolean | true/false | false |
| disabled | 全局禁用 | Boolean | true/false | false |

### 事件

| 名称 | 说明 | 回调参数 |
| ------ | ------ | ------ |
| submit | 生成json回调 | 当前配置的json |

### Avue插件

#### 富文本

```
import AvueUeditor from 'avue-plugin-ueditor'

Vue.use(AvueUeditor)
```

## 打包

### 组件

```sh
$ yarn lib
```

### 源码

```sh
$ yarn build
```

## 捐赠
如果你觉得本项目帮助到你的话，可以给作者买杯咖啡。
<img src="http://112.74.43.150/images/donate.png">

## License

<a href="https://opensource.org/licenses/MIT" target="_blank">MIT</a>
