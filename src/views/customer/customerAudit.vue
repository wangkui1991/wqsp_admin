<template>
  <div class="app-container calendar-list-container">
    <div class="filter-container">
      <el-input @keyup.enter.native="handleFilter" style="width: 150px;" class="filter-item" :placeholder="$t('customer.filter.shopName')" v-model="listQuery.title">
      </el-input>
      <el-select clearable class="filter-item" style="width: 130px" v-model="listQuery.type" :placeholder="$t('customer.filter.channelType')">
        <el-option v-for="item in  calendarTypeOptions" :key="item.key" :label="item.display_name+'('+item.key+')'" :value="item.key">
        </el-option>
      </el-select>
      <el-select clearable class="filter-item" style="width: 130px" v-model="listQuery.type" :placeholder="$t('customer.filter.auditStatus')">
        <el-option v-for="item in  calendarTypeOptions" :key="item.key" :label="item.display_name+'('+item.key+')'" :value="item.key">
        </el-option>
      </el-select>
      <el-input @keyup.enter.native="handleFilter" style="width: 150px;" class="filter-item" :placeholder="$t('customer.filter.belongArea')" v-model="listQuery.title">
      </el-input>
      <el-input @keyup.enter.native="handleFilter" style="width: 150px;" class="filter-item" :placeholder="$t('customer.filter.shopPhone')" v-model="listQuery.title">
      </el-input>
      <el-input @keyup.enter.native="handleFilter" style="width: 150px;" class="filter-item" :placeholder="$t('customer.filter.contactNum')" v-model="listQuery.title">
      </el-input>
      <el-button class="filter-item" type="primary" v-waves icon="el-icon-search" @click="handleFilter">{{$t('customer.filter.search')}}</el-button>
      <el-button class="filter-item" type="primary" v-waves icon="el-icon-search" @click="add">{{$t('premiss.filter.add')}}</el-button>
    </div>

    <el-table  :key='tableKey' :data="list" v-loading="listLoading" element-loading-text="给我一点时间" border stripe fit highlight-current-row
               style="width: 100%">
      <el-table-column type="selection" min-width="45" align="center">
      </el-table-column>
      <el-table-column prop="shopName" align="center" :label="$t('customer.table.shopName')" min-width="150px">
      </el-table-column>
      <el-table-column prop="channelType" :label="$t('customer.table.channelType')" min-width="110"  align="center" >
      </el-table-column>
      <el-table-column prop="shopMan" :label="$t('customer.table.shopMan')" min-width="130"  align="center">
      </el-table-column>
      <el-table-column prop="shopPhone" :label="$t('customer.table.shopPhone')" min-width="80"  align="center">
      </el-table-column>
      <el-table-column prop="linkMan" :label="$t('customer.table.linkMan')"  min-width="80"align="center" >
      </el-table-column>
      <el-table-column prop="contactNum" :label="$t('customer.table.contactNum')" min-width="150"  align="center">
      </el-table-column>
      <el-table-column prop="shopAddress" :label="$t('customer.table.shopAddress')" min-width="90"  align="center">
      </el-table-column>
      <el-table-column :label="$t('customer.table.auditStatus')" min-width="100"  align="center">
        <template slot-scope="scope">
          <span v-if="scope.row.auditStatus===3" style="color:green">审核通过</span>
          <span v-else-if="scope.row.auditStatus===2" style="color:blue">已提交</span>
          <span v-else   style="color:red">未知</span>
        </template>
      </el-table-column>
      <el-table-column prop="opType" label="操作类型" :formatter="opType"  min-width="80"  align="center">
      </el-table-column>
      <el-table-column align="center" :label="$t('customer.table.actions')" fixed="right" min-width="180" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button type="primary" size="mini" icon="el-icon-edit" @click="handleUpdate(scope.row)">{{$t('customer.table.edit')}}</el-button>
          <el-button  size="mini" type="danger" icon="el-icon-delete" @click="handleModifyStatus(scope.row,'deleted')">{{$t('customer.table.delete')}}</el-button>
        </template>
      </el-table-column>
    </el-table>

    <div class="pagination-container">
      <el-pagination background @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page.sync="listQuery.page"
                     :page-sizes="[10,20,30, 50]" :page-size="listQuery.limit" layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </div>

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form :rules="rules" ref="dataForm" :model="temp" label-position="left" label-width="70px" style='width: 400px; margin-left:50px;'>
        <el-form-item :label="$t('table.type')" prop="type">
          <el-select class="filter-item" v-model="temp.type" placeholder="Please select">
            <el-option v-for="item in  calendarTypeOptions" :key="item.key" :label="item.display_name" :value="item.key">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item :label="$t('table.date')" prop="timestamp">
          <el-date-picker v-model="temp.timestamp" type="datetime" placeholder="Please pick a date">
          </el-date-picker>
        </el-form-item>
        <el-form-item :label="$t('table.title')" prop="title">
          <el-input v-model="temp.title"></el-input>
        </el-form-item>
        <el-form-item :label="$t('table.status')">
          <el-select class="filter-item" v-model="temp.status" placeholder="Please select">
            <el-option v-for="item in  statusOptions" :key="item" :label="item" :value="item">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item :label="$t('table.importance')">
          <el-rate style="margin-top:8px;" v-model="temp.importance" :colors="['#99A9BF', '#F7BA2A', '#FF9900']" :max='3'></el-rate>
        </el-form-item>
        <el-form-item :label="$t('table.remark')">
          <el-input type="textarea" :autosize="{ minRows: 2, maxRows: 4}" placeholder="Please input" v-model="temp.remark">
          </el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">{{$t('table.cancel')}}</el-button>
        <el-button v-if="dialogStatus=='create'" type="primary" @click="createData">{{$t('table.confirm')}}</el-button>
        <el-button v-else type="primary" @click="updateData">{{$t('table.confirm')}}</el-button>
      </div>
    </el-dialog>

    <el-dialog title="Reading statistics" :visible.sync="dialogPvVisible">
      <el-table :data="pvData" border fit highlight-current-row style="width: 100%">
        <el-table-column prop="key" label="Channel"> </el-table-column>
        <el-table-column prop="pv" label="Pv"> </el-table-column>
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false">{{$t('table.confirm')}}</el-button>
      </span>
    </el-dialog>
    <add-dialog :addContent="addContent" ref="addDialog"></add-dialog>

  </div>
