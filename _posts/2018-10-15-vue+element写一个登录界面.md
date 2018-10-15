---
layout:     post
title:      vue+element写一个登录界面
subtitle:   登录功能实现
date:       2018-10-15
author:     Ayi
header-img: 
catalog: true
tags:
    - Vue+Element
---

API
```
  // 登录
  getLoginApi (name, pwd) {
    return this._doGetPromise(url4 + '/authorization/Gis/checkuser', {
      name: name,
      pwd: pwd
    })
  },
  // 判断token是否失效
  getLoginIsTimeout (token) {
    return this._doGetPromise(url4 + '/authorization/Gis/checkuserOnline', {
      token: token
    })
  }
```

login.vue
```
<template>
  <div class="login">
    <el-row class="content">
      <el-card class="box-card">
        <div slot="header" class="clearfix">
          <div class="logo">
            <img src="../../static/images/logo.png" height="40" alt="">
            <h1>登录界面</h1>
          </div>
        </div>
        <el-form ref="loginForm" :model="loginForm" :rules="rules" label-width="120px" @submit.native.prevent="null">
          <el-form-item label="账户：" prop="name">
            <el-input v-model="loginForm.name" placeholder="输入登录用户名" autofocus @change="handleInputChange"></el-input>
          </el-form-item>
          <el-form-item label="密码：" prop="password">
            <el-input type="password" v-model="loginForm.password" placeholder="请输入密码" @keyup.native.enter="onSubmit('loginForm')" @change="handleInputChange"></el-input>
          </el-form-item>
          <el-form-item>
            <el-checkbox label="记住账户" v-model="loginForm.remember"></el-checkbox>
          </el-form-item>
          <el-form-item class="buttons">
            <el-button type="primary" size="large" class="submitBtn" :loading="loginForm.loading" @click="onSubmit('loginForm')">{{loginForm.submitText}}</el-button>
            <el-button type="text" @click="resetFields('loginForm')">重置</el-button>
          </el-form-item>
          <el-form-item v-if="loginForm.isSubmit" style="margin-bottom:0;">
            <el-alert type="error" :title="loginForm.errorText" :closable="true" style="line-height: 1.2;" @close="loginForm.isSubmit=false"></el-alert>
          </el-form-item>
        </el-form>
      </el-card>
    </el-row>
  </div>
</template>
<script>
import base64 from 'base-64'
import Api from '@/api/Index.js'
export default {
  data () {
    return {
      loginForm: {
        name: '',
        password: '',
        remember: false,
        loading: false,
        isSubmit: false,
        errorText: '用户名密码错误',
        submitText: '登录'
      },
      rules: {
        name: [{
          required: true,
          message: '请输入账户名称',
          trigger: 'change'
        }],
        password: [{
          required: true,
          message: '请输入密码',
          trigger: 'change'
        }]
      }
    }
  },
  mounted () {
    const storage = window.localStorage
    console.log('mountned')

    if (storage.rememberName) {
      // 本地存储登录用户名
      console.log(storage.rememberName)
      this.loginForm.remember = true
      this.loginForm.name = storage.rememberName
    }

    if (storage.shareUserInfo) {
      // 本地用户信息有存储信息
      let shareUserInfo = JSON.parse(storage.shareUserInfo)
      let time = parseInt(shareUserInfo.time)
      let date = parseInt(shareUserInfo.date)
      if ((time + date) >= Math.round(new Date().getTime() / 1000)) {
        // 判断是否过期
        this.$router.replace('/')
      }
    }
  },
  methods: {
    handleInputChange () {
      this.loginForm.isSubmit = false
    },
    onSubmit (formName) {
      this.$refs[formName].validate((valid) => {
        this.loginForm.isSubmit = false
        if (valid) {
          this.loginForm.loading = true
          let psw = base64.encode(this.loginForm.password)
          Api.getLoginApi(this.loginForm.name, psw).then(data => {
            this.loginForm.loading = false
            let datas = data.data
            if (data.status === 0) {
              let userLocal = {
                name: this.loginForm.name,
                info: datas.result.userInfo,
                psw: psw,
                userguid: datas.result.userInfo.userguid,
                userName: datas.result.userInfo.username,
                token: datas.result.token,
                date: Math.round(new Date().getTime() / 1000),
                time: 3600 * 24 * 1
              }
              // 本地存储
              localStorage.setItem('shareUserInfo', JSON.stringify(userLocal))
              localStorage.removeItem('rememberName')
              if (this.loginForm.remember) {
                localStorage.setItem('rememberName', this.loginForm.name)
              }

              // vuex store
              let userState = datas.result.userInfo
              this.$store.dispatch('setUser', userState)

              // notify
              let message = '欢迎您' + userState.username
              this.$notify({
                title: '登陆成功',
                message: message,
                type: 'success'
              })
              this.$router.replace('/')
            } else {
              console.error(data)
              this.loginForm.errorText = '用户名或密码错误'
              this.loginForm.isSubmit = true
            }
          }).catch(res => {
            this.loginForm.lodading = false
            this.loginForm.isSubmit = true
            this.loginForm.errorText = '提交失败，服务有误'
          })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    resetFields (formName) {
      this.$refs[formName].resetFields()
    }
  }
}

</script>
<style lang="scss">
.login {
  background-image: url(../../static/images/login-bg.jpg);
  height: 1000px;

  .box-card {
    background: rgba(84, 92, 100, 0.2);
    ;
  }

  .logo {
    text-align: center;

    img {
      vertical-align: middle;
    }

    h1 {
      display: inline-block;
      margin-left: 20px;
      font-size: 30px;
      color: #c0c4cc;
      font-weight: 400;
      vertical-align: middle;
      line-height: 46px;
    }
  }

  .box-card {
    width: 520px;
    margin: 200px auto;

    .el-form {
      margin: 25px 25px 10px 0;

      .buttons {
        margin-top: 10px;
      }

      .submitBtn {
        width: 180px;
        margin-right: 15px;
      }

      .el-checkbox__label,
      .el-form-item__label {
        color: #c0c4cc;
      }
    }
  }

  .el-card,.el-input__inner {
    border: 1px solid #506596;
  }

  .el-card__header {
    border-bottom: 1px solid #506596;
  }

  .el-input__inner {
    background-color: rgba(255,255,255,0);
    color: #C0C4CC;
  }
}

</style>

```

