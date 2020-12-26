<template>
  <div class="chart-container">
    <chart height="100%" width="100%" :data="charData" />
  </div>
</template>

<script>
import Chart from './components/LineMarker'
import { appDetail, appStat } from '@/api/app'

export default {
  name: 'ServiceStat',
  components: { Chart },
  data() {
    return { // 返回接口（接口从后端获取数据）里的值，绑定到标签中，传到Chart(LineMaker) 里进行数据的渲染
      charData: {
        'title': '',
        'today': [],
        'yesterday': []
      }
    }
  },
  // 页面创建的时候先执行该函数，从后端接口中拿数据，它会调用methods里的业务逻辑代码
  created() {
    const id = this.$route.params && this.$route.params.id
    this.fetchStat(id)
  },
  methods: {
    // 调用后端接口获取数据
    fetchStat(id) {
      const query = { 'id': id }
      appStat(query).then(responseStat => {
        appDetail(query).then(responseDetail => {
          this.charData = {
            'title': responseDetail.data.name + '流量统计',
            'today': responseStat.data.today,
            'yesterday': responseStat.data.yesterday
          }
        })
      })
    }
  }
}
</script>

<style scoped>
.chart-container{
  position: relative;
  width: 100%;
  height: calc(100vh - 84px);
}
</style>

