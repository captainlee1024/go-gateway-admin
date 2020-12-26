<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input v-model="listQuery.info" placeholder="服务名称/服务描述" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" />
      <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter">
        搜索
      </el-button>
      <router-link :to="'/app/app_create'">
        <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit">
          添加租户
        </el-button>
      </router-link>
    </div>

    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%;"
    >
      <el-table-column label="ID" prop="id" align="center" width="50">
        <template slot-scope="{row}">
          <span>{{ row.id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="app_id" min-width="100px">
        <template slot-scope="{row}">
          <span>{{ row.app_id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="租户名称" min-width="120px">
        <template slot-scope="{row}">
          <span>{{ row.name }}</span>
        </template>
      </el-table-column>
      <el-table-column label="秘钥" min-width="160px">
        <template slot-scope="{row}">
          <span>{{ row.secret }}</span>
        </template>
      </el-table-column>
      <el-table-column label="QPS" width="80px">
        <template slot-scope="{row}">
          <span>{{ row.qps }}</span>
        </template>
      </el-table-column>
      <el-table-column label="日请求量" width="80px">
        <template slot-scope="{row}">
          <span>{{ row.qpd }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="300" class-name="small-padding fixed-width">
        <template slot-scope="{row,$index}">
          <router-link :to="'/app/app_stat/'+row.id">
            <el-button type="primary" size="mini">
              统计
            </el-button>
          </router-link>&nbsp;
          <router-link :to="'/app/app_edit/'+row.id">
            <el-button type="primary" size="mini">
              修改
            </el-button>
          </router-link>&nbsp;
          <el-button size="mini" type="danger" @click="handleDelete(row,$index)">
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total>0" :total="total" :page.sync="listQuery.page_no" :limit.sync="listQuery.page_size" @pagination="getList" />

  </div>
</template>

<script>
import { appList, appDelete } from '@/api/app'
import waves from '@/directive/waves' // waves directive
import Pagination from '@/components/Pagination' // secondary package based on el-pagination

const loadTypeOptions = [
  { key: '0', display_name: 'HTTP' },
  { key: '1', display_name: 'TCP' },
  { key: '2', display_name: 'GRPC' }
]

// arr to obj, such as { CN : "China", US : "USA" }
const loadTypeKeyValue = loadTypeOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})

export default {
  name: 'AppList',
  components: { Pagination },
  directives: { waves },
  filters: {
    loadTypeFilter(type) {
      return loadTypeKeyValue[type]
    }
  },
  data() {
    return {
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: true,
      // 分页查询参数
      listQuery: {
        page_no: 1,
        page_size: 20,
        info: ''
      },
      // 点击某一行的时候，数据会拷贝到temp里，然后进行查询
      temp: {
        id: undefined
      },
      textMap: {
        update: 'Edit',
        create: 'Create'
      }
    }
  },
  // 试图在渲染之前掉的方法
  created() {
    // 这里调用了下面定义在 methods 里的 getList 方法
    this.getList()
  },
  methods: {
    getList() {
      this.listLoading = true
      // 调用我们定义的查询方法
      appList(this.listQuery).then(response => {
        // 我们前端返回的 data 中有两项分别是
        // list (查询的服务列表)
        // total 一共多少条记录
        this.list = response.data.list
        this.total = response.data.total

        // Just to simulate the time of the request
        // 超时时间，1.5 秒后隐藏掉 Loding
        setTimeout(() => {
          this.listLoading = false
        }, 1.5 * 1000)
      })
    },
    // 这里是拦截设置每次只要是第一次进入查询，起始页码都为1
    handleFilter() {
      this.listQuery.page_no = 1
      this.getList()
    },
    handleDelete(row, index) {
      this.$confirm('此操作将永久删除该服务, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => { // 确定
        const deleteQuery = {
          'id': row.id
        }
        appDelete(deleteQuery).then(response => {
          this.$notify({
            title: 'Success',
            message: '删除成功',
            type: 'success',
            duration: 2000
          })
          // this.list.splice(index, 1)
          this.getList()
        })
      }).catch(() => { // 取消
        this.$notify({
          title: 'Success',
          message: '已取消删除',
          type: 'info',
          duration: 2000
        })
      })
    }
  }
}
</script>
