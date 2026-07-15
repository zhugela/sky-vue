<template>
  <div class="navbar">
    <div class="statusBox">
      <hamburger id="hamburger-container"
                 :is-active="sidebar.opened"
                 class="hamburger-container"
                 @toggleClick="toggleSideBar" />
      <span v-if="status===1"
            class="businessBtn">营业中</span>
      <span v-else
            class="businessBtn closing">打烊中</span>
    </div>

    <div :key="restKey"
         class="right-menu">
      <div class="rightStatus">
        <audio ref="audioVo"
               hidden>
          <source src="./../../../assets/preview.mp3" type="audio/mp3" />
        </audio>
        <audio ref="audioVo2"
               hidden>
          <source src="./../../../assets/reminder.mp3" type="audio/mp3" />
        </audio>
        <span class="navicon operatingState" @click="handleStatus"><i />营业状态设置</span>
      </div>
      <div class="avatar-wrapper">
        <div :class="shopShow?'userInfo':''"
             @mouseenter="toggleShow"
             @mouseleave="mouseLeaves">
          <el-button type="primary"
                     :class="shopShow?'active':''">
            {{ name }}<i class="el-icon-arrow-down" />
          </el-button>
          <div v-if="shopShow"
               class="userList">
            <p class="amendPwdIcon"
               @click="handlePwd">
              修改密码<i />
            </p>
            <p class="outLogin"
               @click="logout">
              退出登录<i />
            </p>
          </div>
        </div>
      </div>
    </div>
    <!-- 营业状态弹层 -->
    <el-dialog title="营业状态设置"
               :visible.sync="dialogVisible"
               width="25%"
               :show-close="false">
      <el-radio-group v-model="setStatus">
        <el-radio :label="1">
          营业中
          <span>当前餐厅处于营业状态，自动接收任何订单，可点击打烊进入店铺打烊状态。</span>
        </el-radio>
        <el-radio :label="0">
          打烊中
          <span>当前餐厅处于打烊状态，仅接受营业时间内的预定订单，可点击营业中手动恢复营业状态。</span>
        </el-radio>
      </el-radio-group>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="handleSave">确 定</el-button>
      </span>
    </el-dialog>
    <!-- end -->
    <!-- 修改密码 -->
    <Password :dialog-form-visible="dialogFormVisible"
              @handleclose="handlePwdClose" />
    <!-- end -->
  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from 'vue-property-decorator'
import { AppModule } from '@/store/modules/app'
import { UserModule } from '@/store/modules/user'
import Breadcrumb from '@/components/Breadcrumb/index.vue'
import Hamburger from '@/components/Hamburger/index.vue'
import { getStatus, setStatus } from '@/api/users'
import Cookies from 'js-cookie'
import { debounce, throttle } from '@/utils/common'
import { setNewData, getNewData } from '@/utils/cookies'

// 接口
import { getCountUnread } from '@/api/inform'
// 修改密码弹层
import Password from '../components/password.vue'

@Component({
  name: 'Navbar',
  components: {
    Breadcrumb,
    Hamburger,
    Password,
  },
})
export default class extends Vue {
  private storeId = this.getStoreId
  private restKey: number = 0
  private websocket = null
  private newOrder = ''
  private message = ''
  private audioIsPlaying = false
  private audioPaused = false
  private statusValue = true
  private audioUrl: './../../../assets/preview.mp3'
  private shopShow = false
  private dialogVisible = false
  private status = 1
  private setStatus = 1
  private dialogFormVisible = false
  private ountUnread = 0
  // get ountUnread() {
  //   return Number(getNewData())
  // }
  get sidebar() {
    return AppModule.sidebar
  }

  get device() {
    return AppModule.device.toString()
  }

  getuserInfo() {
    return UserModule.userInfo
  }

  get name() {
    return (UserModule.userInfo as any).name
      ? (UserModule.userInfo as any).name
      : JSON.parse(Cookies.get('user_info') as any).name
  }

