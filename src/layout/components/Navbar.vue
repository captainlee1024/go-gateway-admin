<template>
  <div class="navbar">
    <hamburger
      id="hamburger-container"
      :is-active="sidebar.opened"
      class="hamburger-container"
      @toggleClick="toggleSideBar"
    />

    <breadcrumb id="breadcrumb-container" class="breadcrumb-container" />

    <div class="right-menu">
      <template v-if="device !== 'mobile'">
        <!-- search 不需要了 -->
        <!-- <search id="header-search" class="right-menu-item" /> -->

        <error-log class="errLog-container right-menu-item hover-effect" />

        <screenfull id="screenfull" class="right-menu-item hover-effect" />

        <el-tooltip content="Global Size" effect="dark" placement="bottom">
          <size-select id="size-select" class="right-menu-item hover-effect" />
        </el-tooltip>
      </template>

      <el-dropdown
        class="avatar-container right-menu-item hover-effect"
        trigger="click"
      >
        <div class="avatar-wrapper">
          <img :src="avatar + '?imageView2/1/w/80/h/80'" class="user-avatar">
          <i class="el-icon-caret-bottom" />
        </div>
        <el-dropdown-menu slot="dropdown">
          <a
            target="_blank"
            href="https://github.com/captainlee1024/go-gateway/"
          >
            <el-dropdown-item>Github</el-dropdown-item>
          </a>
          <a
            target="_blank"
            href="http://127.0.0.1:8080/swagger/index.html/"
          >
            <el-dropdown-item>Docs</el-dropdown-item>
          </a>
          <!-- 添加 changePwd 触发事件，去修改密码 -->
          <el-dropdown-item @click.native="changePwd">
            <span style="display: block;">修改密码</span>
          </el-dropdown-item>
          <!-- divided=true 分割线 -->
          <el-dropdown-item divided @click.native="logout">
            <span style="display: block;">退出</span>
          </el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </div>

    <!-- title这里不需要动态绑定，设置成固定的，fre为表单起一个唯一的名字，$refs['dataForm']可以取到它的值，
    rules校验规则，model是from下每一个子元素都关联到相同的
    form，需要设置template，每个字段设置默认值，再后面就是样式操作 -->
    <el-dialog :title="textMap['update']" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="70px" style="width: 400px; margin-left:50px;">
        <!-- 每个字段设置，这里需要两个输入框 -->
        <!-- label需要跟后端返回的名称对应 -->
        <el-form-item label="用户名" prop="username">
          <el-input v-model="temp.username" />
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="temp.password" show-password />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <!-- 点击 Cancel 的时候 dialogFormVisible 设置为 false,隐藏掉弹窗 -->
        <el-button @click="dialogFormVisible = false">
          Cancel
        </el-button>
        <!-- 点击进入到设置的 handleChangePwd 方法 -->
        <!-- <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()"> -->
        <el-button type="primary" @click="handleChangePwd">
          Confirm
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import Breadcrumb from '@/components/Breadcrumb'
import Hamburger from '@/components/Hamburger'
import ErrorLog from '@/components/ErrorLog'
import Screenfull from '@/components/Screenfull'
import SizeSelect from '@/components/SizeSelect'
import { changePwd } from '@/api/user'

export default {
  components: {
    Breadcrumb,
    Hamburger,
    ErrorLog,
    Screenfull,
    SizeSelect
  },
  data() {
    return {
      // 控制弹窗是否显示的值
      dialogFormVisible: false,
      textMap: {
        update: '修改密码'
      },
      // 数据绑定 密码修改时的表单的temp, 在传入后端的时候只接收password，可以复用这个temp
      temp: {
        username: this.$store.getters.name,
        password: undefined
      },
      // 各个数据的校验规则
      rules: {
        // required 是否必填 message 提示信息 trigger 什么时候进行校验 blur 当离开输入框的时候进行校验
        password: [{ required: true, message: 'password is required', trigger: 'blur' }]
        // type: [{ required: true, message: 'type is required', trigger: 'change' }],
        // timestamp: [{ type: 'date', required: true, message: 'timestamp is required', trigger: 'change' }],
        // title: [{ required: true, message: 'title is required', trigger: 'blur' }]
      }
    }
  },
  computed: {
    ...mapGetters(['sidebar', 'avatar', 'device'])
  },
  methods: {
    toggleSideBar() {
      this.$store.dispatch('app/toggleSideBar')
    },
    async logout() {
      await this.$store.dispatch('user/logout')
      this.$router.push(`/login?redirect=${this.$route.fullPath}`)
    },
    // 修改密码 触发的事件，点击触发，显示弹窗（就是吧控制显示的值设置为true）
    changePwd() {
      this.dialogFormVisible = true
      // console.log(this.$store.getters.name)
    },
    // 修改密码的业务函数
    handleChangePwd() {
      // 拿到表单的值进行校验
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          // 校验成功之后就可以拿到表单的值了
          changePwd(this.temp).then(() => {
            // 拿到值调到后端接口之后，隐藏表单
            this.dialogFormVisible = false
            // 给用户提示信息
            this.$notify({
              title: 'Success',
              message: '修改密码成功！',
              type: 'success',
              duration: 2000// 两秒之后隐藏
            })
          })
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.navbar {
  height: 50px;
  overflow: hidden;
  position: relative;
  background: #fff;
  box-shadow: 0 1px 4px rgba(0, 21, 41, 0.08);

  .hamburger-container {
    line-height: 46px;
    height: 100%;
    float: left;
    cursor: pointer;
    transition: background 0.3s;
    -webkit-tap-highlight-color: transparent;

    &:hover {
      background: rgba(0, 0, 0, 0.025);
    }
  }

  .breadcrumb-container {
    float: left;
  }

  .errLog-container {
    display: inline-block;
    vertical-align: top;
  }

  .right-menu {
    float: right;
    height: 100%;
    line-height: 50px;

    &:focus {
      outline: none;
    }

    .right-menu-item {
      display: inline-block;
      padding: 0 8px;
      height: 100%;
      font-size: 18px;
      color: #5a5e66;
      vertical-align: text-bottom;

      &.hover-effect {
        cursor: pointer;
        transition: background 0.3s;

        &:hover {
          background: rgba(0, 0, 0, 0.025);
        }
      }
    }

    .avatar-container {
      margin-right: 30px;

      .avatar-wrapper {
        margin-top: 5px;
        position: relative;

        .user-avatar {
          cursor: pointer;
          width: 40px;
          height: 40px;
          border-radius: 10px;
        }

        .el-icon-caret-bottom {
          cursor: pointer;
          position: absolute;
          right: -20px;
          top: 25px;
          font-size: 12px;
        }
      }
    }
  }
}
</style>
