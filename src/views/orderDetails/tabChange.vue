<!--  -->
<template>
  <div class="tab-change">
    <div v-for="item in changedOrderList"
         :key="item.value"
         class="tab-item"
         :class="{ active: item.value === activeIndex }"
         @click="tabChange(item.value)">
      <el-badge :class="{'special-item':item.num<10}"
                class="item"
                :value="item.num > 99 ? '99+' : item.num"
                :hidden="!([2, 3, 4].includes(item.value) && item.num)">
        {{ item.label }}
      </el-badge>
    </div>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Prop, Watch } from 'vue-property-decorator'
import { getOrderDetailPage } from '@/api/order'

@Component({
  name: 'TabChange'
})
export default class extends Vue {
  @Prop({ default: '' }) orderStatics: any
  @Prop({ default: '' }) defaultActivity: any
  private activeIndex: number = this.defaultActivity || 0

  @Watch('defaultActivity')
  private onChange(val) {
    this.activeIndex = Number(val)
  }

  get changedOrderList() {
    return [
      {
        label: '全部订单',
        value: 0
      },
      {
        label: '待接单',
        value: 2,
        num: this.orderStatics.toBeConfirmed
      },
      {
        label: '待派送',
        value: 3,
        num: this.orderStatics.confirmed
      },
      {
        label: '派送中',
        value: 4,
        num: this.orderStatics.deliveryInProgress
      },
      {
        label: '已完成',
        value: 5
      },
      {
        label: '已取消',
        value: 6
      }
    ]
  }

  private tabChange(activeIndex) {
    this.activeIndex = activeIndex
    this.$emit('tabChange', activeIndex)
  }
}
</script>
<style lang="scss">
/* 美团外卖风格Tab */
.tab-change {
  display: inline-flex;
  border-radius: 28px;
  margin-bottom: 24px;
  background: #ffffff;
  padding: 5px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.06);
  border: 1px solid #DFE6E9;

  .tab-item {
    min-width: 90px;
    padding: 0 18px;
    height: 40px;
    text-align: center;
    line-height: 40px;
    color: #636E72;
    background: transparent;
    border: none;
    border-radius: 20px;
    cursor: pointer;
    font-size: 14px;
    font-weight: 600;
    transition: all 0.25s ease;
    margin: 0 2px;
    position: relative;
    
    &:hover:not(.active) {
      background: #FFF9E6;
      color: #2D3436;
    }
    
    .special-item {
      .el-badge__content {
        width: 20px;
        padding: 0 5px;
      }
    }
    
    .item {
      .el-badge__content {
        background: #FF5252 !important;
        line-height: 18px;
        height: auto;
        min-width: 20px;
        min-height: 20px;
        border: 2px solid #fff;
        box-shadow: 0 2px 8px rgba(255, 82, 82, 0.4);
        font-weight: 700;
      }
      
      .el-badge__content.is-fixed {
        top: 6px;
        right: -4px;
      }
    }
  }
  
  .active {
    background: #FFD100 !important;
    color: #2D3436 !important;
    font-weight: 700;
    box-shadow: 0 3px 10px rgba(255, 209, 0, 0.4);
    
    .item .el-badge__content {
      background: #2D3436 !important;
      color: #FFD100 !important;
      border-color: transparent;
    }
  }
  
  .tab-item:first-child {
    border-left: none;
  }
}
</style>
