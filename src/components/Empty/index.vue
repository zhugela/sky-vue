<!--  -->
<template>
  <div class="empty-box">
    <div class="img-box">
      <img v-if="!isSearch"
           src="../../assets/table_empty.png"
           alt="">
      <img v-else
           src="../../assets/search_table_empty.png">
      <p>{{ !isSearch ? '这里空空如也~' : 'Sorry，木有找到您搜索的内容哦~' }}</p>
    </div>
  </div>
</template>

<script lang='ts'>
import { Vue, Component, Prop } from 'vue-property-decorator'
@Component({
  name: 'Empty'
})
export default class extends Vue {
  @Prop({ default: false }) isSearch: boolean //用来区分是搜索还是默认无数据
}
</script>
<style scoped lang="scss">
.empty-box {
  text-align: center;
  padding: 80px 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 400px;
  
  .img-box {
    animation: fadeIn 0.5s ease-out;
  }
  
  img {
    margin: 0 auto;
    width: 200px;
    height: auto;
    margin-bottom: 24px;
    opacity: 0.8;
    filter: drop-shadow(0 4px 12px rgba(0, 0, 0, 0.08));
  }
  
  p {
    color: #6B7280;
    font-size: 15px;
    font-weight: 500;
    background: linear-gradient(135deg, #9CA3AF 0%, #6B7280 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
