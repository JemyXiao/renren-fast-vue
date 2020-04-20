<template>
  <div class="mod-schedule">
    <el-form :inline="true" :model="dataForm">
      <el-form-item>
        <el-input v-model="dataForm.beanName" placeholder="注册中心名称" clearable></el-input>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList.slice((pageIndex-1)*pageSize,pageIndex*pageSize).filter(data => !dataForm.beanName || data.name.toLowerCase().includes(dataForm.beanName.toLowerCase()))"
      border
      v-loading="dataListLoading"
      style="width: 100%;">
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="注册中心名称">
      </el-table-column>
      <el-table-column
        prop="zkAddressList"
        header-align="center"
        align="center"
        label="注册中心地址">
      </el-table-column>
      <el-table-column
        prop="namespace"
        header-align="center"
        align="center"
        label="命名空间">
      </el-table-column>
<!--      <el-table-column-->
<!--        prop="activated"-->
<!--        header-align="center"-->
<!--        align="center"-->
<!--        label="是否连接">-->
<!--      </el-table-column>-->
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button v-show = "!scope.row.activated" type="primary" size="small" @click="connectHandle(scope.row)">连接</el-button>
          <el-button v-show = "scope.row.activated" type="success" size="small" >已连接</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        dataForm: {
          beanName: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 1,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        logVisible: false
      }
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/registry-center/listCenter'),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.centerList
            this.totalPage = data.centerList.length
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
      },
      connectHandle (val) {
        var data = {
          'name': val.name,
          'namespace': val.namespace,
          'zkAddressList': val.zkAddressList
        }
        this.$confirm(`确定对连接操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/registry-center/connect'),
            method: 'post',
            data: this.$http.adornData(data, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }).catch(() => {
        })
      }
    }
  }
</script>
