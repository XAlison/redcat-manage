<template>
  <div class="mod-log">
    <el-form :inline="true" :model="accountForm" ref="accountForm" @keyup.enter.native="search">
      <el-form-item label="手机号" prop="phoneNumber">
        <el-input v-model="accountForm.phoneNumber" placeholder="请输入手机号"></el-input>
      </el-form-item>
      <el-form-item label="姓名" prop="name">
        <el-input v-model="accountForm.name" placeholder="请输入姓名" clearable/>
      </el-form-item>
      <el-form-item label="用户名" prop="userName">
        <el-input v-model="accountForm.userName" placeholder="请输入用户名" clearable/>
      </el-form-item>
      <!--操作按钮-->
      <el-button-group>
        <el-button @click="search">查询</el-button>
        <el-button @click="reset('accountForm')">重置</el-button>
      </el-button-group>
    </el-form>
    <el-table :data="items" v-loading="loading" :border="true">
      <el-table-column prop="id" header-align="center" align="center" label="ID"></el-table-column>
      <el-table-column
        prop="phoneNumber"
        header-align="center"
        align="center"
        width="120"
        label="手机号"
      ></el-table-column>
      <el-table-column prop="name" header-align="center" align="center" label="姓名"></el-table-column>
      <el-table-column prop="userName" header-align="center" align="center" label="用户名"></el-table-column>
      <el-table-column prop="mailbox" header-align="center" align="center" label="邮箱"></el-table-column>
    </el-table>
    <el-footer class="pagination-footer">
      <el-pagination
        background=""
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="pagination.currentPage"
        :page-sizes="pagination.pageSizes"
        :page-size="pagination.pageSize"
        layout="total, prev, pager, next, sizes"
        :total="pagination.total"
      ></el-pagination>
    </el-footer>
  </div>
</template>

<script>

export default {
  name: 'user-list',
  data () {
    return {
      accountForm: {phoneNumber: '', name: '', userName: ''},
      pagination: {currentPage: 1, pageSize: 10, pageSizes: [10, 20, 30, 50, 100], total: 0},
      loading: false,
      items: [],
      addOrUpdateVisible: false
      
    }
  },
  watch: {
    'pagination.currentPage' (val) {
      this.getTableData()
    },
    'pagination.pageSize' (val) {
      this.getTableData()
    }
  },
  created () {
    this.getTableData()
  },
  methods: {
      // 新增 / 修改
      addOrUpdateHandle (row) {       
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(row)
        })
      },
    search () {
      this.pagination.currentPage = 1
      this.getTableData()
    },
    reset (formName) {
      this.$refs[formName].resetFields()
      this.getTableData()
    },
    getTableData () {
      this.loading = true
      let page = this.pagination
        let filters = {
          index: page.currentPage,
          size: page.pageSize,
          phoneNumber: this.accountForm.phoneNumber,
          name: this.accountForm.name,
          userName:this.accountForm.userName,
          }
        this.$http({
          url: this.$http.adornUrl('/api/accounts/manageAccounts'),
          method: 'get',
          params: filters
        }).then(({ data }) => {          
          this.pagination.total =data.total
          if ((this.pagination.currentPage - 1) * this.pagination.pageSize >= this.pagination.total) {
            this.pagination.currentPage--
            return
          }
          this.items = data.items
          this.loading = false
        }).catch(() => {
          this.loading = false
        })
    },
    handleSizeChange (val) {
      this.pagination.pageSize = val
    },
    handleCurrentChange (val) {
      this.pagination.currentPage = val
    }
  }
}
</script>

