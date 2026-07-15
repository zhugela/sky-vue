<template>
  <div class="login">
    <!-- 装饰性几何图形 -->
    <div class="decorations">
      <div class="deco deco-1"></div>
      <div class="deco deco-2"></div>
      <div class="deco deco-3"></div>
      <div class="deco deco-4"></div>
      <div class="deco deco-5"></div>
    </div>
    
    <div class="login-box">
      <div class="login-left">
        <div class="welcome-content">
          <div class="brand-logo">
            <!-- 外卖袋图标 -->
            <svg width="80" height="80" viewBox="0 0 40 40" fill="none" xmlns="http://www.w3.org/2000/svg">
              <circle cx="20" cy="20" r="18" fill="rgba(255,255,255,0.95)"/>
              <path d="M12 16C12 14.8954 12.8954 14 14 14H26C27.1046 14 28 14.8954 28 16V28C28 29.1046 27.1046 30 26 30H14C12.8954 30 12 29.1046 12 28V16Z" fill="#2D3436"/>
              <path d="M15 14V12C15 9.23858 17.2386 7 20 7C22.7614 7 25 9.23858 25 12V14" stroke="#2D3436" stroke-width="2.5" stroke-linecap="round"/>
              <circle cx="20" cy="22" r="3.5" fill="#FFD100"/>
              <path d="M18 21.5L19.5 23L22.5 20" stroke="#2D3436" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
          </div>
          <h1 class="welcome-title">闪送达</h1>
          <p class="welcome-subtitle">美味即刻送达，让经营更简单</p>
          <div class="feature-list">
            <div class="feature-item">
              <span class="feature-icon">📊</span>
              <span>实时数据分析</span>
            </div>
            <div class="feature-item">
              <span class="feature-icon">🚀</span>
              <span>高效订单管理</span>
            </div>
            <div class="feature-item">
              <span class="feature-icon">💡</span>
              <span>智能营业决策</span>
            </div>
          </div>
        </div>
      </div>
      <div class="login-form">
        <el-form ref="loginForm" :model="loginForm" :rules="loginRules">
          <div class="login-form-title">
            <div class="logo-wrapper">
              <svg class="logo-icon" width="48" height="48" viewBox="0 0 40 40" fill="none" xmlns="http://www.w3.org/2000/svg">
                <circle cx="20" cy="20" r="18" fill="#FFD100"/>
                <path d="M12 16C12 14.8954 12.8954 14 14 14H26C27.1046 14 28 14.8954 28 16V28C28 29.1046 27.1046 30 26 30H14C12.8954 30 12 29.1046 12 28V16Z" fill="#2D3436"/>
                <path d="M15 14V12C15 9.23858 17.2386 7 20 7C22.7614 7 25 9.23858 25 12V14" stroke="#2D3436" stroke-width="2.5" stroke-linecap="round"/>
                <circle cx="20" cy="22" r="3" fill="#FFD100"/>
              </svg>
              <span class="title-label">闪送达</span>
            </div>
            <p class="login-desc">商家管理后台 · 欢迎回来</p>
          </div>
          <el-form-item prop="username">
            <el-input
              v-model="loginForm.username"
              type="text"
              auto-complete="off"
              placeholder="请输入账号"
              prefix-icon="iconfont icon-user"
            />
          </el-form-item>
          <el-form-item prop="password">
            <el-input
              v-model="loginForm.password"
              type="password"
              placeholder="请输入密码"
              prefix-icon="iconfont icon-lock"
              @keyup.enter.native="handleLogin"
            />
          </el-form-item>
          <el-form-item style="width: 100%">
            <el-button
              :loading="loading"
              class="login-btn"
              size="medium"
              type="primary"
              style="width: 100%"
              @click.native.prevent="handleLogin"
            >
              <span v-if="!loading">立即登录</span>
              <span v-else>登录中...</span>
            </el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from 'vue-property-decorator'
