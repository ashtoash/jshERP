<!-- from 7 5 2 7 1 8 9 2 0 -->
<template>
  <a-row :gutter="24">
    <a-col :md="24">
      <a-card :bordered="false">
        <!-- 查询区域 -->
        <div class="table-page-search-wrapper">
          <a-form layout="inline" @keyup.enter.native="searchQuery">
            <a-row :gutter="24">
              <a-col :md="6" :sm="24">
                <a-form-item label="名称" :labelCol="labelCol" :wrapperCol="wrapperCol">
                  <a-input placeholder="请输入名称" v-model="queryParam.name"></a-input>
                </a-form-item>
              </a-col>
              <a-col :md="6" :sm="24">
                <a-form-item label="编号" :labelCol="labelCol" :wrapperCol="wrapperCol">
                  <a-input placeholder="请输入编号" v-model="queryParam.serialNo"></a-input>
                </a-form-item>
              </a-col>
              <a-col :md="4" :sm="24">
                <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                  <a-button type="primary" @click="searchQuery">查询</a-button>
                  <a-button style="margin-left: 8px" v-print="'#reportPrint'" icon="printer">打印</a-button>
                  <a-button style="margin-left: 8px" @click="exportExcel" icon="download">导出</a-button>
                </span>
              </a-col>
              <a-col :md="3" :sm="24">
                <a-form-item label="本月发生总额">
                  {{allMonthAmount}}
                </a-form-item>
              </a-col>
              <a-col :md="3" :sm="24">
                <a-form-item label="当前总余额">
                  {{allCurrentAmount}}
                </a-form-item>
              </a-col>
            </a-row>
          </a-form>
        </div>
        <!-- table区域-begin -->
        <section ref="print" id="reportPrint">
          <a-table
            bordered
            ref="table"
            size="middle"
            rowKey="id"
            :columns="columns"
            :dataSource="dataSource"
            :pagination="ipagination"
            :scroll="scroll"
            :loading="loading"
            @change="handleTableChange">
              <span slot="action" slot-scope="text, record">
                <a @click="showAccountInOutList(record)">流水</a>
              </span>
          </a-table>
        </section>
        <!-- table区域-end -->
        <account-in-out-list ref="accountInOutList" @ok="modalFormOk"></account-in-out-list>
      </a-card>
    </a-col>
  </a-row>
</template>
<script>
  import AccountInOutList from './modules/AccountInOutList'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import { openDownloadDialog, sheet2blob} from "@/utils/util"
  import JEllipsis from '@/components/jeecg/JEllipsis'
  import {getAction} from '@/api/manage'
  export default {
    name: "AccountReport",
    mixins:[JeecgListMixin],
    components: {
      AccountInOutList,
      JEllipsis
    },
    data () {
      return {
        labelCol: {
          span: 5
        },
        wrapperCol: {
          span: 18,
          offset: 1
        },
        // 查询条件
        queryParam: {
          name:'',
          serialNo:''
        },
        ipagination:{
          pageSizeOptions: ['10', '20', '30', '100', '200']
        },
        allMonthAmount: '',
        allCurrentAmount: '',
        tabKey: "1",
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          { title: '名称', dataIndex: 'name', width: 100},
          { title: '编号', dataIndex: 'serialNo', width: 150, align: "center"},
          { title: '期初金额', dataIndex: 'initialAmount', width: 100, align: "center"},
          { title: '本月发生额', dataIndex: 'thisMonthAmount', width: 100, align: "center"},
          { title: '当前余额', dataIndex: 'currentAmount', width: 100, align: "center"},
          { title: '账户流水', dataIndex: 'action', align:"center", width: 200,
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/account/list",
          getStatistics: "/account/getStatistics"
        }
      }
    },
    created () {
      this.getAccountStatistics()
    },
    methods: {
      getAccountStatistics() {
        getAction(this.url.getStatistics, this.queryParam).then((res)=>{
          if(res && res.code === 200) {
            if(res.data){
              this.allMonthAmount = res.data.allMonthAmount
              this.allCurrentAmount = res.data.allCurrentAmount
            }
          }
        })
      },
      searchQuery() {
        this.loadData(1);
        this.getAccountStatistics();
      },
      showAccountInOutList(record) {
        this.$refs.accountInOutList.show(record);
        this.$refs.accountInOutList.title = "查看账户流水";
        this.$refs.accountInOutList.disableSubmit = false;
      },
      exportExcel() {
        let aoa = [['名称', '编号', '期初金额', '本月发生额', '账户流水']]
        for (let i = 0; i < this.dataSource.length; i++) {
          let ds = this.dataSource[i]
          let item = [ds.name, ds.serialNo, ds.initialAmount, ds.thisMonthAmount, ds.currentAmount]
          aoa.push(item)
        }
        openDownloadDialog(sheet2blob(aoa), '账户统计')
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>