header.vue
```
<template>
  <div class="header flexBw">
    <div class="title flexBw">
      <img src="../assets/img/logo.png" height="35" alt="">
      <router-link class="locationTitle" to="">header界面</router-link>
    </div>

    <div class="headerR flexBw">
      <div class="user-info flexBw">
        <div class="user-img" @click="handleUsercenter">
          <el-tooltip class="item" effect="dark" :content="username" placement="bottom">
            <div class="user-imgIn">
              <img src="../assets/img/people.jpg" height="36">
            </div>
          </el-tooltip>
        </div>
        <div class="user-more">
          <el-menu
            :default-active="activeIndex2"
            mode="horizontal"
            @select="handleSelect"
            background-color="#545c64"
            text-color="#fff"
            active-text-color="#fff">
            <el-submenu index="1">
              <template slot="title"><i class="iconfont icon-more"></i></template>
              <el-menu-item index="1-1" @click="handleUsercenter">个人中心</el-menu-item>
              <el-menu-item index="1-2" @click="handleLoginOut">退出</el-menu-item>
            </el-submenu>
          </el-menu>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Api from '@/api/Index.js'
export default {
  name: 'Header',
  data () {
    return {
      activeIndex1: '/home/locationMonitor',
      activeIndex2: '',
      enterOnoff: true,
      username: ''
    }
  },
  computed: {
    shareUserInfo () {
      return this.$store.state.shareUserInfo
    }
  },
  watch: {
    $route (val) {
      if (val.query.UserAccount) {
        this.init()
      }
    }
  },
  mounted () {
    // this.init()
  },
  methods: {
    isEmptyData (data) { // 判断数据是否为空，一般数据是object或array
      if (data instanceof Object) {
        var t
        for (t in data) {
          return false
        }
        return true
      } else if (data instanceof Array) {
        if (data.length === 0) return true
        return false
      }
    }, 
      
    init () {
      if (this.isEmptyData(this.shareUserInfo)) {
        const storage = window.localStorage
        if (storage.shareUserInfo) {
          // 判断本地是否有存储 本地有存储
          let shareUserInfo = {}
          shareUserInfo = JSON.parse(storage.shareUserInfo)
          this.username = shareUserInfo.userName
          let time = parseInt(shareUserInfo.time)
          let date = parseInt(shareUserInfo.date)
          if ((time + date) >= Math.round(new Date().getTime() / 1000)) {
            // 判断是否过期
            Api.getLoginIsTimeout(shareUserInfo.token).then(data => {
              if (data.status === 0) {
                let user = shareUserInfo.info
                this.$store.dispatch('setShareUser', user)
              } else {
                this.$store.dispatch('setShareUser', {})
                storage.removeItem('shareUserInfo')
                this.$message({
                  type: 'warning',
                  message: '登录失效，请重新登录'
                })
                this.$router.replace('/login')
              }
            }).catch(res => {
              this.$store.dispatch('setShareUser', {})
              storage.removeItem('shareUserInfo')
              this.$message({
                type: 'error',
                message: '登录失败，请重新登录'
              })
              this.$router.replace('/login')
            })
          } else {
            this.$router.replace('/login')
          }
        } else {
          // 本地没有存储
          this.$router.replace('/login')
        }
      }
    },
    // 切换选项
    handleSelect () {

    },
    // 个人中心
    handleUsercenter () {

    },
    // 退出
    handleLoginOut () {
      window.localStorage.removeItem('shareUserInfo')
      window.sessionStorage.setItem('redirect', this.$route.fullPath)
      this.$message({
        type: 'success',
        message: '用户成功退出！'
      })
      this.$router.replace('/login')
      this.$store.dispatch('setShareUser', {})
      this.enterOnoff = false
    }
  }
}
</script>

<style lang="scss">
  %height {
    height: 50px;
    line-height: 50px;
  }
  .header {
    width: 100%;
    height: 50px;
    background: #545c64;
    padding: 0 12px;
    box-sizing: border-box;

    .title { 
      font-size: 18px;

      img {
        margin-right: 4px;
      }
      a {
        color: #fbfcff;
      }
    }

    .headerR {
      /* 菜单 */
      .el-menu--horizontal {
        >.el-menu-item {
          @extend %height;
        }
        >.el-submenu {
          .el-submenu__title {
             @extend %height;
             padding: 0;
             border-bottom: none;
          }
          .el-submenu__icon-arrow {
            display: none;
          }
        }
        .el-menu-item {
          border: none !important;
        }
        .is-active {
          background-color: rgb(67, 74, 80) !important;
        }
      }
      /* 用户 */
      .user-img {
        width: 40px;
        height: 40px;
        border: solid 1px #acafb6;
        padding: 1px;
        color: #20a0ff;
        text-align: center;
        line-height: 42px;
        cursor: pointer;
        border-radius: 50%;
        margin: 0 12px;

        .user-imgIn {
          width: 36px;
          height: 36px;
          border-radius: 50%;
          overflow: hidden;
        }

        .iconfont {
          font-size: 24px;
        }
      }
      /* 更多 */
      .icon-more {
        font-size: 24px;
        border-bottom: none;
        color: #fbfcff;
      }
    }
  }
</style>

```