</template>

<script>
  import { fetchList, fetchPv, createArticle, updateArticle } from '@/api/article'
  import waves from '@/directive/waves' // 水波纹指令
  import { parseTime } from '@/utils'
  import addDialog from '@/components/Dialog/addDialog'

  const calendarTypeOptions = [
    { key: '2', display_name: '审核通过' },
    { key: '1', display_name: '待审核' },
    { key: '0', display_name: '退回' },

  ]

  // arr to obj ,such as { CN : "China", US : "USA" }
  const calendarTypeKeyValue = calendarTypeOptions.reduce((acc, cur) => {
    acc[cur.key] = cur.display_name
    return acc
  }, {})

  var datas=require("../../mock/falseData/5_customer/1_customerAudit")
  var address=require("../../mock/address")

  export default {
    name: 'complexTable',
    directives: {
      waves
    },
    data() {
      return {
        tableKey: 0,
        list: datas.data,
        total: 20,
        listLoading: false,
        listQuery: {
          page: 1,
          limit: 10,
          importance: undefined,
          title: undefined,
          type: undefined,
          sort: '+id'
        },
        importanceOptions: [1, 2, 3],
        calendarTypeOptions,
        sortOptions: [{ label: 'ID Ascending', key: '+id' }, { label: 'ID Descending', key: '-id' }],
        statusOptions: ['published', 'draft', 'deleted'],
        showReviewer: false,
        temp: {
          id: undefined,
          importance: 1,
          remark: '',
          timestamp: new Date(),
          title: '',
          type: '',
          status: 'published'
        },
        dialogFormVisible: false,
        dialogStatus: '',
        textMap: {
          update: 'Edit',
          create: 'Create'
        },
        dialogPvVisible: false,
        pvData: [],
        rules: {
          type: [{ required: true, message: 'type is required', trigger: 'change' }],
          timestamp: [{ type: 'date', required: true, message: 'timestamp is required', trigger: 'change' }],
          title: [{ required: true, message: 'title is required', trigger: 'blur' }]
        },
        downloadLoading: false,
        addContent:{
          title:"新增客户信息",
          width:'80%',
          type:0,
          content:[
            {type:3,label:"所属区域",select:address,prov:'',city:'',area:'',width:24},
            {type:0,label:"店铺名称",placehold:'店铺名称',select:'',width:12},
            {type:1,label:"渠道类型",placehold:'选择渠道类型',width:12,select:'流通',options:['流通','特通']},
            {type:0,label:"店铺地址",placehold:'店铺地址',width:24,select:''},
            {type:1,label:"证件类型",placehold:'请选择',width:12,select:'',options:['烟草许可证','营业执照号']},
            {type:0,label:"证件号",placehold:'证件号',width:12,select:''},
            {type:0,label:"店主姓名",placehold:'店主姓名',width:12,select:''},
            {type:0,label:"店主手机",placehold:'店主手机',width:12,select:''},
            {type:0,label:"积分",placehold:'0',width:12,select:'',disabled:true},
            {type:0,label:"固定电话",placehold:'固定电话',width:12,select:''},
            {type:0,label:"联系人",placehold:'联系人',width:12,select:''},
            {type:0,label:"联系电话",placehold:'联系电话',width:12,select:''},
            {type:1,label:"地推人员",placehold:'请选择',width:24,select:'',options:['普通用户','商户用户']},
          ]
        }
      }
    },
    filters: {
      statusFilter(status) {
        const statusMap = {
          published: 'success',
          draft: 'info',
          deleted: 'danger'
        }
        return statusMap[status]
      },
      typeFilter(type) {
        return calendarTypeKeyValue[type]
      }
    },
    // created() {
    //   this.getList()
    // },
    methods: {
      add: function () {
        this.$refs.addDialog.add()
      },
      opType: function (row) {
        return row.opType == 2 ? '修改' : row.opType == 1 ? '增加' : '未知';
      },
      getList() {
        this.listLoading = true
        fetchList(this.listQuery).then(response => {
          this.list = response.data.items
          this.total = response.data.total
          this.listLoading = false
        })
      },
      handleFilter() {
        this.listQuery.page = 1
        this.getList()
      },
      handleSizeChange(val) {
        this.listQuery.limit = val
        this.getList()
      },
      handleCurrentChange(val) {
        this.listQuery.page = val
        this.getList()
      },
      handleModifyStatus(row, status) {
        this.$message({
          message: '操作成功',
          type: 'success'
        })
        row.status = status
      },
      resetTemp() {
        this.temp = {
          id: undefined,
          importance: 1,
          remark: '',
          timestamp: new Date(),
          title: '',
          status: 'published',
          type: ''
        }
      },
      handleCreate() {
        this.resetTemp()
        this.dialogStatus = 'create'
        this.dialogFormVisible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].clearValidate()
        })
      },
      createData() {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.temp.id = parseInt(Math.random() * 100) + 1024 // mock a id
            this.temp.author = 'vue-element-admin'
            createArticle(this.temp).then(() => {
              this.list.unshift(this.temp)
              this.dialogFormVisible = false
              this.$notify({
                title: '成功',
                message: '创建成功',
                type: 'success',
                duration: 2000
              })
            })
          }
        })
      },
      handleUpdate(row) {
        this.temp = Object.assign({}, row) // copy obj
        this.temp.timestamp = new Date(this.temp.timestamp)
        this.dialogStatus = 'update'
        this.dialogFormVisible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].clearValidate()
        })
      },
      updateData() {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            const tempData = Object.assign({}, this.temp)
            tempData.timestamp = +new Date(tempData.timestamp) // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
            updateArticle(tempData).then(() => {
              for (const v of this.list) {
                if (v.id === this.temp.id) {
                  const index = this.list.indexOf(v)
                  this.list.splice(index, 1, this.temp)
                  break
                }
              }
              this.dialogFormVisible = false
              this.$notify({
                title: '成功',
                message: '更新成功',
                type: 'success',
                duration: 2000
              })
            })
          }
        })
      },
      handleDelete(row) {
        this.$notify({
          title: '成功',
          message: '删除成功',
          type: 'success',
          duration: 2000
        })
        const index = this.list.indexOf(row)
        this.list.splice(index, 1)
      },
      handleFetchPv(pv) {
        fetchPv(pv).then(response => {
          this.pvData = response.data.pvData
          this.dialogPvVisible = true
        })
      },
      handleDownload() {
        this.downloadLoading = true
        import('@/vendor/Export2Excel').then(excel => {
          const tHeader = ['timestamp', 'title', 'type', 'importance', 'status']
          const filterVal = ['timestamp', 'title', 'type', 'importance', 'status']
          const data = this.formatJson(filterVal, this.list)
          excel.export_json_to_excel(tHeader, data, 'table-list')
          this.downloadLoading = false
        })
      },
      formatJson(filterVal, jsonData) {
        return jsonData.map(v => filterVal.map(j => {
          if (j === 'timestamp') {
            return parseTime(v[j])
          } else {
            return v[j]
          }
        }))
      }
    },
    components: {
      addDialog
    }
  }
</script>
