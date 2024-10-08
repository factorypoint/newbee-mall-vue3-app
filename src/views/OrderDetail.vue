<!--
 * 严肃声明：
 * 开源版本请务必保留此注释头信息，若删除我方将保留所有法律责任追究！
 * 本系统已申请软件著作权，受国家版权局知识产权以及国家计算机软件著作权保护！
 * 可正常分享和学习源码，不得用于违法犯罪活动，违者必究！
 * Copyright (c) 2020 陈尼克 all rights reserved.
 * 版权所有，侵权必究！
 *
-->

<template>
  <div class="order-detail-box">
    <s-header :name="'Order details'" @callback="close"></s-header>
    <div class="order-status">
      <div class="status-item">
        <label>Order status: </label>
        <span>{{ state.detail.orderStatusString }}</span>
      </div>
      <div class="status-item">
        <label>Order number: </label>
        <span>{{ state.detail.orderNo }}</span>
      </div>
      <div class="status-item">
        <label>Hour of order: </label>
        <span>{{ state.detail.createTime }}</span>
      </div>
      <van-button v-if="state.detail.orderStatus == 3" style="margin-bottom: 10px" color="#1baeae" block @click="handleConfirmOrder(state.detail.orderNo)">Confirm order</van-button>
      <van-button v-if="state.detail.orderStatus == 0" style="margin-bottom: 10px" color="#1baeae" block @click="showPayFn">Go to pay</van-button>
      <van-button v-if="!(state.detail.orderStatus < 0 || state.detail.orderStatus == 4)" block @click="handleCancelOrder(state.detail.orderNo)">Cancel order</van-button>
    </div>
    <div class="order-price">
      <div class="price-item">
        <label>Item price: </label>
        <span>{{ state.detail.totalPrice }}€</span>
      </div>
      <div class="price-item">
        <label>Delivery method: </label>
        <span>Standard express delivery</span>
      </div>
    </div>
    <van-card
      v-for="item in state.detail.newBeeMallOrderItemVOS"
      :key="item.goodsId"
      style="background: #fff"
      :num="item.goodsCount"
      :price="item.sellingPrice"
      desc="Delivery included"
      :title="item.goodsName"
      :thumb="$filters.prefix(item.goodsCoverImg)"
    />
    <van-popup
      v-model:show="state.showPay"
      position="bottom"
      :style="{ height: '24%' }"
    >
      <div :style="{ width: '90%', margin: '0 auto', padding: '20px 0' }">
        <van-button :style="{ marginBottom: '10px' }" color="#1989fa" block @click="handlePayOrder(state.detail.orderNo, 1)">AliPay</van-button>
        <van-button color="#4fc08d" block @click="handlePayOrder(state.detail.orderNo, 2)">WeChatPay</van-button>
      </div>
    </van-popup>
  </div>
</template>

<script setup>
import { reactive, toRefs, onMounted } from 'vue'
import sHeader from '@/components/SimpleHeader.vue'
import { getOrderDetail, cancelOrder, confirmOrder, payOrder } from '@/service/order'
import { showConfirmDialog, showLoadingToast, closeToast, showSuccessToast, closeDialog } from 'vant'
import { useRoute } from 'vue-router'
const route = useRoute()
const state = reactive({
  detail: {},
  showPay: false
})

onMounted(() => {
  init()
})

const init = async () => {
  showLoadingToast({
    message: 'Loading...',
    forbidClick: true
  });
  const { id } = route.query
  const { data } = await getOrderDetail(id)
  state.detail = data
  closeToast()
}

const handleCancelOrder = (id) => {
  showConfirmDialog({
    title: 'Do you want to cancel the order?',
  }).then(() => {
    cancelOrder(id).then(res => {
      if (res.resultCode == 200) {
        showSuccessToast('Deletion successful')
        init()
      }
    })
  }).catch(() => {
    // on cancel
  });
}

const handleConfirmOrder = (id) => {
  showConfirmDialog({
    title: 'Confirm order?',
  }).then(() => {
    confirmOrder(id).then(res => {
      if (res.resultCode == 200) {
        showSuccessToast('Confirmation succesful')
        init()
      }
    })
  }).catch(() => {
    // on cancel
  });
}

const showPayFn = () => {
  state.showPay = true
}

const handlePayOrder = async (id, type) => {
  await payOrder({ orderNo: id, payType: type })
  state.showPay = false
  init()
}

const close = () => {
  closeDialog
}
</script>

<style lang="less" scoped>
  .order-detail-box {
    background: #f7f7f7;
    .order-status {
      background: #fff;
      padding: 20px;
      font-size: 15px;
      .status-item {
        margin-bottom: 10px;
        label {
          color: #999;
        }
        span {

        }
      }
    }
    .order-price {
      background: #fff;
      margin: 20px 0;
      padding: 20px;
      font-size: 15px;
      .price-item {
        margin-bottom: 10px;
        label {
          color: #999;
        }
        span {

        }
      }
    }
    .van-card {
      margin-top: 0;
    }
    .van-card__content {
      display: flex;
      flex-direction: column;
      justify-content: center;
    }
  }
</style>
