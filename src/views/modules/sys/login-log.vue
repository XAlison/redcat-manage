<template>
  <div class="mod-log">
    <el-form :inline="true" :model="loginLogForm" ref="loginLogForm" @keyup.enter.native="search">
      <el-form-item  label="登录名" prop="userId">
        <el-input v-model="loginLogForm.userId" placeholder="请输入登录名" clearable/>
      </el-form-item>
      <el-form-item  label="城市名称" prop="cityName">
        <el-input v-model="loginLogForm.cityName" placeholder="请输入城市名称" clearable/>
      </el-form-item>
      <!--操作按钮-->
      <el-button-group>
        <el-button @click="search">查询</el-button>
        <el-button @click="reset('loginLogForm')">重置</el-button>
      </el-button-group>
    </el-form>
    <el-table :data="items" v-loading="loading">
      <el-table-column prop="id" header-align="center" align="center" width="50" label="ID"></el-table-column>
      <el-table-column prop="userId" header-align="center" align="center" width="120" label="用户名"></el-table-column>
      <el-table-column prop="cityId" header-align="center" align="center" label="城市ID"></el-table-column>
      <el-table-column prop="cityName" header-align="center" align="center" label="城市名称"></el-table-column>
      <el-table-column
        prop="browserName"
        label="浏览器名称"
        header-align="center"
        align="center"
        width="150"
        :show-overflow-tooltip="true"
      ></el-table-column>
      <el-table-column
        prop="browserType"
        label="浏览器类型"
        header-align="center"
        align="center"
        width="150"
        :show-overflow-tooltip="true"
      ></el-table-column>
      <el-table-column
        prop="browserVersion"
        label="浏览器版本"
        header-align="center"
        align="center"
        width="150"
        :show-overflow-tooltip="true"
      ></el-table-column>
      <el-table-column
        prop="deviceType"
        label="设备"
        header-align="center"
        align="center"
        width="150"
        :show-overflow-tooltip="true"
      ></el-table-column>
      <el-table-column
        prop="loginIp"
        label="登录IP"
        header-align="center"
        align="center"
        width="150"
        :show-overflow-tooltip="true"
      ></el-table-column>
      <el-table-column
        prop="loginTime"
        label="登录时间"
        header-align="center"
        align="center"
        width="150"
        :show-overflow-tooltip="true"
      ></el-table-column>
      <el-table-column
        prop="systemGroup"
        label="系统家族"
        header-align="center"
        align="center"
        width="150"
        :show-overflow-tooltip="true"
      ></el-table-column>
      <el-table-column
        prop="systemName"
        label="系统名称"
        header-align="center"
        align="center"
        width="150"
        :show-overflow-tooltip="true"
      ></el-table-column>
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
  name: 'login-log',
  data () {
    return {
      loginLogForm: {userId: '', cityName: ''},
      pagination: {currentPage: 1, pageSize: 10, pageSizes: [10, 20, 30, 50, 100], total: 0},
      loading: false,
      items: [],
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
          userId: this.loginLogForm.userId,
          cityName: this.loginLogForm.cityName
          }
        this.$http({
          url: this.$http.adornUrl('/sys/log/loginLogs'),
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

