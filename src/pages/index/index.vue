<template>
  <div class="content">
    <input placeholder="sdkAppID" v-model="sdkAppID" />
    <input placeholder="userID" v-model="userID" />
    <input placeholder="userSig" v-model="userSig" maxlength="-1" />
    <input placeholder="roomID" v-model="roomID" />
    <div class="btn" @click="gotoTrtc">进入多人会议视频</div>
  </div>
</template>

<script>

export default {
  data () {
    return {
      sdkAppID: '',
      userID: '',
      userSig: '',
      roomID: ''
    }
  },

  methods: {
    gotoTrtc () {
      if (!this.sdkAppID) {
        this.showToast('输入sdkAppID')
        return false
      }
      if (!this.userID) {
        this.showToast('输入userID')
        return false
      }
      if (!this.userSig) {
        this.showToast('输入userSig')
        return false
      }
      if (!this.roomID) {
        this.showToast('输入roomID')
        return false
      }
      wx.navigateTo({ url: `/pages/chatRoom/main?sdkAppID=${this.sdkAppID}&userID=${this.userID}&userSig=${this.userSig}&roomID=${this.roomID}` })
    },
    showToast (title) {
      wx.showToast({
        title: title, // 提示的内容,
        icon: 'none', // 图标,
        duration: 2000, // 延迟时间,
        mask: true // 显示透明蒙层，防止触摸穿透
      })
    }
  },
  created () {
    // let app = getApp()
  }
}
</script>

<style scoped>
.content {
  padding: 30px 10px 0 10px;
}
input {
  border:1px solid #ccc;
  height: 40px;
  margin: 10px 0;
}
.btn{
  margin: 20px;
  height: 40px;
  border: 1px solid #000;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 14px;
  color: #000;
}
</style>
