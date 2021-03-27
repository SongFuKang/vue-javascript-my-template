<template>
  <div class="top-nav">
    <div class="log">后台管理系统</div>
    <i v-if="isScroll" class="el-icon-arrow-left leftIcon" @click="leftmove" />
    <div id="classifyScroll" class="module-list-wrap">
      <el-menu
        :active-text-color="variables.menuActiveText"
        :default-active="activeMenu"
        mode="horizontal"
        @select="handleSelect"
      >
        <div v-for="item in routes" :key="item.path" class="nav-item">
          <app-link :to="resolvePath(item)">
            <el-menu-item
              v-if="!item.hidden"
              :index="item.path"
            >{{ item.meta ? item.meta.title : item.children[0].meta.title }}</el-menu-item>
          </app-link>
        </div>
      </el-menu>
    </div>
    <i v-if="isScroll" class="el-icon-arrow-right rightIcon" @click="rightmove" />
    <div class="right-menu">
      <el-dropdown class="avatar-container" trigger="click">
        <img :src="avatar+'?imageView2/1/w/80/h/80'" class="user-avatar">
        <el-dropdown-menu slot="dropdown" style="padding: 6px 0;width: 110px;text-align: center;">
          <router-link to="/">
            <el-dropdown-item>
              Home
            </el-dropdown-item>
          </router-link>
          <a target="_blank" href="https://github.com/PanJiaChen/vue-admin-template/">
            <el-dropdown-item>Github</el-dropdown-item>
          </a>
          <a target="_blank" href="https://github.com/SongFuKangM/vue-admin-template-demo">
            <el-dropdown-item>userGithub</el-dropdown-item>
          </a>
          <a target="_blank" href="https://panjiachen.github.io/vue-element-admin-site/#/">
            <el-dropdown-item>Docs</el-dropdown-item>
          </a>
          <el-dropdown-item style="line-height: 30px;" @click.native="logout">
            <span>退出登录</span>
          </el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import AppLink from './Sidebar/Link'
import { constantRoutes } from '@/router'
import variables from '@/styles/variables.scss'
import { isExternal } from '@/utils/validate'

export default {
  name: 'Topbar',
  components: {
    AppLink
  },
  data() {
    return {
      clock: null,
      isScroll: false,
      routes: constantRoutes
    }
  },
  computed: {
    activeMenu() {
      const route = this.$route
      const { meta, path } = route
      // if set path, the sidebar will highlight the path you set
      if (meta.activeMenu) {
        return meta.activeMenu
      }
      // 如果是首页，首页高亮
      if (path === '/dashboard') {
        return '/'
      }
      // 如果不是首页，高亮一级菜单
      const activeMenu = '/' + path.split('/')[1]
      return activeMenu
    },
    variables() {
      return variables
    },
    sidebar() {
      return this.$store.state.app.sidebar
    },
    ...mapGetters(['avatar'])
  },
  mounted() {
    // 初始化判断
    this.handleScroll()
    // 全局窗口变化监听，判断父元素和子元素的大小，从而控制isScroll的开关
    window.addEventListener('resize', this.handleScroll)
    this.initCurrentRoutes()
  },
  methods: {
    // 判断是否有滚动条
    handleScroll() {
      const div = document.getElementsByClassName('el-menu')[0]
      if ((div.scrollWidth > div.clientWidth) || (div.offsetWidth > div.clientWidth)) {
        this.isScroll = true
      } else {
        clearInterval(this.clock)
        this.isScroll = false
      }
    },
    // 向左侧滚动
    leftmove() {
      clearInterval(this.clock)
      const div = document.getElementsByClassName('el-menu')[0]
      const num = div.scrollLeft - 240 > 0 ? div.scrollLeft - 240 : 0
      // 添加动画效果
      this.clock = setInterval(function() {
        if (div.scrollLeft !== num) {
          div.scrollLeft -= 10
        } else {
          clearInterval(this.clock)
        }
      }, 10)
    },
    // 向右侧滚动
    rightmove() {
      clearInterval(this.clock)
      const div = document.getElementsByClassName('el-menu')[0]
      const num = div.scrollLeft + 240 < div.scrollWidth ? div.scrollLeft + 240 : div.scrollWidth
      // 添加动画效果
      this.clock = setInterval(function() {
        if (div.scrollLeft !== num) {
          div.scrollLeft += 10
        } else {
          clearInterval(this.clock)
        }
      }, 10)
    },
    // 通过当前路径找到二级菜单对应项，存到store，用来渲染左侧菜单
    initCurrentRoutes() {
      const { path } = this.$route
      let route = this.routes.find(
        item => item.path === '/' + path.split('/')[1]
      )
      // 如果找不到这个路由，说明是首页
      if (!route) {
        route = this.routes.find(item => item.path === '/')
      }
      this.$store.commit('permission/SET_CURRENT_ROUTES', route)
      this.setSidebarHide(route)
    },
    // 判断该路由是否只有一个子项或者没有子项，如果是，则在一级菜单添加跳转路由
    isOnlyOneChild(item) {
      if (item.children && item.children.length === 1) {
        return true
      }
      return false
    },
    resolvePath(item) {
      // 如果是个完成的url直接返回
      if (isExternal(item.path)) {
        return item.path
      }
      // 如果是首页，就返回重定向路由
      if (item.path === '/') {
        const path = item.redirect
        return path
      }

      // 如果有子项，默认跳转第一个子项路由
      let path = ''
      /**
       * item 路由子项
       * parent 路由父项
       */
      const getDefaultPath = (item, parent) => {
        // 如果path是个外部链接（不建议），直接返回链接，存在个问题：如果是外部链接点击跳转后当前页内容还是上一个路由内容
        if (isExternal(item.path)) {
          path = item.path
          return
        }
        // 第一次需要父项路由拼接，所以只是第一个传parent
        if (parent) {
          path += (parent.path + '/' + item.path)
        } else {
          path += ('/' + item.path)
        }
        // 如果还有子项，继续递归
        if (item.children) {
          getDefaultPath(item.children[0])
        }
      }
      if (item.children) {
        getDefaultPath(item.children[0], item)
        return path
      }

      return item.path
    },
    handleSelect(key, keyPath) {
      // 把选中路由的子路由保存store
      const route = this.routes.find(item => item.path === key)
      this.$store.commit('permission/SET_CURRENT_ROUTES', route)
      this.setSidebarHide(route)
    },
    // 设置侧边栏的显示和隐藏
    setSidebarHide(route) {
      if (!route.children || route.children.length === 1) {
        this.$store.dispatch('app/toggleSideBarHide', true)
      } else {
        this.$store.dispatch('app/toggleSideBarHide', false)
      }
    },
    async logout() {
      await this.$store.dispatch('user/logout')
      this.$router.push(`/login?redirect=${this.$route.fullPath}`)
    }
  }
}
</script>

<style lang="scss" scoped>
@import "~@/styles/_main.scss";
@import "~@/styles/variables.scss";
.header-bar {
  @include box(row, space-between);
  @include size(100vw, #{$headerHeight});
  background-color: $menuBg;
  z-index: 999;
  position: fixed;
  top: 0;
}

.header-name {
  font-size: 24px;
  color: #ffffff;
  padding-left: 20px;
}

.right-menu {
  float: right;
  height: 100%;
  line-height: 14px;
  padding-right: 20px;
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
    margin-right: 10px;
    margin-top: 14px;
    .user-avatar {
      cursor: pointer;
      width: 40px;
      height: 26px;
      margin-top: 2px;
      border-radius: 10px;
    }
  }
}
</style>
