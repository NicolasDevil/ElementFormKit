<template>
  <div v-loading="isLoad.page">
    <div class="container">
      <el-tabs v-model="viewActive" class="margin-bottom">
        <!------------------- 调查信息模块 ------------------->
        <el-tab-pane name="first" class="view-container" :disabled="isLoad.save">
          <div slot="label" class="tabsHeader">调查信息</div>
          <div>
            <el-alert
              title="*说明：请根据贷款调查情况，核对并完善客户信息。"
              v-if="isuser"
              type="warning"
              show-icon
              close-text="知道了">
            </el-alert>
            <el-alert
              title="用户不存在"
              description="请去柜面开会，未开户期间您将无法操作此合同信息"
              v-else
              type="error"
              show-icon
              close-text="知道了">
            </el-alert>
          </div>
          <el-card class="box-card" shadow="never" v-if="rxdData.Cust">
            <h4 slot="header">客户基本信息</h4>
            <card-form :config="formSetting.custombase" v-model="rxdData.Cust.xdCustBase" ref="custombase" v-if="rxdData.Cust.xdCustBase"></card-form>
            <card-form :config="formSetting.mateinfo" v-model="rxdData.Cust.xdCustSocialRelations" title="配偶信息" :columns="4" ref="mateinfo" v-if="rxdData.Cust.xdCustSocialRelations"></card-form>
            <card-form :config="formSetting.familyinfo" v-model="rxdData.Cust.xdCustFamily" title="农户家庭情况" :columns="4" ref="familyinfo" v-if="rxdData.Cust.xdCustFamily"></card-form>
            <card-form :config="formSetting.manageinfo" v-model="rxdData.Cust.xdCustDealPriv" title="经营信息" :columns="4" ref="manageinfo" v-if="rxdData.Cust.xdCustDealPriv" v-loading="isLoad.DealPriv" element-loading-text="正在查询经营信息"></card-form>
            <card-form :config="formSetting.professioninfo" v-model="rxdData.Cust.xdCustProfession" title="职业信息" :columns="4" ref="professioninfo" v-if="rxdData.Cust.xdCustProfession"></card-form>
            <card-form title="地址信息" v-model="isLoad.defaults">
              <template slot="content">
                <div>
                  <el-table :data="rxdData.Cust.xdCustAddrInfos || []" :border="false" stripe>
                    <el-table-column prop="addrtypeName" label="地址类型" align="center"></el-table-column>
                    <el-table-column prop="natisign2" label="国家/地区" align="center">
                      <template slot-scope="{row}">
                        <span>{{row.natisign2 == "CHN" ? "中国" : row.natisign2}}</span>
                      </template>
                    </el-table-column>
                    <el-table-column prop="administrative" label="地址行政区划" align="center"></el-table-column>
                    <el-table-column prop="post" label="邮编" align="center"></el-table-column>
                    <el-table-column prop="addr" label="地址" align="center"></el-table-column>
                    <el-table-column label="操作" align="center">
                      <template slot-scope="scope">
                        <el-button type="text" icon="el-icon-edit" size="mini" @click="changeAddress(scope)">编辑</el-button>
                        <!-- <el-button type="danger" icon="el-icon-delete" size="mini" @click="() => rxdData.Cust.xdCustAddrInfos.splice(scope.$index, 1)"></el-button> -->
                      </template>
                    </el-table-column>
                  </el-table>
                  <!-- <div class="newAddress"><el-button type="warning" icon="el-icon-plus" class="addaddress-buttom" @click="changeAddress(null)">新增地址</el-button></div> -->
                </div>
              </template>
            </card-form>
          </el-card>

          <el-card class="box-card" shadow="never" v-if="rxdData.Contract">
            <h4 slot="header">贷款调查</h4>
            <el-tabs type="border-card" class="margin-bottom">
              <el-tab-pane label="贷款信息">
                <el-table :data="TableData.Credits.dkxx || []" :border="false" v-loading="isLoad.Credits">
                  <el-table-column prop="listid" label="借款编号" align="center"></el-table-column>
                  <el-table-column prop="custname" label="客户名称" align="center"></el-table-column>
                  <el-table-column prop="prodname" label="产品名称" align="center"></el-table-column>
                  <el-table-column prop="tcapi" label="贷款金额(元)" align="center" min-width="98"></el-table-column>
                  <el-table-column prop="bal" label="贷款余额(元)" align="center" min-width="98"></el-table-column>
                  <el-table-column prop="oberbal" label="拖欠本金(元)" align="center" min-width="98"></el-table-column>
                  <el-table-column prop="overinte" label="拖欠利息(元)" align="center" min-width="98"></el-table-column>
                  <el-table-column prop="begindate" label="放款日期" align="center"></el-table-column>
                  <el-table-column prop="enddate" label="到期日期" align="center"></el-table-column>
                  <el-table-column prop="assukind" label="担保方式" align="center"></el-table-column>
                  <el-table-column prop="interate" label="贷款利率" align="center"></el-table-column>
                  <el-table-column prop="riskflag" label="风险分类" align="center"></el-table-column>
                  <el-table-column prop="acflag" label="账户状态" align="center"></el-table-column>
                  <el-table-column prop="bankid" label="经办行" align="center"></el-table-column>
                  <el-table-column prop="operid" label="经办人" align="center"></el-table-column>
                </el-table>
              </el-tab-pane>
              <el-tab-pane label="对外担保情况" :disabled="TableData.Credits.dwdbqk && TableData.Credits.dwdbqk.length == 0">
                <el-table :data="TableData.Credits.dwdbqk || []" :border="false" v-loading="isLoad.Credits">
                  <el-table-column prop="assucontno" label="担保合同编号" align="center" width="110"></el-table-column>
                  <el-table-column prop="assucustid" label="担保人客户号" align="center" width="110"></el-table-column>
                  <el-table-column prop="assucustname" label="担保人名称" align="center" width="110"></el-table-column>
                  <el-table-column prop="assutype" label="担保方式" align="center" width="110"></el-table-column>
                  <el-table-column prop="assustatus" label="担保状态" align="center" width="110"></el-table-column>
                  <el-table-column prop="contno" label="合同编号" align="center" width="110"></el-table-column>
                  <el-table-column prop="custno" label="借款人客户号" align="center" width="110"></el-table-column>
                  <el-table-column prop="custname" label="借款人名称" align="center" width="110"></el-table-column>
                  <el-table-column prop="assuname" label="产品名称" align="center" width="110"></el-table-column>
                  <el-table-column prop="assucontamt" label="担保合同金额(元)" align="center" width="130"></el-table-column>
                  <el-table-column prop="useassuamt" label="本次使用担保金额(元)" align="center" width="150"></el-table-column>
                  <el-table-column prop="recvassuamt" label="已恢复担保金额" align="center" width="150"></el-table-column>
                  <el-table-column prop="begindate" label="担保起始日期" align="center" width="110"></el-table-column>
                  <el-table-column prop="enddate" label="担保到期日期" align="center" width="110"></el-table-column>
                  <el-table-column prop="bankid" label="经办行" align="center" width="110"></el-table-column>
                  <el-table-column prop="operid" label="经办人" align="center" width="110"></el-table-column>
                </el-table>
              </el-tab-pane>
              <el-tab-pane label="未解除借款合同" :disabled="TableData.Credits.wjcjkht && TableData.Credits.wjcjkht.length == 0">
                <el-table :data="TableData.Credits.wjcjkht || []" :border="false" v-loading="isLoad.Credits">
                  <el-table-column prop="contno" label="合同编号" align="center"></el-table-column>
                  <el-table-column prop="custid" label="客户编号" align="center"></el-table-column>
                  <el-table-column prop="custname" label="客户名称" align="center"></el-table-column>
                  <el-table-column prop="prodid" label="产品代码" align="center"></el-table-column>
                  <el-table-column prop="prodname" label="产品名称" align="center"></el-table-column>
                  <el-table-column prop="conttotalamt" label="合同金额(元)" align="center"></el-table-column>
                  <el-table-column prop="conttouseamt" label="合同可用余额(元)" align="center" width="130"></el-table-column>
                  <el-table-column prop="contterm" label="合同期限(月)" align="center"></el-table-column>
                  <el-table-column prop="assukind" label="担保方式" align="center"></el-table-column>
                  <el-table-column prop="begindate" label="起始日期" align="center"></el-table-column>
                  <el-table-column prop="enddate" label="到期日期" align="center"></el-table-column>
                  <el-table-column prop="contstatus" label="合同状态" align="center"></el-table-column>
                  <el-table-column prop="bankid" label="经办行" align="center"></el-table-column>
                  <el-table-column prop="operid" label="经办人" align="center"></el-table-column>
                </el-table>
              </el-tab-pane>
            </el-tabs>
            <card-form :config="formSetting.creditrecord" v-model="rxdData.Contract.xdContracts" :columns="5" label-position="right" class="margin-bottom" v-if="rxdData.Contract.xdContracts"></card-form>
            <card-form :config="formSetting.eloanCard" v-model="rxdData.Contract.xdContracts" title="合同基本信息" :columns="4" ref="eloanCard" v-if="rxdData.Contract.xdContracts">
              <!-- <template slot="item">
                <el-form-item label="银行卡卡号" :rules="[{ required: true, message: '银行卡卡号不能为空', trigger: 'blur', validate: LoanCardvalidate }]">
                  <el-divider content-position="left" v-if="isLoad.easyCard">
                    <span style="color: #ccc"><i class="el-icon-loading"></i> &nbsp;正在加载银行卡</span>
                  </el-divider>
                  <div v-else>
                    <el-select
                      v-model="rxdData.Contract.xdContracts.easyLoanCard"
                      style="width: 210px"
                      filterable
                      clearable
                      placeholder="请选择银行卡卡号"
                      v-if="cardData.length !== 0">
                      <el-option
                        v-for="item in cardData"
                        :key="item.id"
                        :label="item.name"
                        :value="item.id">
                      </el-option>
                    </el-select>
                    <el-input
                      v-else
                      type="text"
                      v-model="rxdData.Contract.xdContracts.easyLoanCard"
                      style="width: 210px"
                      placeholder="请输入银行卡卡号"/>
                  </div>
                </el-form-item>
              </template> -->
            </card-form>
            <card-form :config="formSetting.rateInfo" v-model="rxdData.Contract.rate" title="利率信息" :columns="4" ref="rateInfo" v-if="rxdData.Contract.rate"></card-form>
            <card-form :config="formSetting.guaranteeinfo" v-model="rxdData.Contract.assucontract" :columns="4" title="担保信息" label-position="right" class="margin-bottom" v-if="rxdData.Contract.assucontract" ref="guaranteeinfo"></card-form>
          </el-card>
          <!-- public modules views -->
          <div class="margin-bottom" v-if="rxdData.Contract && rxdData.Contract.impaInfo && rxdData.Contract.impaFwInfo">
            <el-card class="box-card margin-bottom" shadow="never" v-if="rxdData.Contract.impaInfo">
              <h4 slot="header">押品信息</h4>
              <card-form :config="formSetting.mortgageinfo" v-model="rxdData.Contract.impaInfo" :columns="4" ref="mortgageinfo"></card-form>
            </el-card>
            <el-card class="box-card" shadow="never" v-if="rxdData.Contract.impaFwInfo">
              <h4 slot="header">房产信息</h4>
              <card-form :config="formSetting.houseinfo" v-model="rxdData.Contract.impaFwInfo" :columns="4" ref="houseinfo"></card-form>
            </el-card>
          </div>
          <!-- public modules views -->
          <el-card class="box-card" shadow="never" v-if="rxdData.Contract && rxdData.Contract.approvalLists">
            <h4 slot="header">贷款调查处理意见</h4>
            <card-form v-model="isLoad.defaults">
              <template slot="content">
                <el-table :data="rxdData.Contract.approvalLists" border>
                  <el-table-column prop="contId" align="center" label="合同号"></el-table-column>
                  <el-table-column prop="userName" align="center" label="客户经理"></el-table-column>
                  <el-table-column prop="userOrgan" align="center" label="客户经理所属机构"></el-table-column>
                  <el-table-column prop="approvalTime" align="center" label="审批时间"></el-table-column>
                  <el-table-column prop="remark" align="center" label="审批意见"></el-table-column>
                </el-table>
              </template>
            </card-form>
          </el-card>
        </el-tab-pane>
        <!------------------- 调查信息模块 ------------------->
        
        <!------------------- 拟制合同模块 ------------------->
        <el-tab-pane name="second" class="view-container" :disabled="isLoad.save">
          <div slot="label" class="tabsHeader">拟制合同</div>
          <el-card class="box-card" shadow="never" v-if="rxdData.Contract && rxdData.Contract.xdContracts">
            <h4 slot="header">合同基本信息</h4>
            <card-form :config="formSetting.personalLoaninfo" v-model="rxdData.Contract.xdContracts" label-position="right" :label-width="140" ref="personalLoaninfo"></card-form>
            <card-form :config="formSetting.payinfo" v-model="rxdData.Contract.xdContracts" title="支付信息" :columns="4" label-position="right" class="margin-bottom"></card-form>
            <card-form :config="formSetting.repaymentinfo" v-model="rxdData.Contract.xdContracts" title="还款信息" :columns="4" label-position="right" class="margin-bottom"></card-form>
            <card-form :config="formSetting.otherinfo" v-model="rxdData.Contract.xdContracts" title="其他信息" :columns="4" label-position="right" class="margin-bottom" ref="otherinfo"></card-form>
          </el-card>
          <el-card class="box-card margin-bottom" shadow="never" v-if="rxdData.Contract && rxdData.Contract.assucontract">
            <h4 slot="header">担保合同信息</h4>
            <card-form :config="formSetting.guarantycontractinfo" v-model="rxdData.Contract.assucontract" label-position="right" :label-width="140" v-if="rxdData.Contract.assucontract"></card-form>
          </el-card>
          <!-- public modules views -->
          <div class="margin-bottom" v-if="rxdData.Contract && rxdData.Contract.impaInfo && rxdData.Contract.impaFwInfo">
            <el-card class="box-card margin-bottom" shadow="never" v-if="rxdData.Contract.impaInfo">
              <h4 slot="header">押品信息</h4>
              <card-form :config="formSetting.mortgageinfo" v-model="rxdData.Contract.impaInfo" :columns="4" ref="mortgageinfo2"></card-form>
            </el-card>
            <el-card class="box-card" shadow="never" v-if="rxdData.Contract.impaFwInfo">
              <h4 slot="header">房产信息</h4>
              <card-form :config="formSetting.houseinfo" v-model="rxdData.Contract.impaFwInfo" :columns="4" ref="houseinfo2"></card-form>
            </el-card>
          </div>
          <!-- public modules views -->
        </el-tab-pane>
        <!------------------- 拟制合同模块 ------------------->
      </el-tabs>

      <div class="view-container">
        <el-alert
          title="操作提示"
          description="*请仔细核对合同信息，合同签订后，客户即可通过手机银行执行放款操作。"
          type="warning"
          show-icon
          v-if="viewActive === 'second'"
          close-text="知道了">
        </el-alert>
      </div>

      <div class="fixed-buttom" v-if="isuser && pageState">
        <el-button type="primary" size="small" v-if="viewActive === 'first'" :loading="isLoad.save" @click="save" :disabled="pageState === 'EFF' || pageState === 'END'">
          <span v-if="pageState === 'ADD'">立即保存</span>
          <span v-if="pageState === 'EFF'">合同已提交</span>
          <span v-if="pageState === 'END'">合同已提交</span>
        </el-button>
        <el-button type="primary" size="small" v-if="viewActive === 'second'" :loading="isLoad.save" @click="orderin" :disabled="pageState === 'EFF' || pageState === 'END'">
          <span v-if="pageState === 'ADD'">签订合同</span>
          <span v-if="pageState === 'EFF'">合同已签订</span>
          <span v-if="pageState === 'END'">合同已签订</span>
        </el-button>
      </div>
    </div>

    <!-- 新增地址弹框 -->
    <el-dialog :title="addressDialog.formIdx == null ? '新增地址' : '编辑地址'" :visible.sync="addressDialog.visible" width="380px">
      <el-form
        ref="addressForm"
        :model="addressDialog.form"
        label-position="right"
        label-width="120px"
        size="mini">
        <el-form-item label="地址类型" prop="addrtype" :rules="[{ required: true, message: '地址类型不能为空', trigger: 'blur' }]">
          <el-select v-model="addressDialog.form.addrtype" filterable placeholder="请选择地址类型" @change="addressOption" class="public-style">
            <el-option
              v-for="item in addressSelect.contract_addrtype || []"
              :key="item.dictValue"
              :label="item.dictLabel"
              :value="item.dictValue">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="国家/地区" prop="natisign2" :rules="[{ required: true, message: '国家/地区不能为空', trigger: 'blur' }]">
          <el-select v-model="addressDialog.form.natisign2" filterable placeholder="请选择国家/地区" class="public-style">
            <el-option
              v-for="item in addressSelect.contract_natisign2 || []"
              :key="item.dictValue"
              :label="item.dictLabel"
              :value="item.dictValue">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="地址行政区划" prop="administrative" >
          <el-input type="text" v-model="addressDialog.form.administrative" placeholder="请选择地址行政区划" class="public-style"/>
        </el-form-item>
        <el-form-item label="邮编" prop="post" :rules="[{ required: true, message: '邮编不能为空', trigger: 'blur' }]">
          <el-input type="text" v-model="addressDialog.form.post" placeholder="请选择邮编" class="public-style"/>
        </el-form-item>
        <el-form-item label="地址" prop="addr" :rules="[{ required: true, message: '地址不能为空', trigger: 'blur' }]">
          <el-input type="text" v-model="addressDialog.form.addr" placeholder="请选择地址" class="public-style"/>
        </el-form-item>
      </el-form>
      <div solt="footer" class="dialogFooter">
        <el-button type="primary" class="addaddress-buttom" size="mini" @click="addressSave(addressDialog.form)">保 存</el-button>
        <el-button type="info" class="addaddress-buttom" @click="addressDialog.visible = false" size="mini">取 消</el-button>
      </div>
    </el-dialog>

    <el-backtop :bottom="90" :right="30"></el-backtop>
  </div>
