<template>
  <div class="mixin-components-container">
    <el-row>
      <el-card class="box-card">
        <div slot="header" class="clearfix">
          <span v-if="isEdit===false">创建TCP服务</span>
          <span v-if="isEdit===true">修改TCP服务</span>
        </div>
        <div style="margin-bottom:50px;">
          <el-form ref="form" :model="form" label-width="140px">
            <el-form-item label="服务名称" class="is-required">
              <el-input v-model="form.service_name" placeholder="6-128位数字字母下划线" :disabled="isEdit===true" />
            </el-form-item>
            <el-form-item label="服务描述" class="is-required">
              <el-input v-model="form.service_desc" placeholder="最多255字符，必填" />
            </el-form-item>

            <el-form-item label="端口" class="is-required">
              <el-input v-model="form.port" placeholder="需要设置8001-8999范围内的数字，必填" :disabled="isEdit===true" />
            </el-form-item>

            <el-form-item label="开启验证">
              <el-switch
                v-model="form.open_auth"
                :active-value="1"
                :inactive-value="0"
              />
            </el-form-item>
            <el-form-item label="IP白名单">
              <el-input
                v-model="form.white_list"
                type="textarea"
                :autosize="{ minRows: 2 }"
                placeholder="格式：127.0.0.1 多条换行（注意：白名单优先级高于黑名单！)"
              />
            </el-form-item>
            <el-form-item label="IP黑名单">
              <el-input
                v-model="form.black_list"
                type="textarea"
                :autosize="{ minRows: 2 }"
                placeholder="格式：127.0.0.1 多条换行"
              />
            </el-form-item>
            <el-form-item label="客户端限流">
              <el-input v-model="form.clientip_flow_limit" placeholder="0表示无限制" />
            </el-form-item>
            <el-form-item label="服务端限流">
              <el-input v-model="form.service_flow_limit" placeholder="0表示无限制" />
            </el-form-item>

            <el-form-item label="轮询方式">
              <el-radio-group v-model="form.round_type">
                <el-radio :label="0">random</el-radio>
                <el-radio :label="1">round-robin</el-radio>
                <el-radio :label="2">weight_round-robin</el-radio>
                <el-radio :label="3">ip_hash</el-radio>
              </el-radio-group>
            </el-form-item>
            <el-form-item label="IP列表" class="is-required">
              <el-input
                v-model="form.ip_list"
                type="textarea"
                :autosize="{ minRows: 2 }"
                placeholder="格式：127.0.0.1:80 多条换行"
              />
            </el-form-item>
            <el-form-item label="权重列表" class="is-required">
              <el-input
                v-model="form.weight_list"
                type="textarea"
                :autosize="{ minRows: 2}"
                placeholder="格式：50 多条换行"
              />
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
import { serviceAddTcp, serviceDetail, serviceUpdateTcp } from '@/api/service'
export default {
  name: 'ServiceCreateTcp',
  data() {
    return {
      submitButDisabled: false,
      isEdit: false,
      form: {
        // info
        service_name: '',
        service_desc: '',
        // rule_type: 0,
        // rule: '',
        // need_https: 0,
        // need_strip_uri: 1,
        // need_websocket: 0,
        // url_rewrite: '',
        // header_transfor: '',

        // tcp
        port: '',

        // accesscontrol
        open_auth: 0,
        white_list: '',
        white_host_name: '', // 暂未使用，白名单主机名称
        black_list: '',
        clientip_flow_limit: '',
        service_flow_limit: '',

        // loadbalance
        round_type: 1,
        ip_list: '',
        weight_list: '',
        forbid_list: '' // 暂未使用
        // upstream_connect_timeout: '',
        // upstream_header_timeout: '',
        // upstream_idle_timeout: '',
        // upstream_max_idle: ''
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
      serviceDetail(query).then(response => {
        // info
        this.form.id = response.data.info.id
        this.form.service_name = response.data.info.service_name
        this.form.service_desc = response.data.info.service_desc
        // tcp_rule
        this.form.port = response.data.tcp_rule.port
        // http_rule
        // this.form.rule_type = response.data.http_rule.rule_type
        // this.form.rule = response.data.http_rule.rule
        // this.form.url_rewrite = response.data.http_rule.url_rewrite.replace(/,/g, '\n')
        // this.form.header_transfor = response.data.http_rule.header_transfor.replace(/,/g, '\n')
        // this.form.need_https = response.data.http_rule.need_https
        // this.form.need_strip_uri = response.data.http_rule.need_strip_uri
        // this.form.need_websocket = response.data.http_rule.need_websocket
        // accesscontrol
        this.form.open_auth = response.data.access_control.open_auth
        this.form.white_list = response.data.access_control.white_list.replace(/,/g, '\n')
        this.form.black_list = response.data.access_control.black_list.replace(/,/g, '\n')
        this.form.clientip_flow_limit = response.data.access_control.clientip_flow_limit
        this.form.service_flow_limit = response.data.access_control.service_flow_limit
        // loadbalance
        this.form.round_type = response.data.load_balance.round_type
        this.form.ip_list = response.data.load_balance.ip_list.replace(/,/g, '\n')
        this.form.weight_list = response.data.load_balance.weight_list.replace(/,/g, '\n')
      }).catch(() => {

      })
    },
    handleSubmit() {
      this.submitButDisabled = true
      const query = Object.assign({}, this.form)
      console.log(query)
      // 换行转 ',' 传递到后端
      //   query.url_rewrite = query.url_rewrite.replace(/\n/g, ',')
      //   query.header_transfor = query.header_transfor.replace(/\n/g, ',')
      query.white_list = query.white_list.replace(/\n/g, ',')
      query.black_list = query.black_list.replace(/\n/g, ',')
      query.ip_list = query.ip_list.replace(/\n/g, ',')
      query.weight_list = query.weight_list.replace(/\n/g, ',')
      // 字段类型转换 true false -> 1 0
      // 数字类型要从字符串转换成数字
      query.port = Number(query.port)
      query.clientip_flow_limit = Number(query.clientip_flow_limit)
      query.service_flow_limit = Number(query.service_flow_limit)
      //   query.upstream_connect_timeout = Number(query.upstream_connect_timeout)
      //   query.upstream_header_timeout = Number(query.upstream_header_timeout)
      //   query.upstream_idle_timeout = Number(query.upstream_idle_timeout)
      //   query.upstream_max_idle = Number(query.upstream_max_idle)

      if (this.isEdit) {
        serviceUpdateTcp(query).then(response => {
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
        serviceAddTcp(query).then(response => {
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
