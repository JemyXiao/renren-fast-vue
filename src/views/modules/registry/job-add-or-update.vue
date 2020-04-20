<template>
  <el-dialog
    :title="'修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="作业名称" prop="jobName">
        <el-input v-model="dataForm.jobName" disabled></el-input>
      </el-form-item>
      <el-form-item label="作业类型" prop="jobType" >
        <el-input v-model="dataForm.jobType" disabled></el-input>
      </el-form-item>
      <el-form-item label="作业实现类" prop="jobClass" >
        <el-input v-model="dataForm.jobClass" disabled></el-input>
      </el-form-item>
      <el-form-item label="Cron表达式" prop="cron">
        <el-input v-model="dataForm.cron"></el-input>
      </el-form-item>
      <el-form-item label="作业描述信息" prop="description">
        <el-input v-model="dataForm.description"></el-input>
      </el-form-item>
      <el-form-item label="作业分片总数" prop="shardingTotalCount">
        <el-input v-model="dataForm.shardingTotalCount"></el-input>
      </el-form-item>
      <el-form-item label="自定义参数" prop="jobParameter">
        <el-input v-model="dataForm.jobParameter"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          jobName: '',
          jobType: '',
          jobClass: '',
          cron: '',
          description: '',
          shardingTotalCount: 1,
          jobParameter: '',
          maxTimeDiffSeconds: '',
          monitorPort: '',
          monitorExecution: true,
          failover: true,
          misfire: true,
          shardingItemParameters: '',
          jobProperties: {
            executor_service_handler: '',
            job_exception_handler: ''
          },
          scriptCommandLine: '',
          reconcileIntervalMinutes: ''
        }
      }
    },
    methods: {
      init (name) {
        this.$http({
          url: this.$http.adornUrl('/jobs/config/getJobSettings'),
          method: 'get',
          params: this.$http.adornParams({
            'jobName': name
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            console.log(data.jobSettings)
            this.dataForm.jobName = data.jobSettings.jobName
            this.dataForm.jobType = data.jobSettings.jobType
            this.dataForm.jobClass = data.jobSettings.jobClass
            this.dataForm.cron = data.jobSettings.cron
            this.dataForm.description = data.jobSettings.description
            this.dataForm.shardingTotalCount = data.jobSettings.shardingTotalCount
            this.dataForm.jobParameter = data.jobSettings.jobParameter
            this.dataForm.maxTimeDiffSeconds = data.jobSettings.maxTimeDiffSeconds
            this.dataForm.monitorPort = data.jobSettings.monitorPort
            this.dataForm.monitorExecution = data.jobSettings.monitorExecution
            this.dataForm.failover = data.jobSettings.failover
            this.dataForm.misfire = data.jobSettings.misfire
            this.dataForm.shardingItemParameters = data.jobSettings.shardingItemParameters
            this.dataForm.jobProperties = data.jobSettings.jobProperties
            this.dataForm.scriptCommandLine = data.jobSettings.scriptCommandLine
            this.dataForm.reconcileIntervalMinutes = data.jobSettings.reconcileIntervalMinutes
          }
        })
        this.visible = true
      },
      // 表单提交
      dataFormSubmit () {
        console.log(this.dataForm)
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/jobs/config/updateJobSettings`),
              method: 'post',
              data: this.$http.adornData({
                'jobName': this.dataForm.jobName,
                'jobType': this.dataForm.jobType,
                'jobClass': this.dataForm.jobClass,
                'cron': this.dataForm.cron,
                'description': this.dataForm.description,
                'shardingTotalCount': this.dataForm.shardingTotalCount,
                'jobParameter': this.dataForm.jobParameter,
                'maxTimeDiffSeconds': this.dataForm.maxTimeDiffSeconds,
                'monitorPort': this.dataForm.monitorPort,
                'monitorExecution': this.dataForm.monitorExecution,
                'failover': this.dataForm.failover,
                'misfire': this.dataForm.misfire,
                'shardingItemParameters': this.dataForm.shardingItemParameters,
                'jobProperties': this.dataForm.jobProperties,
                'scriptCommandLine': this.dataForm.scriptCommandLine,
                'reconcileIntervalMinutes': this.dataForm.reconcileIntervalMinutes
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