import { Route } from 'vue-router'
import { Form as ElForm, Input } from 'element-ui'
import { UserModule } from '@/store/modules/user'
import { isValidUsername } from '@/utils/validate'

@Component({
  name: 'Login',
})
export default class extends Vue {
  private validateUsername = (rule: any, value: string, callback: Function) => {
    if (!value) {
      callback(new Error('请输入用户名'))
    } else {
      callback()
    }
  }
  private validatePassword = (rule: any, value: string, callback: Function) => {
    if (value.length < 6) {
      callback(new Error('密码必须在6位以上'))
    } else {
      callback()
    }
  }
  private loginForm = {
    username: 'admin',
    password: '123456',
  } as {
    username: String
    password: String
  }

  loginRules = {
    username: [{ validator: this.validateUsername, trigger: 'blur' }],
    password: [{ validator: this.validatePassword, trigger: 'blur' }],
  }
  private loading = false
  private redirect?: string

  @Watch('$route', { immediate: true })
  private onRouteChange(route: Route) {}

  // 登录
  private handleLogin() {
    ;(this.$refs.loginForm as ElForm).validate(async (valid: boolean) => {
      if (valid) {
        this.loading = true
        await UserModule.Login(this.loginForm as any)
          .then((res: any) => {
            if (String(res.code) === '1') {
              this.$router.push('/')
            } else {
              // this.$message.error(res.msg)
              this.loading = false
            }
          })
          .catch(() => {
            // this.$message.error('用户名或密码错误！')
            this.loading = false
          })
      } else {
        return false
      }
    })
  }
}
</script>

<style lang="scss">
@import '@/styles/_variables.scss';

