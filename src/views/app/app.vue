<template>
  <div class="mixin-components-container">
    <el-row>
      <el-card class="box-card">
        <div slot="header" class="clearfix">
          <span v-if="isEdit===false">添加租户</span>
          <span v-if="isEdit===true">修改租户信息</span>
        </div>
        <div style="margin-bottom:50px;">
          <el-form ref="form" :model="form" label-width="140px">
            <el-form-item label="app_id" class="is-required">
              <el-input v-model="form.app_id" placeholder="6-128位数字字母下划线" :disabled="isEdit===true" />
            </el-form-item>
            <el-form-item label="租户名称" class="is-required">
              <el-input v-model="form.name" placeholder="最多255字符，必填" />
            </el-form-item>
            <el-form-item label="32位秘钥">
              <el-input v-model="form.secret" placeholder="32位字符串，非必填，为空时自动生成" />
            </el-form-item>
            <el-form-item label="QPS限流">
              <el-input v-model="form.qps" placeholder="0表示无限制" />
            </el-form-item>
            <el-form-item label="日请求量限流">
              <el-input v-model="form.qpd" placeholder="0表示无限制" />
            </el-form-item>
            <el-form-item>
              <el-button type="primary" :disabled="submitButDisabled" @click="handleSubmit">立即提交</el-button>
            </el-form-item>
          </el-form>
        </div>
      </el-card>
    </el-row>

  </div>
</template>

<script>
import { appAdd, appDetail, appUpdate } from '@/api/app'
export default {
  name: 'AppCreate',
  data() {
    return {
      submitButDisabled: false,
      isEdit: false,
      form: {
        app_id: '',
        name: '',
        secret: '',
        qps: '',
        qpd: ''
        // white_list: '' // 暂未使用
      }
    }
  },
  created() {
    // 获取 id ，如果能拿到就是修改操作
    const id = this.$route.params && this.$route.params.id
    if (id > 0) {
      this.isEdit = true
      this.fetchData(id)
    }
    // Why need to make a copy of this.$route here?
    // Because if you enter this page and quickly switch tag, may be in the execution of the setTagsViewTitle function, this.$route is no longer pointing to the current page
    // https://github.com/PanJiaChen/vue-element-admin/issues/1221
    this.tempRoute = Object.assign({}, this.$route)
  },
  methods: {
    fetchData(id) {
      const query = { id: id }
      // 根据服务ID获取详细信息
      // response获取的数据绑定到表单
      appDetail(query).then(response => {
        this.form.id = response.data.id
        this.form.app_id = response.data.app_id
        this.form.name = response.data.name
        this.form.qps = response.data.qps
        this.form.qpd = response.data.qpd
        this.form.secret = response.data.secret
        // this.form.white_ips = response.data.white_ips.replace(/,/g, '\n') // 暂不使用
      }).catch(() => {

      })
    },
    handleSubmit() {
      this.submitButDisabled = true
      const query = Object.assign({}, this.form)
      console.log(query)
      // 换行转 ',' 传递到后端
      //   query.white_ips = query.white_ips.replace(/\n/g, ',') // 暂不使用
      // 字段类型转换 true false -> 1 0
      // 数字类型要从字符串转换成数字
      query.qps = Number(query.qps)
      query.qpd = Number(query.qpd)

      if (this.isEdit) {
        appUpdate(query).then(response => {
          this.submitButDisabled = false
          this.$notify({
            title: 'Success',
            message: '修改成功',
            type: 'success',
            duration: 2000
          })
        }).catch(() => {
          this.submitButDisabled = false
        })
      } else {
        appAdd(query).then(response => {
          this.submitButDisabled = false
          this.$notify({
            title: 'Success',
            message: '添加成功',
            type: 'success',
            duration: 2000
          })
        }).catch(() => {
          this.submitButDisabled = false
        })
      }
    }
  }
}
</script>

<style scoped>
.mixin-components-container {
  background-color: #f0f2f5;
  padding: 30px;
  min-height: calc(100vh - 84px);
}
.component-item{
  min-height: 100px;
}
</style>
