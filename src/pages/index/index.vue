<template>
  <view class="page">
    <view class="hero">
      <text class="hero__title">校园生活小助手</text>
      <text class="hero__desc">uni-app 空模板已就绪，开始写你的第一个页面</text>
    </view>

    <view class="card">
      <text class="card__label">当前运行环境</text>
      <text class="card__value">{{ platform }}</text>
    </view>

    <view class="tips">
      <text class="tips__item">· 新建页面：src/pages/ 下建目录，再到 pages.json 注册</text>
      <text class="tips__item">· 全局样式：src/App.vue</text>
      <text class="tips__item">· 主题变量：src/uni.scss</text>
    </view>
  </view>
</template>

<script setup>
import { ref } from 'vue'

const platform = ref(getPlatformText())

function getPlatformText() {
  // #ifdef MP-WEIXIN
  return '微信小程序'
  // #endif
  // #ifdef H5
  return 'H5'
  // #endif
  // #ifdef APP-PLUS
  return 'App'
  // #endif
  return '未知平台'
}
</script>

<style lang="scss" scoped>
.page {
  min-height: 100vh;
  padding: $space-lg;
  box-sizing: border-box;
}

.hero {
  display: flex;
  flex-direction: column;
  padding: $space-xl $space-lg;
  border-radius: $radius-lg;
  background: linear-gradient(135deg, $brand-color 0%, #6aa4ff 100%);

  &__title {
    font-size: $font-size-xl;
    font-weight: 600;
    color: #ffffff;
  }

  &__desc {
    margin-top: $space-sm;
    font-size: $font-size-base;
    color: rgba(255, 255, 255, 0.85);
  }
}

.card {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: $space-base;
  padding: $space-lg;
  border-radius: $radius-base;
  background-color: $color-bg-card;

  &__label {
    font-size: $font-size-base;
    color: $color-text-regular;
  }

  &__value {
    font-size: $font-size-base;
    font-weight: 600;
    color: $brand-color;
  }
}

.tips {
  display: flex;
  flex-direction: column;
  margin-top: $space-base;
  padding: $space-lg;
  border-radius: $radius-base;
  background-color: $color-bg-card;

  &__item {
    font-size: $font-size-sm;
    line-height: 2;
    color: $color-text-secondary;
  }
}
</style>
