<template>
  <div class="cardsform" v-loading="isload" :class="{'title-card': !title}">
    <el-card :shadow="title ? 'hover' : 'never'" class="card-title">
      <h5 slot="header" v-if="title"><i class="el-icon-info"></i> {{title}}</h5>
      <el-form
        ref="cardsform"
        :model="FormData"
        v-if="config && config.length !== 0"
        :inline="true"
        :label-position="labelPosition"
        :size="size">
        <div class="form-item" :style="`grid-template-columns: repeat(${columns}, 1fr)`">
          <el-form-item
            :label="configs.label"
            v-for="(configs, index) in config.filter(ele => confController(ele))"
            :class="{ 'gutters-line': configs.inline }"
            :key="index"
            :label-width="labelWidth + 'px'"
            :prop="configs.keys"
            :rules="configs.rules">
            <div v-if="configs.type === 'text'">
              <span v-if="configs.keydata">{{showDict(configs.keydata, FormData[configs.keys])}}</span>
              <span v-else>{{FormData[configs.keys] || "暂无数据"}}</span>
            </div>
            <div v-if="configs.type === 'textarea' && true">
              <el-input
                type="textarea"
                v-model="FormData[configs.keys]"
                :disabled="configs.disabled || false"
                class="card-textarea"
                :autosize="{ minRows: 2, maxRows: 4}"
                :placeholder="configs.placeholder"/>
            </div>
            <div v-if="configs.type === 'input'">
              <el-input
                type="text"
                class="public-style"
                v-model="FormData[configs.keys]"
                :disabled="configs.disabled || false"
                :placeholder="configs.placeholder"/>
            </div>
            <div v-if="configs.type === 'select'">
              <el-select
                v-model="FormData[configs.keys]"
                filterable
                @change="FormChange($event, configs)"
                :disabled="configs.disabled || false"
                :placeholder="configs.placeholder"
                class="public-style">
                <el-option
                  v-for="item in configs.hasOwnProperty('$remote') && Array.isArray(configs.$remote) ? configs.$remote : selectData[configs.keydata] || []"
                  :key="item.dictValue"
                  :label="item.dictLabel"
                  :value="item.dictValue">
                </el-option>
              </el-select>
            </div>
            <div v-if="configs.type === 'date'">
              <el-date-picker
                type="date"
                :disabled="configs.disabled || false"
                v-model="FormData[configs.keys]"
                format="yyyy-MM-dd"
                class="public-style"
                :placeholder="configs.placeholder"
                value-format="yyyy-MM-dd">
              </el-date-picker>
            </div>
            <div v-if="configs.type === 'daterange'">
              <el-date-picker
                v-if="configs.hasOwnProperty('startkey' && 'endkey')"
                type="daterange"
                v-model="DataRange[configs.startkey]"
                format="yyyy-MM-dd"
                :disabled="configs.disabled || false"
                value-format="yyyy-MM-dd"
                start-placeholder="开始时间"
                end-placeholder="结束时间"
                range-separator="至"
                :style="{width: '235px'}"
                @change="daterangeChange($event, configs)">
              </el-date-picker>
              <span class="text-warn" v-else>需要配置startkeys和endkeys</span>
            </div>
            <div v-if="configs.type === 'checkbox' && FormData[configs.keys]">
              <el-checkbox-group v-model="FormData[configs.keys]" :disabled="configs.disabled || false">
                <el-checkbox v-for="(items, index) in configs.keydata || []" :key="index" :label="items.id">{{items.label}}</el-checkbox>
              </el-checkbox-group>
            </div>
            <div v-if="configs.type === 'region'">
              <el-cascader
                v-if="regionData.length !== 0"
                :ref="configs.keys"
                filterable
                v-model="FormData[configs.keys]"
                :disabled="configs.disabled || false"
                :props="{ value: 'id', label: 'label', children: 'children' }"
                class="public-style"
                :show-all-levels="true"
                @change="regionChange(FormData[configs.keys], configs)"
                :options="regionData">
              </el-cascader>
              <el-divider content-position="left" v-else>
                <span v-if="loadedMap.region" class="no-text"><i class="el-icon-loading"></i> &nbsp;正在加载地区数据</span>
                <span class="no-text" v-else>暂无地区数据</span>
              </el-divider>
            </div>
            <div v-if="configs.type === 'cascader'">
              <el-cascader
                v-if="typeof configs.options !== 'function' && configs.options !== 0"
                :ref="configs.keys"
                filterable
                v-model="FormData[configs.keys]"
                @change="fixedPinterclearValidate(configs)"
                :disabled="configs.disabled || false"
                :props="{ value: 'id', label: 'label', children: 'children' }"
                class="public-style"
                :show-all-levels="true"
                :options="configs.options || []">
              </el-cascader>
              <el-divider content-position="left" v-else>
                <span class="no-text"><i class="el-icon-loading"></i> &nbsp;正在加载数项</span>
                <!-- <span class="no-text" v-else>暂无数据项</span> -->
              </el-divider>
            </div>
          </el-form-item>
          <slot name="item"/>
        </div>
      </el-form>
      <slot name="content"/>
    </el-card>
  </div>