.login {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  min-height: 100vh;
  background: linear-gradient(135deg, #FFE259 0%, #FFC107 50%, #FF9500 100%);
  position: relative;
  overflow: hidden;
}

// 装饰性几何图形 - 美团风格
.decorations {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  pointer-events: none;
  z-index: 0;
}

.deco {
  position: absolute;
  opacity: 0.2;
  animation: float 6s ease-in-out infinite;
}

.deco-1 {
  width: 120px;
  height: 120px;
  border: 4px solid #fff;
  border-radius: 50%;
  top: 10%;
  left: 8%;
  animation-delay: 0s;
}

.deco-2 {
  width: 60px;
  height: 60px;
  background: rgba(255, 255, 255, 0.4);
  border-radius: 50%;
  top: 20%;
  right: 15%;
  animation-delay: 1s;
}

.deco-3 {
  width: 80px;
  height: 80px;
  border: 4px solid #fff;
  border-radius: 16px;
  top: 60%;
  left: 5%;
  transform: rotate(15deg);
  animation-delay: 2s;
}

.deco-4 {
  width: 50px;
  height: 50px;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  bottom: 15%;
  right: 10%;
  animation-delay: 3s;
}

.deco-5 {
  width: 100px;
  height: 100px;
  border: 3px dashed rgba(255, 255, 255, 0.35);
  border-radius: 50%;
  bottom: 25%;
  left: 15%;
  animation-delay: 4s;
}

@keyframes float {
  0%, 100% { transform: translateY(0) rotate(0deg); }
  50% { transform: translateY(-15px) rotate(3deg); }
}

.login-box {
  width: 1000px;
  min-height: 540px;
  border-radius: 24px;
  display: flex;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.15);
  overflow: hidden;
  position: relative;
  z-index: 1;
  animation: slideUp 0.5s ease-out;
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.login-left {
  width: 55%;
  background: linear-gradient(135deg, #FFD100 0%, #FFB800 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 50px;
  position: relative;
  overflow: hidden;

  &::before {
    content: '';
    position: absolute;
    width: 300px;
    height: 300px;
    background: rgba(255, 255, 255, 0.15);
    border-radius: 50%;
    top: -100px;
    right: -100px;
  }

  &::after {
    content: '';
    position: absolute;
    width: 200px;
    height: 200px;
    background: rgba(255, 255, 255, 0.1);
    border-radius: 50%;
    bottom: -50px;
    left: -50px;
  }
}

.welcome-content {
  color: #2D3436;
  position: relative;
  z-index: 1;
  text-align: center;
}

.brand-logo {
  margin-bottom: 24px;
  
  svg {
    filter: drop-shadow(0 4px 12px rgba(0, 0, 0, 0.1));
  }
}

.welcome-title {
  font-size: 48px;
  font-weight: 800;
  margin-bottom: 16px;
  letter-spacing: 4px;
  color: #2D3436;
  font-family: 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', sans-serif;
}

.welcome-subtitle {
  font-size: 18px;
  color: rgba(45, 52, 54, 0.8);
  margin-bottom: 40px;
  font-weight: 500;
}

.feature-list {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.feature-item {
  display: flex;
  align-items: center;
  gap: 14px;
  font-size: 15px;
  padding: 14px 24px;
  background: rgba(255, 255, 255, 0.5);
  border-radius: 16px;
  color: #2D3436;
  font-weight: 500;
  transition: all 0.25s ease;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);

  &:hover {
    background: rgba(255, 255, 255, 0.8);
    transform: translateX(6px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  }
}

.feature-icon {
  font-size: 26px;
}

.login-form {
  background: #ffffff;
  width: 45%;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 50px 40px;

  .el-form {
    width: 100%;
    max-width: 320px;
  }

  .el-form-item {
    margin-bottom: 24px;
  }

  .el-form-item.is-error .el-input__inner {
    border-color: #FF5252 !important;
    background: #fff !important;
  }

  .el-input__inner {
    border: 2px solid #DFE6E9;
    border-radius: 25px;
    font-size: 15px;
    font-weight: 500;
    color: #2D3436;
    height: 50px;
    line-height: 50px;
    padding-left: 50px;
    transition: all 0.25s ease;

    &:focus {
      border-color: #FFD100;
      box-shadow: 0 0 0 4px rgba(255, 209, 0, 0.15);
    }
  }

  .el-input__prefix {
    left: 18px;
    color: #B2BEC3;
    font-size: 18px;
  }

  .el-input--prefix .el-input__inner {
    padding-left: 50px;
  }

  .el-input__inner::placeholder {
    color: #B2BEC3;
    font-size: 14px;
  }

  .el-form-item--medium .el-form-item__content {
    line-height: 50px;
  }

  .el-input--medium .el-input__icon {
    line-height: 50px;
  }
}

.login-btn {
  border-radius: 25px;
  padding: 0 !important;
  height: 50px;
  margin-top: 10px;
  font-weight: 700;
  font-size: 16px;
  border: 0;
  color: #2D3436;
  background: #FFD100;
  box-shadow: 0 6px 20px rgba(255, 209, 0, 0.4);
  transition: all 0.25s ease;
  letter-spacing: 3px;

  &:hover,
  &:focus {
    background: #FFE066;
    box-shadow: 0 8px 28px rgba(255, 209, 0, 0.5);
    transform: translateY(-2px);
    color: #2D3436;
  }

  &:active {
    transform: translateY(0);
  }
}

.login-form-title {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 36px;
  text-align: center;

  .logo-wrapper {
    display: flex;
    align-items: center;
    gap: 12px;
    margin-bottom: 12px;
  }

  .logo-icon {
    filter: drop-shadow(0 2px 6px rgba(0, 0, 0, 0.1));
  }

  .title-label {
    font-weight: 800;
    font-size: 28px;
    color: #2D3436;
    letter-spacing: 2px;
  }

  .login-desc {
    font-size: 14px;
    color: #636E72;
    margin-top: 8px;
    font-weight: 500;
  }
}

// 响应式设计
@media (max-width: 1024px) {
  .login-box {
    width: 90%;
    max-width: 500px;
    flex-direction: column;
  }

  .login-left {
    display: none;
  }

  .login-form {
    width: 100%;
    padding: 60px 40px;
  }
}
</style>