</template>

<script>
import cardForm from "@/components/cardForm"
import config from "./config.js"
import { queryCustInfo, queryContractInfo, updateCustInfo, contractCust, updateContractInfo, getDictsList, getCreditHistory, submitContract, selectCardNo, getCompanyInfo } from "@/api/loanContract/contract";
export default {
  name: "bargainDetail",
  data () {
    return {
      isLoad: { page: false, save: false, defaults: {}, houseinfo: true, easyCard: false, Credits: false, DealPriv: false },
      isuser: true,
      viewActive: 'first',
      TableData: {
        Creditsload: false,
        Credits: {}
      },
      rxdData: {},
      cardData: [],
      formSetting: Object.assign(config),
      addressDialog: {
        visible: false,
        form: {},
        formIdx: null
      },
      addressSelect: {}
    }
  },
  async created() {
    try {
      this.GetList();
      const addressSel = await getDictsList(["contract_addrtype", "contract_natisign2"]);
      const _user = await contractCust(this.views.custId);   // 查询用户是否存在客户信息
      const Credits = await getCreditHistory(this.views.custId);  // 获取借款人信贷记录
      if(addressSel && addressSel.code === 200) this.addressSelect = addressSel.data;
      if(Credits && Credits.code === 200 && Object.keys(Credits.data) !== 0) this.TableData.Credits = Credits.data;
      if(_user.code === 200) {this.isuser = true} else {this.isuser = false};
    } catch (error) {
      console.log(error)
    }
  },
  methods: {
    GetList() {
      this.isLoad.page = true
      const CustInfo = queryCustInfo(this.views.custId), ContractInfo = queryContractInfo(this.views.contId);
      Promise.all([CustInfo, ContractInfo]).then(rxd => {
        console.log(rxd)
        const Cust = rxd[0].data || {}, Contract = rxd[1].data || {};
        this.rxdData = { Cust, Contract }
        if(Cust && Cust.xdCustBase) this.GeteasyCard(Cust.xdCustBase.paperid);
        if(Contract && Contract.xdContracts) this.rxdData.Contract = { ...Contract, xdContracts: { ...Contract.xdContracts, loanrate: Contract.rate.loanrate } };
        this.isLoad.page = false
      }).catch(err => {
        console.log(err)
        this.isLoad.page = false
      })
    },

    save() {
      const _refs = ['custombase', 'familyinfo', 'manageinfo', 'mateinfo', 'professioninfo', 'mortgageinfo', 'houseinfo', 'rateInfo', 'eloanCard', 'guaranteeinfo']
      this.componentsValidate(_refs).then(() => {
        const Cust = { ...this.rxdData.Cust }, Contract = { ...this.rxdData.Contract };
        this.saveRequest(Cust, Contract)
      })
    },
    async saveRequest(CustInfo, ContractInfo) {
      try {
        this.isLoad.save = true
        await updateCustInfo({
          ...CustInfo,
          xdCustBase: { ...CustInfo.xdCustBase, operid: ContractInfo.xdContracts.operid, bankid: ContractInfo.xdContracts.bankid }
        })
        await updateContractInfo(ContractInfo)
        this.$message.success("保存成功！")
        this.GetList()
        this.viewActive = "second"
        this.isLoad.save = false
      } catch (error) {
        console.log(error)
        this.isLoad.save = false
        // this.GetList()
      }
    },
    componentsValidate(formArray) {   // 表单校验
      return new Promise(resolve => {
        let valids = new Array
        formArray.map(ele => {
          if(this.$refs[ele]) {
            valids.push(
              new Promise((resolve, reject) => {
                this.$refs[ele].$refs["cardsform"].validate(valid => { if(valid) { resolve() } else { reject() } })
              })
            )
          }
        })
        Promise.all([...valids]).then(() => {
          resolve()
        }).catch(err => {
          this.$message.error("请将标星号的栏目填写完整后重新尝试!")
        })
      })
    },
    orderin() {   // 立刻签订
      const _refs = ['personalLoaninfo', 'otherinfo', 'houseinfo2', 'mortgageinfo2'], Contracts = this.rxdData.Contract.xdContracts || {};
      this.componentsValidate(_refs).then(async () => {
        try {
          this.isLoad.save = true
          await updateContractInfo(this.rxdData.Contract)
          await submitContract({ contid: Contracts.contId, custid: Contracts.custId, custno: Contracts.custno, assukind: Contracts.assukind })
          this.$message.success("签订成功！")
          await this.GetList()
          this.isLoad.save = false
        } catch (error) {
          this.isLoad.save = false
        }
      })
    },

    LoanCardvalidate(data) {
      console.log(date)
    },

    // 地址信息模块数据交互逻辑区块
    changeAddress(scope) {
      if(scope) {   // edit
        const { row, $index } = scope
        this.addressDialog.form = row
        this.addressDialog.formIdx = $index
        this.addressDialog.visible = true
      } else {    // add
        this.addressDialog.form = {}
        this.addressDialog.formIdx = null
        this.$nextTick(() => { this.$refs.addressForm.clearValidate() } )
        this.addressDialog.visible = true
      }
    },
    addressSave(data) {
      const addressDialog = this.addressDialog
      this.$refs.addressForm.validate(valid => {
        if (valid) {
          const AddrInfos = this.rxdData.Cust.xdCustAddrInfos;
          if(AddrInfos) {
            if(addressDialog.formIdx == null) this.rxdData.Cust.xdCustAddrInfos.push(data);
          } else {
            this.rxdData.Cust.xdCustAddrInfos = data
          }
          this.addressDialog.visible = false
        } else {
          this.$message.error("请将表单填写完整后尝试！")
        }
      })
    },
    addressOption(e) {
      const index = this.addressSelect.contract_addrtype.findIndex(item => item.dictValue === e), {dictLabel, dictValue} = this.addressSelect.contract_addrtype[index];
      this.addressDialog.form.addrtypeName = dictLabel
    },
    // 地址信息模块数据交互逻辑区块

    // 表单回调事件区块
    // LoaninfoChange(val) {
    //   const flag = val.value, config = { 
    //     label: "涉农贷款用途:",
    //     type: 'select',
    //     placeholder: "请选择涉农贷款用途",
    //     rules: [{ required: true, message: '涉农贷款用途不能为空', trigger: 'blur' }],
    //     disabled: false,
    //     inline: true,
    //     isshow: true,
    //     keys: "isagridesc",
    //     keydata: "contract_isagridesc"
    //   }
    //   if(val.keys === "isagriflag" && flag == 1) {
    //     this.formSetting.personalLoaninfo.push(config)
    //     this.$refs.personalLoaninfo._initcomponent()
    //   } else if(val.keys === "isagriflag" && flag == 0) {
    //     this.formSetting.personalLoaninfo.pop()
    //   }
    // },
    // 表单回调事件区块
    
    // 页面信息拉取区块
    GeteasyCard(idNo) {   // 获取易贷卡卡号数据
      this.isLoad.easyCard = true
      selectCardNo(idNo || null).then(res => {
        if(res.code === 200 && res.data) {
          const rxd = res.data.map(item => ({
            name: `${item.b151}（余额：${item.b152}）`,
            id: item.b151
          }))
          this.cardData = rxd
          this.isLoad.easyCard = false
        }
      })
    },
    GetcompanyInfo(creditCode) {   // 获取企业信息
      this.isLoad.DealPriv = true
      getCompanyInfo(creditCode || '').then(res => {
        // console.log(res)
        const rxd = res.data
        this.rxdData = {...this.rxdData, Cust: {...this.rxdData.Cust, xdCustDealPriv: {
          ...this.rxdData.Cust.xdCustDealPriv,
          dealproj: rxd.companyName,
          licetype: rxd.companyOrgTypeCode,
          dealname: rxd.legalPersonName,
          comedate: rxd.estiblishTime,
          liceaddr: rxd.regLocation
        }}}
        this.isLoad.DealPriv = false
      }).catch(err => {
        this.isLoad.DealPriv = false
      })
    },
    // 页面信息拉取区块
  },
  computed: {
    views() {
      return this.$route.query
    },
    changecontno() {
      if(this.rxdData.Contract && this.rxdData.Contract.xdContracts) {
        const { operid, assukind, bankid } = this.rxdData.Contract.xdContracts || {};
        return { operid, assukind, bankid }
      }
    },
    pageState() {
      if(this.rxdData.Contract && this.rxdData.Contract.xdContracts && this.rxdData.Contract.xdContracts.status) {
        return this.rxdData.Contract.xdContracts.status
      } else {
        return null
      }
    }
  },
  watch: {
    changecontno(value, old) {
      let _assuk = ''
      switch (value.assukind) {
        case "10":
          _assuk = 1
          break;
        case "20":
          _assuk = 2
          break;
        case "30":
          _assuk = 3
          break;
        default:
          _assuk = 4
          break;
      }
      this.rxdData.Contract.xdContracts.contno = `503${value.operid || ''}${_assuk}1${this.views.orderId || ''}`;
      if(this.rxdData.Contract.assucontract) this.rxdData.Contract.assucontract.assucontno = `503${value.bankid || ''}${this.views.orderId || ''}`;
      if(old && old.bankid != value.bankid) this.$refs.guaranteeinfo._initcomponent();
    },
    "rxdData.Cust.xdCustDealPriv.liceid": {
      deep: true,
      handler: function(_new, _old) {
        if(_old && _new !== _old) this.GetcompanyInfo(_new);
      }
    }
  },
  components: { cardForm }
}
</script>