</template>

<script>
/**
 * card-form 参数使用说明
 * @param {config} 【必填】表单配置项，该项应为数组类型
 * @param {config.label} 表单项名称
 * @param {config.type} 表单项类型【text（文本）、input、select、textarea、checkbox、date（日期选择器）、daterange（日期区间选择器）、region(地区选择器)】
 * @param {config.placeholder} 原生input placeholder
 * @param {config.disabled} 是否禁用
 * @param {config.keys} 表单项key值，该项应该和后台返回该表单项的字段对应，方便将修改后的数据与后台直接交互
 * @param {config.startkeys || config.endkeys} 日期区间选择器项key值，考虑到区间选择是个数组所以需要指定2个字段接收；仅在type=daterange时生效
 * @param {config.keydata} 此属性仅在type为select和checkbox时生效，它将作为该select项拉取数据（getDictsList接口）时需要传输的参数
 * @param {config.rules} 表单项校验规则，不填则无
 * @param {config.inline} 该表单项是否在新的一行整行显示（true、false）
 * @param {labelPosition} 表单项对齐规则（left、right、top）
 * @param {labelWidth} 表单项标题宽度，数字类型，不填默认为125px，此参数仅在labelPosition为left、right时生效，为top时会自动忽略
 * @param {columns} 每行显示多少列表单项，默认5列
 * @param {title} 表单项头标题
 * @param {size} 表单内子项目元素尺寸
 * 完整实例： 
 * <card-form
 *    :config="[{ label: "客户号:", type: 'input', placeholder: "请输入客户号", disabled: false, keys: "aa", rules: [{ required:true, message:'行内客户号不能为空'}] }]" 
 *    v-model="base"
 *    :columns="4"
 *    size="mini"
 *    title="担保信息">
 * </card-form>
 * 
 * -------- 关于组件插槽的使用 --------
 *  只设置了2个插槽，slot="content"和slot="item"，在设置插槽时v-model你任然需要加上，你也可以不写，其实影响不大，但页面会报红
 *  item插槽： 此插槽会将你的内容和表单项并级
 *  content插槽： 此插槽会将你的内容和整个表单并级
 *  实例（content插槽）：
 *  <card-form v-model="base" title="标题">
 *     <template slot="content">
 *        <div>你的内容</div>
 *     </template>
 *  </card-form>
 * -------- 关于组件插槽的使用 --------
 * 
 * 注意：当config项中存在checkbox配置项，你需要在组件v-model绑定的变量中初始化为一个键为checkbox的keys，值为空数组，因为checkbox只能接受数组类型
 * 备注1：使用时加上refs="cardForm"参数（注意在重复使用组件时refs的值不能重复），组件内的所有方法在组件外都可使用例如：this.$refs.cardForm[组件内方法名]
 * 备注2：组件加载时已对config参数进行遍历操作，如需要对参数进行特殊处理应在_initcomponent类内新增构造函数，在init方法内执行以避免不必要的重复遍历
 */
