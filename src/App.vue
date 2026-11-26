<script>
export default {
  onLaunch() {
    console.log('[App] Launch')
  },
  onShow() {
    console.log('[App] Show')
  },
  onHide() {
    console.log('[App] Hide')
  }
}
</script>

<style>
/* 全局样式写在这里，会作用于所有页面 */
page {
  background-color: #f5f6f8;
  color: #1f2329;
  font-size: 28rpx;
  font-family: -apple-system, BlinkMacSystemFont, 'Helvetica Neue', 'PingFang SC',
    'Microsoft YaHei', sans-serif;
}
</style>
