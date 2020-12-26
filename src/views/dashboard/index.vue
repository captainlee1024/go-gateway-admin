<template>
  <div class="dashboard-editor-container">
    <!-- 右上角 github 图标 -->
    <!-- <github-corner class="github-corner" /> -->

    <!-- 四个指标 -->
    <panel-group :data="panelGroupData" />

    <!-- 总流量统计，放在下面的小卡片中展示了 -->
    <!-- <el-row style="background:#fff;padding:16px 16px 0;margin-bottom:32px;">
      <line-chart :chart-data="lineChartData" />
    </el-row> -->

    <el-row :gutter="32">
      <el-col :xs="24" :sm="24" :lg="16">
        <div class="chart-wrapper">
          <!-- 总流量统计 -->
          <line-chart :chart-data="lineChartData" />
        </div>
      </el-col>
      <el-col :xs="24" :sm="24" :lg="8">
        <div class="chart-wrapper">
          <!-- 服务类型占比概览 -->
          <pie-chart :chart-data="pieChartData" />
        </div>
      </el-col>
    </el-row>

    <!-- 预留的综合图 -->
    <el-row style="background:#fff;padding:16px 16px 0;margin-bottom:32px;">
      <chart height="800px" width="100%" :chart-data="mixChartData" />
    </el-row>

  </div>
</template>

<script>
import PanelGroup from './components/PanelGroup'
import LineChart from './components/LineChart'
import PieChart from './components/PieChart'
import Chart from './components/MixChart'
import { panelGroupData, flowStat, serviceStat } from '@/api/dashboard'

// 用来动态的选择渲染哪些数据，在４各指标中选择，这里暂时没有需求
// const lineChartData = {
//   newVisitis: {
//     expectedData: [100, 120, 161, 134, 105, 160, 165],
//     actualData: [120, 82, 91, 154, 162, 140, 145]
//   },
//   messages: {
//     expectedData: [200, 192, 120, 144, 160, 130, 140],
//     actualData: [180, 160, 151, 106, 145, 150, 130]
//   },
//   purchases: {
//     expectedData: [80, 100, 121, 104, 105, 90, 100],
//     actualData: [120, 90, 100, 138, 142, 130, 130]
//   },
//   shoppings: {
//     expectedData: [130, 140, 141, 142, 145, 150, 160],
//     actualData: [120, 82, 91, 154, 162, 140, 130]
//   }
// }

export default {
  name: 'DashboardAdmin',
  components: {
    PanelGroup,
    LineChart,
    PieChart,
    Chart
  },
  data() {
    return {
      panelGroupData: {
        'service_num': 23,
        'today_request_num': 10000,
        'current_qps': 100,
        'app_num': 3
      },
      lineChartData: {
        'title': '今日流量统计',
        'today': [],
        'yesterday': []
      },
      pieChartData: {
        'title': '服务类型占比',
        'legend': [],
        'series': []
      },
      mixChartData: {
        'title': '综合图',
        'sum': [620, 574, 507, 463, 452, 475, 422, 360, 512, 535, 494, 494],
        'http': [220, 182, 191, 134, 150, 120, 110, 125, 145, 122, 165, 122],
        'tcp': [120, 110, 125, 145, 122, 165, 122, 220, 182, 191, 134, 150],
        'grpc': [280, 282, 291, 184, 180, 190, 190, 185, 185, 222, 195, 222]
      }
    }
  },
  created() {
    this.fetchPanelGroupData()
    this.fetchFlowStat()
    this.fetchServiceStat()
    this.fetchMixChartData()
  },
  methods: {
    fetchPanelGroupData() {
      panelGroupData({}).then(response => {
        // this.panelGroupData.service_num = response.data.service_num
        // this.panelGroupData.today_request_num = response.data.today_request_num
        // this.panelGroupData.current_qps = response.data.current_qps
        // this.panelGroupData.app_num = response.data.app_num
        this.panelGroupData = response.data
      }).catch(() => {
      })
    },
    fetchFlowStat() {
      flowStat({}).then(response => {
        // this.panelGroupData.service_num = response.data.service_num
        // this.panelGroupData.today_request_num = response.data.today_request_num
        // this.panelGroupData.current_qps = response.data.current_qps
        // this.panelGroupData.app_num = response.data.app_num
        this.lineChartData.today = response.data.today
        this.lineChartData.yesterday = response.data.yesterday
      }).catch(() => {
      })
    },
    fetchServiceStat() {
      serviceStat({}).then(response => {
        // this.panelGroupData.service_num = response.data.service_num
        // this.panelGroupData.today_request_num = response.data.today_request_num
        // this.panelGroupData.current_qps = response.data.current_qps
        // this.panelGroupData.app_num = response.data.app_num
        this.pieChartData.legend = response.data.legend
        this.pieChartData.series = response.data.data
      }).catch(() => {
      })
    },
    fetchMixChartData() {
      // 预留图，待完善
      this.mixChartData.title = '混合图（预留接口，仅展示 MOCK 数据）'
    },
    handleSetLineChartData(type) {
      // this.lineChartData = lineChartData[type]
    }
  }
}
</script>

<style lang="scss" scoped>
.dashboard-editor-container {
  padding: 32px;
  background-color: rgb(240, 242, 245);
  position: relative;

  .github-corner {
    position: absolute;
    top: 0px;
    border: 0;
    right: 0;
  }

  .chart-wrapper {
    background: #fff;
    padding: 16px 16px 0;
    margin-bottom: 32px;
  }
}

@media (max-width:1024px) {
  .chart-wrapper {
    padding: 8px;
  }
}
</style>