/store/index.js 
```
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    userInfo: {

    },
    shareUserInfo: {

    }
  },
  mutations: {
    SET_USER (state, payload) {
      state.userInfo = payload
    },
    SET_SHARE_USER (state, payload) {
      state.shareUserInfo = payload
    }
  },
  actions: {
    setUser ({ commit }, payload) {
      commit('SET_USER', payload)
    },
    setShareUser ({ commit }, payload) {
      commit('SET_SHARE_USER', payload)
    }
  },
  strict: false
})

export default store

```

/router/index.js
```
import Vue from 'vue'
import Router from 'vue-router'
import Home from '@/views/Home.vue'
import Login from '@/components/Login.vue'
import TrajectoryManage from '@/views/LocationMonitor/TrajectoryManage/Index.vue'
import PointTrajectory from '@/views/LocationMonitor/PointTrajectory/Index.vue'

Vue.use(Router)
export default new Router({
  routes: [
    {
      path: '/',
      name: 'App',
      redirect: '/home'
    }, {
      path: '/home',
      name: 'Home',
      component: Home,
      redirect: '/home/locationMonitor',
      children: [{
        path: 'locationMonitor',
        name: 'LocationMonitor',
        component: resolve => require(['@/views/LocationMonitor/Index.vue'], resolve)
      }]
    },
    {
      path: '/login',
      name: 'Login',
      component: Login
    }
  ]
})
```

