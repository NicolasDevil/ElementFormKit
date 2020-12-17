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
| -------- | :-----  |  :----: |  :----:   |  :----: | :----: |
| label |  该表单项名称  |   -   |  String  |   -   |  -
| type |  该表单项类型  |   -   |  String  |   -   |  -
| placeholder |  该表单项placeholder  |   -   |  String  |   -   |  -
| disabled |  该表单项是否禁用  |   true、false   |  boolean  |   -   |  -