import { getDictsList, getProvincesCity } from "@/api/loanContract/contract";
export default {
  name: "cardForm",
  props: {
    value: { required: true },
    config: {
      type: Array,
      default: function() {
        return []
      }
    },
    labelPosition: {    // 表单项对齐规则（left、right、top）
      type: String,
      default: "top"
    },
    labelWidth: {    // 表单项标题宽度，不填默认为125px，此参数仅在labelPosition为left、right时生效
      type: Number,
      default: 125
    },
    columns: {  // 每行显示多少列，默认5列
      type: Number,
      default: 5
    },
    title: {    // 表单项头标题项
      type: String,
      default: null
    },
    size: {     // 表单项所有元素尺寸
      type: String,
      default: "mini"
    }
  },
  data () {
    return {
      isload: false,
      loadedMap: { region: true },
      treeselectLoaded: false,
      regionData: [],
      checkboxs: {},
      selectData: {},
      DataRange: {}  // 时间区间选择器数据
    }
  },
  computed: {
    FormData: {
      get() {
        return this.value || {}
      },
      set(newVal) {
        this.$emit("change", newVal);
      }
    }
  },
  created() {
    setTimeout(() => { this._initcomponent() }, 1500)
  },
  methods: {
    _initcomponent() {
      const config = this.config, THIS = this;
      return new class {
        constructor() {
          this.keys = { keydata: [], region: [] }
          const setConf = new Set(config)
          if(setConf.size > 0) this.init(setConf), THIS.loaded = true;
        }
        init(_confs, FUNC = ["getKeys", "daterange", "handelcheckbox", "remote", "cascaders"]) {    // 初始化需要针对config处理的逻辑在此处执行
          try {
            for (const iterator of _confs) FUNC.map(name => this[name] ? this[name](iterator) : null);
            this.requests(["selects_get", "region_get"])
          } catch (error) {
            console.warn(error)
            [THIS.isload, this.loadedMap.region] = [false, false]
          }
        }
        requests(FUNC = []) {
          if(FUNC.length !== 0) FUNC.map(name => THIS[name] ? THIS[name](this.keys) : null);
        }
        getKeys(config) {
          if(config.hasOwnProperty('keydata') && typeof config.keydata === 'string' && config.keydata !== '') this.keys.keydata.push(config.keydata);
          if(config.hasOwnProperty('type') && config.type === 'region') this.keys.region.push(config.keys);
        }
        remote(config) {   // 远程搜索方法开始检索远程数据
          if(config.hasOwnProperty('$remote') && typeof config.$remote === 'function') {
            config.$remote(THIS.FormData[config.remoteKey] || null).then(res => {
              if(res.code === 200) {
                if(config.hasOwnProperty('props') && typeof config.props === 'object') {
                  config.$remote = THIS.dataHandel(res.data, config.props.label, config.props.value)
                } else {
                  config.$remote = res.data
                }
              };
            })
          }
        }
        cascaders(config) {   // 获取cascader option值
          if(config.hasOwnProperty('type' && 'options') && config.type === 'cascader' && typeof config.options === 'function') {
            const _options = config.options
            _options().then(res => {
              if(res.code === 200) config.options = res.data;
            })
          }
        }
        daterange(config) {   // 处理时间区间选择器数据显示
          if(config.hasOwnProperty('startkey' && 'endkey' && 'type') && config.type === 'daterange') {
            THIS.DataRange = { ...THIS.DataRange, [config.startkey]: [THIS.FormData[config.startkey] || '', THIS.FormData[config.endkey] || ''] }
          }
        }
        handelcheckbox(config) {  // 处理checkbox数据
          if(config.hasOwnProperty('type') && config.type === "checkbox") {
            const _value = THIS.FormData[config.keys]
            THIS.FormData[config.keys] = _value.map(Number)
          };
        }
      }
    },
 
    regionChange(values, config) {    // 监控地区选择器数据(labelkey作为获取地区文字的key值，特殊业务时作为绑定地区文字的字段)
      const labelKey = config.labelkey || null;
      if(labelKey && labelKey !== "" && this.$refs[config.keys][0]) {
        this.fixedPinterclearValidate(config);
        const _labels = this.$refs[config.keys][0].getCheckedNodes()[0].pathLabels;
        this.FormData[labelKey] = _labels.length !== 0 ? _labels.join("/") : '';
      }
    },
    showDict(dict, value) {   // 根据字典值获取、显示字典数据
      const _dict = this.selectData[dict] || [];
      return _dict.find(ele => ele.dictValue == value) ? _dict.find(ele => ele.dictValue == value).dictLabel : "暂无数据"
    },

    daterangeChange(value, config) {    // 监控时间区间选择器数据
      if(value && value.length !== 0) {
        this.FormData = Object.assign(this.FormData, { [config.startkey]: value[0], [config.endkey]: value[1] })
        this.fixedPinterclearValidate(config)
      } else {
        this.FormData = Object.assign(this.FormData, { [config.startkey]: null, [config.endkey]: null })
      }
    },

    FormChange(value, config) {   // 监控表单数据更改
      this.fixedPinterclearValidate(config)
      if(value !== undefined && value.trim() !== "") this.$emit("change", { value, ...config });
    },
    fixedPinterclearValidate(config = {}) {    // 定点清除表单项校验
      if(config.hasOwnProperty('keys' && 'rules') && this.$refs.cardsform) this.$refs.cardsform.clearValidate([config.keys]);
    },

    confController(config) {   // 校验配置项中是否包含link关联
      if(config.hasOwnProperty("link")) {
        const { value, key } = config.link
        this.fixedPinterclearValidate(config);
        if(this.FormData[key] != value) return config;
      } else {
        return config
      }
    },
    dataHandel(data = [], label, keys) {
      if(data.length !== 0) {
        const handel = data.map(item => ({
          dictLabel: item[label] || null,
          dictValue: item[keys] || null
        }))
        return handel
      } else {
        return []
      }
    },

    /********** 组件基础数据信息获取 **********/
    region_get(keys) {
      const keysArray = keys.region || [];
      if(keysArray.length > 0) {
        this.loadedMap.region = true
        getProvincesCity().then(res => {
          const rxd = res.data
          if(rxd && rxd.length !== 0) this.regionData = [rxd];
          this.loadedMap.region = false
        })
      }
    },
    selects_get(keys) {
      const keysArray = keys.keydata || [];
      if(keysArray.length !== 0) {
        this.isload = true
        getDictsList(Array.from(new Set(keysArray))).then(res => {
          this.isload = false
          this.selectData = res.data
        })
      }
    }
    /********** 组件基础数据信息获取 **********/
  }
};
</script>

<style lang="scss" scoped>
.titles {
  margin-bottom: 12px;
}
.card-textarea {
  width: 460px;
}
h5 {
  margin: 0;
}
.title-card {
  /deep/ .el-card {
    border: none;
  }
}
.cardsform {
  .card-title {
    margin-bottom: 20px;
  }
  .form-item {
    display: grid;
    grid-column-gap: 15px;
    align-items: center;
    .public-style {
      width: 210px;
    }
  }
  .gutters-line {
    grid-column: 1/-1;
  }
  .full-line {
    width: 100%;
  }
  .text-warn {
    font-size: 12px;
    color: #aaa;
  }
  /deep/ .el-card__header {
    padding: 15px 20px;
    border-bottom: none;
    background: #f0f0f0;
  }
  /deep/ .el-card__body {
    padding: 15px 20px 0 20px;
  }
  .no-text {
    color: #ccc;
  }
}
</style>