  get getStoreId() {
    let storeId = ''
    if (UserModule.storeId) {
      storeId = UserModule.storeId
    } else if ((UserModule.userInfo as any).stores != null) {
      storeId = (UserModule.userInfo as any).stores[0].storeId
    }
    return storeId
  }
  mounted() {
    document.addEventListener('click', this.handleClose)
    //console.log(this.$store.state.app.statusNumber)
    // const msg = {
    //   data: {
    //     type: 2,
    //     content: '订单1653904906519客户催单，已下单23分钟，仍未接单。',
    //     details: '434'
    //   }
    // }
    this.getStatus()
  }
  created() {
    this.webSocket()
  }
  onload() {
  }
  destroyed() {
    this.websocket.close() //离开路由之后断开websocket连接
  }

  // 添加新订单提示弹窗
  webSocket() {
    const that = this as any
    let clientId = Math.random().toString(36).substr(2)
    let socketUrl = process.env.VUE_APP_SOCKET_URL + clientId
    console.log(socketUrl, 'socketUrl')
    if (typeof WebSocket == 'undefined') {
      that.$notify({
        title: '提示',
        message: '当前浏览器无法接收实时报警信息，请使用谷歌浏览器！',
        type: 'warning',
        duration: 0,
      })
    } else {
      this.websocket = new WebSocket(socketUrl)
      // 监听socket打开
      this.websocket.onopen = function () {
        console.log('浏览器WebSocket已打开')
      }
      // 监听socket消息接收
      this.websocket.onmessage = function (msg) {
        // 转换为json对象
        that.$refs.audioVo.currentTime = 0
        that.$refs.audioVo2.currentTime = 0

        console.log(msg, JSON.parse(msg.data), 'msg')
        // const h = this.$createElement
        const jsonMsg = JSON.parse(msg.data)
        if (jsonMsg.type === 1) {
          that.$refs.audioVo.play()
        } else if (jsonMsg.type === 2) {
          that.$refs.audioVo2.play()
        }
        that.$notify({
          title: jsonMsg.type === 1 ? '待接单' : '催单',
          duration: 0,
          dangerouslyUseHTMLString: true,
          onClick: () => {
            that.$router
              .push(`/order?orderId=${jsonMsg.orderId}`)
              .catch((err) => {
                console.log(err)
              })
            setTimeout(() => {
              location.reload()
            }, 100)
          },
          // 这里也可以把返回信息加入到message中显示
          message: `${
            jsonMsg.type === 1
              ? `<span>您有1个<span style=color:#419EFF>订单待处理</span>,${jsonMsg.content},请及时接单</span>`
              : `${jsonMsg.content}<span style='color:#419EFF;cursor: pointer'>去处理</span>`
          }`,
        })
      }
      // 监听socket错误
      this.websocket.onerror = function () {
        that.$notify({
          title: '错误',
          message: '服务器错误，无法接收实时报警信息',
          type: 'error',
          duration: 0,
        })
      }
      // 监听socket关闭
      this.websocket.onclose = function () {
        console.log('WebSocket已关闭')
      }
    }
  }

  private toggleSideBar() {
    AppModule.ToggleSideBar(false)
  }
  // 退出
  private async logout() {
    this.$store.dispatch('LogOut').then(() => {
      // location.href = '/'
      this.$router.replace({ path: '/login' })
    })
    // this.$router.push(`/login?redirect=${this.$route.fullPath}`)
  }
  // 获取未读消息
  async getCountUnread() {
    const { data } = await getCountUnread()
    if (data.code === 1) {
      // this.ountUnread = data.data
      AppModule.StatusNumber(data.data)
      // setNewData(data.data)
      // this.$message.success('操作成功！')
    } else {
      this.$message.error(data.msg)
    }
  }
  // 营业状态
  async getStatus() {
    const { data } = await getStatus()
    this.status = data.data
    this.setStatus = this.status
  }
  // 下拉菜单显示
  toggleShow() {
    this.shopShow = true
  }
  // 下拉菜单隐藏
  mouseLeaves() {
    this.shopShow = false
  }
  // 触发空白处下来菜单关闭
  handleClose() {
    // clearTimeout(this.leave)
    // this.shopShow = false
  }
  // 设置营业状态
  handleStatus() {
    this.dialogVisible = true
  }
  // 营业状态设置
  async handleSave() {
    const { data } = await setStatus(this.setStatus)
    if (data.code === 1) {
      this.dialogVisible = false
      this.getStatus()
    }
  }
  // 修改密码
  handlePwd() {
    this.dialogFormVisible = true
  }
  // 关闭密码编辑弹层
  handlePwdClose() {
    this.dialogFormVisible = false
  }
}
</script>

<style lang="scss" scoped>
.navbar {
  height: 64px;
  position: relative;
  background: #FFD100;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  
  .statusBox {
    float: left;
    height: 100%;
    align-items: center;
    display: flex;
  }
  
  .hamburger-container {
    padding: 0 16px 0 20px;
    cursor: pointer;
    transition: all 0.25s ease;
    -webkit-tap-highlight-color: transparent;
    color: #2D3436;

    &:hover {
      background: rgba(0, 0, 0, 0.06);
      border-radius: 8px;
    }
  }

  .breadcrumb-container {
    float: left;
  }
  
  .right-menu {
    float: right;
    margin-right: 20px;
    color: #2D3436;
    font-size: 14px;
    display: flex;
    align-items: center;
    height: 64px;

    span {
      padding: 8px 16px;
      width: auto;
      display: inline-block;
      cursor: pointer;
      border-radius: 20px;
      transition: all 0.25s ease;
      font-weight: 500;
      
      &:hover {
        background: rgba(0, 0, 0, 0.08);
      }
    }
    
    .amendPwdIcon {
      i {
        width: 18px;
        height: 18px;
        background: url(./../../../assets/icons/btn_gaimi@2x.png) no-repeat;
        background-size: contain;
        margin-top: 8px;
      }
    }
    
    .outLogin {
      i {
        width: 18px;
        height: 18px;
        background: url(./../../../assets/icons/btn_close@2x.png) no-repeat 100% 100%;
        background-size: contain;
        margin-top: 8px;
      }
    }
    
    .outLogin {
      cursor: pointer;
    }

    &:focus {
      outline: none;
    }

    .right-menu-item {
      display: inline-block;
      padding: 0 12px;
      height: 100%;
      font-size: 18px;
      color: #2D3436;
      vertical-align: text-bottom;

      &.hover-effect {
        cursor: pointer;
        transition: all 0.25s ease;

        &:hover {
          background: rgba(0, 0, 0, 0.06);
        }
      }
    }
  }
  
  .rightStatus {
    display: flex;
    align-items: center;
  }
  
  .avatar-wrapper {
    margin-left: 12px;
    position: relative;
    width: 140px;
    text-align: left;
    
    .user-avatar {
      cursor: pointer;
      width: 40px;
      height: 40px;
      border-radius: 50%;
    }

    .el-icon-caret-bottom {
      cursor: pointer;
      position: absolute;
      right: -20px;
      top: 25px;
      font-size: 12px;
    }

    .el-button--primary {
      background: #2D3436;
      border-radius: 20px;
      padding-top: 0px;
      padding-bottom: 0px;
      position: relative;
      width: 140px;
      padding-left: 16px;
      text-align: left;
      border: none;
      height: 36px;
      line-height: 36px;
      color: #fff;
      font-weight: 600;
      transition: all 0.25s ease;
      
      &:hover {
        background: #3D4448;
      }
      
      &.active {
        background: #3D4448;
        
        .el-icon-arrow-down {
          transform: rotate(-180deg);
        }
      }
    }
  }
  
  .businessBtn {
    height: 28px;
    line-height: 26px;
    background: #00C853;
    border: none;
    border-radius: 14px;
    display: inline-block;
    padding: 0 14px;
    color: #fff;
    font-weight: 600;
    font-size: 12px;
    letter-spacing: 0.5px;
    box-shadow: 0 2px 8px rgba(0, 200, 83, 0.3);
  }
  
  .closing {
    background: #636E72;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  }
  
  .navicon {
    i {
      display: inline-block;
      width: 18px;
      height: 18px;
      vertical-align: sub;
      margin: 0 6px 0 0;
    }
  }
  
  .operatingState {
    i {
      background: url('./../../../assets/icons/time.png') no-repeat;
      background-size: contain;
    }
  }
  
  .mesCenter {
    i {
      background: url('./../../../assets/icons/msg.png') no-repeat;
      background-size: contain;
    }
  }
}
</style>
<style lang="scss">
.el-notification {
  width: 420px !important;
  border-radius: 16px !important;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.12) !important;
  border: 1px solid rgba(0, 0, 0, 0.05);
  
  .el-notification__title {
    margin-bottom: 14px;
    color: #2D3436;
    font-weight: 600;
    
    .el-notification__content {
      color: #636E72;
    }
  }
}

.navbar {
  .el-dialog {
    min-width: auto !important;
    border-radius: 16px !important;
    overflow: hidden;
  }
  
  .el-dialog__header {
    height: 64px;
    line-height: 64px;
    background: #FFD100;
    padding: 0 30px;
    font-size: 17px;
    color: #2D3436;
    font-weight: 700;
    border: 0 none;
  }
  
  .el-dialog__body {
    padding: 20px 30px 30px;
    
    .el-radio,
    .el-radio__input {
      white-space: normal;
    }
    
    .el-radio__label {
      padding-left: 8px;
      color: #2D3436;
      font-weight: 600;
      
      span {
        display: block;
        line-height: 22px;
        padding-top: 12px;
        color: #636E72;
        font-weight: normal;
      }
    }
    
    .el-radio__input.is-checked .el-radio__inner {
      background: #FFD100;
      border-color: #FFD100;
      
      &::after {
        background: #2D3436;
      }
    }
    
    .el-radio-group {
      & > .is-checked {
        border: 2px solid #FFD100;
        background: #FFF9E6;
      }
    }
    
    .el-radio {
      width: 100%;
      background: #F5F6FA;
      border: 2px solid #DFE6E9;
      border-radius: 12px;
      padding: 18px 24px;
      margin-top: 16px;
      transition: all 0.25s ease;
      
      &:hover {
        border-color: #FFD100;
        background: #FFF9E6;
      }
    }
  }
  
  .el-badge__content.is-fixed {
    top: 24px;
    right: 2px;
    width: 20px;
    height: 20px;
    font-size: 11px;
    line-height: 18px;
    border-radius: 50%;
    padding: 0;
    background: #FF5252;
    border: 2px solid #fff;
    box-shadow: 0 2px 8px rgba(255, 82, 82, 0.4);
  }
  
  .badgeW {
    .el-badge__content.is-fixed {
      width: 32px;
      border-radius: 12px;
    }
  }
}

.el-icon-arrow-down {
  background: url('./../../../assets/icons/up.png') no-repeat 50% 50%;
  background-size: contain;
  width: 10px;
  height: 10px;
  transform: rotate(0deg);
  margin-left: 16px;
  position: absolute;
  right: 14px;
  top: 13px;
  filter: brightness(0) invert(1);
  transition: transform 0.25s ease;
  
  &:before {
    content: '';
  }
}

.userInfo {
  background: #fff;
  position: absolute;
  top: 0px;
  left: 0;
  z-index: 99;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
  width: 100%;
  border-radius: 16px;
  line-height: 36px;
  padding: 0 0 8px;
  height: auto;
  border: 1px solid rgba(0, 0, 0, 0.05);
  
  .userList {
    width: 95%;
    padding: 8px 5px 0;
  }
  
  p {
    cursor: pointer;
    height: 42px;
    line-height: 42px;
    padding: 0 14px;
    border-radius: 10px;
    margin: 4px 0;
    transition: all 0.2s ease;
    color: #636E72;
    font-size: 14px;
    font-weight: 500;
    
    i {
      margin-left: 10px;
      vertical-align: middle;
      margin-top: 4px;
      float: right;
    }
    
    &:hover {
      background: #FFF9E6;
      color: #2D3436;
    }
  }
}

.msgTip {
  color: #FF9500;
  padding: 0 5px;
  font-weight: 600;
}
// .el-dropdown{
//   .el-button--primary{
//     height: 32px;
//     background: rgba(255,255,255,0.52);
//     border-radius: 4px;
//     padding-top: 0px;
//     padding-bottom: 0px;
//   }
//   margin-top: 2px;
// }
// .el-popper{
//   top: 45px !important;
//   padding-top: 50px !important;
//   border-radius: 0 0 4px 4px;
// }
// .el-popper[x-placement^=bottom] .popper__arrow::after,.popper__arrow{
//   display: none !important;
// }
</style>
