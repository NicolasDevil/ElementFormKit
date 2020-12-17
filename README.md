# form-combination组件
基于vue-element表单组件进行的组合封装，通过数据流方式将表单数据化，项目基于vue-elementUI
## 参数使用说明

### components Attributes
| 参数         | 说明    |    类型   |  默认值  | 
| -------- | :-----  |  :----:  |  :----: |
| config |  表单配置项，详细config配置参数参考下方config配置表  |  Array  |   空数组
| labelPosition |  表单项对齐规则,和elementUI的label-position一样  |  String  |   top  
| labelWidth |  表单项标题宽度（此参数仅在labelPosition为left、right时生效，为top时会自动忽略）  |  Number  |   125 
| columns |  每行显示多少列表单项  |  Number  |   5  
| title |  表单项头标题  |  String  |   null
| size |  表单内子项目元素尺寸（可选值为medium / small / mini）  |  String  |   mini 

### config Attributes
| 参数         |  说明  |  可选值  |   类型   |  默认值  |  实例
| -------- | :-----:  |  :----: |  :----:   |  :----: | :----: |
| label |  该表单项名称  |   -   |  String  |   -   |  -
| type |  该表单项类型  |   text（文本）、input、select、textarea、checkbox、date（日期选择器）、daterange（日期区间选择器）、region(地区选择器)   |  String  |   -   |  -
| placeholder |  该表单项placeholder  |   -   |  String  |   -   |  -
| disabled |  该表单项是否禁用  |   true、false   |  boolean  |   -   |  -
| keys |  表单项key值(该项应该和后台返回该表单项的字段对应，方便将修改后的数据与后台直接交互)  |   -   |  String  |   -   |  -
| startkeys |  日期区间选择器字段：开始时间key值  |   -   |  String  |   -   |  -
| endkeys |  日期区间选择器字段：结束时间key值  |   -   |  String  |   -   |  -
| keydata |  此属性仅在type为select和checkbox时生效，它将作为该select项拉取数据（getDictsList接口，根据业务需要替换）时需要传输的参数；checkbox时它作为数据来源  |   -   |  String  |   -   |  `{label: "多选:", type: 'checkbox', keydata: [{ label: "医疗保险", id: 1 }]}`
| rules |  表单项校验规则，不填则不校验  |   -   |  Array  |   空数组   |  -
| inline |  该表单项是否在新的一行整行显示（注意当type为checkbox时默认为true）  |   true、false   |  boolean  |   -   |  -
| $remote |  该表单项需要进行远程数据加载的自定义请求  |   -   |  Promise  |   -   |  `{$remote: ()=> import { selectCardNo } from "@/api/contract"}`
| props |  该表单项进行远程数据加载的自定义请求时指定选项的值为选项对象的某个属性值  |   label/value   |  Object  |   -   |  `{props: { label: 'name', value: 'id' }}`
| remoteKey |  该表单项需要进行远程数据加载自定义请求时需要的参数，该remoteKey需要应该在组件v-model绑定数据集存在  |   -   |  String  |   -   |  `{remoteKey: "custId"}`
| link |  该表单项显示需要关联的字段  |   -   |  Object  |   -   |  `{link: { key: "showid", value: 0 }}`表示表单内字段`showid`的值为0时该项不显示




