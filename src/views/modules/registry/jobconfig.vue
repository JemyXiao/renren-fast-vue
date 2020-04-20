<template>
  <div class="mod-schedule">
    <el-form :inline="true" :model="dataForm">
      <el-form-item>
        <el-input v-model="dataForm.beanName" placeholder="作业名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()" type="primary">查询</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList.slice((pageIndex-1)*pageSize,pageIndex*pageSize).filter(data => !dataForm.beanName || data.jobName.toLowerCase().includes(dataForm.beanName.toLowerCase()))"
      border
      v-loading="dataListLoading"
      style="width: 100%;">
      <el-table-column
        prop="jobName"
        header-align="center"
        align="center"
        label="作业名称">
      </el-table-column>
      <el-table-column
        prop="shardingTotalCount"
        header-align="center"
        align="center"
        label="作业分片总数">
      </el-table-column>
      <el-table-column
        prop="cron"
        header-align="center"
        align="center"
        label="Corn表达式">
      </el-table-column>
      <el-table-column
        prop="description"
        header-align="center"
        align="center"
        label="作业描述信息">
      </el-table-column>
      <el-table-column
        prop="status"
        header-align="center"
        align="center"
        label="状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status === 'OK'" size="small" type="success">正常</el-tag>
          <el-tag v-if="scope.row.status === 'DISABLED'" size="small" type="danger">已失效</el-tag>
          <el-tag v-if="scope.row.status === 'SHARDING_FLAG'" size="small" type="info">待分片</el-tag>
          <el-tag v-if="scope.row.status === 'CRASHED'" size="small" type="danger">已下线</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="300"
        label="操作">
        <template slot-scope="scope">
          <el-tooltip class="item" effect="dark"  content="修改作业" placement="top-start">
            <el-button v-if="isAuth('job:update')" type="primary" icon="el-icon-edit"  circle @click="updateHandle(scope.row.jobName)"></el-button>
          </el-tooltip>
          <el-tooltip class="item" effect="dark" content="触发作业" placement="top-start">
            <el-button v-show = "isAuth('job:trigger') && scope.row.status === 'OK'" type="warning" icon="el-icon-bell" circle @click="triggerHandle(scope.row.jobName)"></el-button>
          </el-tooltip>
          <el-tooltip class="item" effect="dark" content="失效作业" placement="top-start">
            <el-button v-show = "isAuth('job:disabled') && scope.row.status === 'OK'" type="danger" icon="el-icon-error" circle @click="disabledHandle(scope.row.jobName)"></el-button>
          </el-tooltip>
          <el-tooltip class="item" effect="dark" content="生效作业" placement="top-start">
            <el-button v-show = "isAuth('job:enabled') && scope.row.status === 'DISABLED'" type="danger" icon="el-icon-success" circle @click="enableHandle(scope.row.jobName)"></el-button>
          </el-tooltip>
          <el-tooltip class="item" effect="dark" content="终止作业" placement="top-start">
            <el-button v-show = "isAuth('job:shutdown') && scope.row.status !== 'CRASHED'" type="danger" icon="el-icon-remove" circle @click="shutDownHandle(scope.row.jobName)"></el-button>
          </el-tooltip>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[5, 10, 20, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>

  </div>

</template>

<script>
  import AddOrUpdate from './job-add-or-update'
  export default {
    data () {
      return {
        dataForm: {
          beanName: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 5,
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
    components: {
      AddOrUpdate
    },
    methods: {
      buttonShowHandle (val) {
        if (val.status === 'OK') {
          return true
        }
        return false
      },
      updateHandle (jobName) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(jobName)
        })
      },
      triggerHandle (jobName) {
        this.$confirm(`确定对[${jobName}]进行触发操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/jobs/trigger'),
            method: 'get',
            params: this.$http.adornParams({
              'jobName': jobName
            })
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
        }).catch(() => {})
      },
      disabledHandle (jobName) {
        this.$confirm(`确定对[${jobName}]进行失效操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/jobs/disable'),
            method: 'get',
            params: this.$http.adornParams({
              'jobName': jobName
            })
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
        }).catch(() => {})
      },
      enableHandle (jobName) {
        this.$confirm(`确定对[${jobName}]进行有效操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/jobs/enable'),
            method: 'get',
            params: this.$http.adornParams({
              'jobName': jobName
            })
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
        }).catch(() => {})
      },
      shutDownHandle (jobName) {
        this.$confirm(`确定对[${jobName}]进行终止操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/jobs/shutdown'),
            method: 'get',
            params: this.$http.adornParams({
              'jobName': jobName
            })
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
        }).catch(() => {})
      },
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/jobs/list'),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.jobList
            this.totalPage = data.jobList.length
          } else {
            this.dataList = []
            this.totalPage = 0
            this.$message.error(data.msg)
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
        this.$confirm(`确定连接操作?`, '提示', {
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
