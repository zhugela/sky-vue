<template>
  <section class="app-main">
    <transition
      name="fade-transform"
      mode="out-in"
    >
      <router-view />
    </transition>
  </section>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'

@Component({
  'name': 'AppMain'
})
export default class extends Vue {}
</script>

<style lang="scss" scoped>
.app-main {
  height: calc(100vh - 64px);
  overflow-y: auto;
  overflow-x: hidden;
  background: #F5F6FA;
  
  // 自定义滚动条 - 美团风格
  &::-webkit-scrollbar {
    width: 8px;
  }
  
  &::-webkit-scrollbar-track {
    background: rgba(0, 0, 0, 0.02);
    border-radius: 4px;
  }
  
  &::-webkit-scrollbar-thumb {
    background: rgba(255, 209, 0, 0.35);
    border-radius: 4px;
    
    &:hover {
      background: rgba(255, 209, 0, 0.55);
    }
  }
}
</style>
