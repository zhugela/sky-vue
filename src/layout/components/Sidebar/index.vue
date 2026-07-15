<template>
  <div>
    <div class="logo">
      <!-- 闪送达 Logo - 美团外卖风格 -->
      <div v-if="!isCollapse" class="sidebar-logo">
        <div class="logo-icon">
          <!-- 外卖袋图标 -->
          <svg width="36" height="36" viewBox="0 0 40 40" fill="none" xmlns="http://www.w3.org/2000/svg">
            <circle cx="20" cy="20" r="18" fill="#FFD100"/>
            <path d="M12 16C12 14.8954 12.8954 14 14 14H26C27.1046 14 28 14.8954 28 16V28C28 29.1046 27.1046 30 26 30H14C12.8954 30 12 29.1046 12 28V16Z" fill="#2D3436"/>
            <path d="M15 14V12C15 9.23858 17.2386 7 20 7C22.7614 7 25 9.23858 25 12V14" stroke="#2D3436" stroke-width="2.5" stroke-linecap="round"/>
            <circle cx="20" cy="22" r="3" fill="#FFD100"/>
            <path d="M18.5 21.5L19.5 22.5L22 20" stroke="#2D3436" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </div>
        <span class="logo-text">闪送达</span>
      </div>
      <div v-else class="sidebar-logo-mini">
        <svg width="38" height="38" viewBox="0 0 40 40" fill="none" xmlns="http://www.w3.org/2000/svg">
          <circle cx="20" cy="20" r="18" fill="#FFD100"/>
          <path d="M12 16C12 14.8954 12.8954 14 14 14H26C27.1046 14 28 14.8954 28 16V28C28 29.1046 27.1046 30 26 30H14C12.8954 30 12 29.1046 12 28V16Z" fill="#2D3436"/>
          <path d="M15 14V12C15 9.23858 17.2386 7 20 7C22.7614 7 25 9.23858 25 12V14" stroke="#2D3436" stroke-width="2.5" stroke-linecap="round"/>
          <circle cx="20" cy="22" r="3" fill="#FFD100"/>
        </svg>
      </div>
    </div>
    <el-scrollbar wrap-class="scrollbar-wrapper">
      <el-menu :default-openeds="defOpen"
               :default-active="defAct"
               :collapse="isCollapse"
               :background-color="variables.menuBg"
               :text-color="variables.menuText"
               :active-text-color="variables.menuActiveText"
               :unique-opened="false"
               :collapse-transition="false"
               mode="vertical">
        <sidebar-item v-for="route in routes"
                      :key="route.path"
                      :item="route"
                      :base-path="route.path"
                      :is-collapse="isCollapse" />
        <!-- <div class="sub-menu">
          <div class="avatarName">
            {{ name }}
          </div>
          <div class="img">
            <img
              src="./../../../assets/icons/btn_close@2x.png"
              class="outLogin"
              alt="退出"
              @click="logout"
            />
          </div>
        </div> -->
      </el-menu>
    </el-scrollbar>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator'
import { AppModule } from '@/store/modules/app'
import { UserModule } from '@/store/modules/user'
import SidebarItem from './SidebarItem.vue'
import variables from '@/styles/_variables.scss'
import { getSidebarStatus, setSidebarStatus } from '@/utils/cookies'
import Cookies from 'js-cookie'
@Component({
  name: 'SideBar',
  components: {
    SidebarItem
  }
})
export default class extends Vue {
  private restKey: number = 0
  get name() {
    return (UserModule.userInfo as any).name
      ? (UserModule.userInfo as any).name
      : JSON.parse(Cookies.get('user_info') as any).name
  }
  get defOpen() {
    // const urlArr = this.$route.path.split('/')
    // const openStr = urlArr.length > 2 ? `/${urlArr[1]}` : '/'
    let path = ['/']
    this.routes.forEach((n: any, i: number) => {
      if (n.meta.roles && n.meta.roles[0] === this.roles[0]) {
        path.splice(0, 1, n.path)
      }
    })
    return path
  }

  get defAct() {
    let path = this.$route.path
    return path
  }

  get sidebar() {
    return AppModule.sidebar
  }

  get roles() {
    return UserModule.roles
  }

  get routes() {
    let routes = JSON.parse(
      JSON.stringify([...(this.$router as any).options.routes])
    )
    console.log('-=-=routes=-=-=', routes)
    console.log('-=-=routes=-=-=', this.roles[0])
    let menuList = []
    let menu = routes.find(item => item.path === '/')
    if (menu) {
      menuList = menu.children
    }
    console.log('-=-=routes=-wwww=-=', routes)
    return menuList
  }

  get variables() {
    return variables
  }

  get isCollapse() {
    return !this.sidebar.opened
  }
  private async logout() {
    this.$store.dispatch('LogOut').then(() => {
      // location.href = '/'
      this.$router.replace({ path: '/login' })
    })
    // this.$router.push(`/login?redirect=${this.$route.fullPath}`)
  }
}
</script>

<style lang="scss" scoped>
.logo {
  text-align: center;
  background: #FFD100;
  padding: 0 16px;
  height: 64px;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  z-index: 2;
}

.sidebar-logo {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  
  .logo-icon {
    display: flex;
    align-items: center;
    justify-content: center;
    filter: drop-shadow(0 2px 6px rgba(0, 0, 0, 0.15));
  }
  
  .logo-text {
    font-size: 22px;
    font-weight: 800;
    color: #2D3436;
    letter-spacing: 2px;
    font-family: 'PingFang SC', 'Microsoft YaHei', sans-serif;
  }
}

.sidebar-logo-mini {
  display: flex;
  align-items: center;
  justify-content: center;
  
  svg {
    filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.12));
  }
}

.el-scrollbar {
  height: 100%;
  background: #2D3436;
}

.el-menu {
  border: none;
  height: calc(100vh - 64px);
  width: 100% !important;
  padding: 16px 12px 0;
  background: transparent !important;
}
</style>