<style lang="scss" scoped>
.container {
  padding: 5px 0 80px;
  background: #f0f2f5;
  .tabsHeader {
    height: 50px;
    line-height: 50px;
  }
  .newAddress {
    padding: 8px 0;
    text-align: center;
    .addaddress-buttom {
      width: 100%;
    }
  }
  /deep/ .el-tabs__header {
    background: #fff;
  }
  /deep/ .el-tabs__nav-wrap {
    padding: 0 25px;
  }
  /deep/ .el-tabs__item {
    height: auto;
  }
  /deep/ .el-card {
    border-radius: 0;
  }
}
.view-container {
  padding: 0 24px;
  box-sizing: border-box;
}
h4, h5 {
  margin: 0 0 6px 0;
}
.box-card {
  margin-top: 12px;
  /deep/ .el-tabs--border-card {
    box-shadow: none;
  }
  /deep/ .el-tabs__nav-wrap {
    padding: 0;
  }
}
.dialogFooter {
  text-align: center;
}
.margin-bottom {
  margin-bottom: 20px;
}
.fixed-buttom {
  position: fixed;
  bottom: 0;
  left: 0;
  z-index: 10;
  width: 100%;
  padding: 12px 30px;
  box-sizing: border-box;
  background: #fff;
  text-align: right;
  box-shadow: 10px 0 10px #bbb;
}
.divider-context {
  display: flex;
  align-items: center;
  min-height: 200px;
  padding: 0 50px;
}
.public-style {
  width: 160px;
}
</style>