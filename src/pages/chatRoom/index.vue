<template>
  <div class="page-room">
    <trtc-room id="trtc-component" :config="rtcConfig"></trtc-room>
  </div>
</template>

<script>

export default {
  data () {
    return {
      trtcComponent: null,
      sdkAppID: '',
      userID: '',
      userSig: '',
      roomID: '',
      rtcConfig: {}
    }
  },
  onLoad (e) {
    this.sdkAppID = e.sdkAppID
    this.userID = e.userID
    this.userSig = e.userSig
    this.roomID = e.roomID
    this.rtcConfig = {
      sdkAppID: e.sdkAppID, // 您的实时音视频服务创建应用后分配的 sdkAppID
      userID: e.userID,
      userSig: e.userSig,
      template: 'grid', // 1v1 grid custom
      frontCamera: 'front',
      enableEarMonitor: false,
      enableAutoFocus: true,
      localMirror: true,
      enableAgc: true,
      enableAns: true,
      encsmall: false,
      videoWidth: 360,
      videoHeight: 640,
      scene: 'rtc',
      maxBitrate: 900,
      minBitrate: 600,
      beautyLevel: 0, // 默认开启美颜
      enableIM: 'false', // 可选，仅支持初始化设置，不支持动态修改
      debugMode: 'false'
    }
  },
  onReady () {
    this.trtcComponent = this.$mp.page.selectComponent('#trtc-component') // 获取 rtcroom 实例
    this.bindTRTCRoomEvent() // 监听TRTC Room 关键事件
    this.joinRoom() // 进入视频房间
  },
  methods: {
    joinRoom () {
      this.trtcComponent.enterRoom({ roomID: this.roomID }).then(() => {
        this.trtcComponent.publishLocalVideo()
        this.trtcComponent.publishLocalAudio()
      }).catch((res) => {
        console.error('* room joinRoom 进房失败:', res)
      })
    },
    bindTRTCRoomEvent () {
      const TRTC_EVENT = this.trtcComponent.EVENT
      this.timestamp = []
      // 初始化事件订阅
      this.trtcComponent.on(TRTC_EVENT.LOCAL_JOIN, (event) => {
        console.log('* room LOCAL_JOIN', event)
        // 进房成功，触发该事件后可以对本地视频和音频进行设置
        if (this.options.localVideo === true || this.options.template === '1v1') {
          this.trtcComponent.publishLocalVideo()
        }
        if (this.options.localAudio === true || this.options.template === '1v1') {
          this.trtcComponent.publishLocalAudio()
        }
      })
      this.trtcComponent.on(TRTC_EVENT.LOCAL_LEAVE, (event) => {
        console.log('* room LOCAL_LEAVE', event)
      })
      this.trtcComponent.on(TRTC_EVENT.ERROR, (event) => {
        console.log('* room ERROR', event)
      })
      // 远端用户进房
      this.trtcComponent.on(TRTC_EVENT.REMOTE_USER_JOIN, (event) => {
        console.log('* room REMOTE_USER_JOIN', event, this.trtcComponent.getRemoteUserList())
        this.timestamp.push(new Date())
        // 1v1视频通话时限制人数为两人的简易逻辑，建议通过后端实现房间人数管理
        // 2人以上同时进行通话请选择网格布局
        if (this.template === '1v1' && this.timestamp.length > 1) {
          const interval = this.timestamp[1] - this.timestamp[0]
          if (interval < 1000) {
            // 房间里已经有两个人
            this.setData({
              showTipToast: true
            }, () => {
              setTimeout(() => {
                this.setData({
                  showTipToast: false
                })
                wx.navigateBack({
                  delta: 1
                })
              }, 4000)
            })
          }
        }
      })
      // 远端用户退出
      this.trtcComponent.on(TRTC_EVENT.REMOTE_USER_LEAVE, (event) => {
        console.log('* room REMOTE_USER_LEAVE', event, this.trtcComponent.getRemoteUserList())
        if (this.template === '1v1') {
          this.timestamp = []
        }
        if (this.template === '1v1' && this.remoteUser === event.data.userID) {
          this.remoteUser = null
        }
      })
      // 远端用户推送视频
      this.trtcComponent.on(TRTC_EVENT.REMOTE_VIDEO_ADD, (event) => {
        console.log('* room REMOTE_VIDEO_ADD', event, this.trtcComponent.getRemoteUserList())
        // 订阅视频
        const userList = this.trtcComponent.getRemoteUserList()
        const data = event.data
        if (this.template === '1v1' && (!this.remoteUser || this.remoteUser === data.userID)) {
          // 1v1 只订阅第一个远端流
          this.remoteUser = data.userID
          this.trtcComponent.subscribeRemoteVideo({
            userID: data.userID,
            streamType: data.streamType
          })
        } else if (this.template === 'grid') {
          this.trtcComponent.subscribeRemoteVideo({
            userID: data.userID,
            streamType: data.streamType
          })
        }
        if (this.template === 'custom' && data.userID && data.streamType) {
          let index = userList.findIndex((item) => {
            return item.userID === data.userID
          })
          index++
          const y = 320 * index + 160
          // 设置远端视图坐标和尺寸
          this.trtcComponent.setViewRect({
            userID: data.userID,
            streamType: data.streamType,
            xAxis: '480rpx',
            yAxis: y + 'rpx',
            width: '240rpx',
            height: '320rpx'
          })
        }
      })
      // 远端用户取消推送视频
      this.trtcComponent.on(TRTC_EVENT.REMOTE_VIDEO_REMOVE, (event) => {
        console.log('* room REMOTE_VIDEO_REMOVE', event, this.trtcComponent.getRemoteUserList())
      })
      // 远端用户推送音频
      this.trtcComponent.on(TRTC_EVENT.REMOTE_AUDIO_ADD, (event) => {
        console.log('* room REMOTE_AUDIO_ADD', event, this.trtcComponent.getRemoteUserList())
        // 订阅音频
        const data = event.data
        if (this.template === '1v1' && (!this.remoteUser || this.remoteUser === data.userID)) {
          this.remoteUser = data.userID
          this.trtcComponent.subscribeRemoteAudio({ userID: data.userID })
        } else if (this.template === 'grid' || this.template === 'custom') {
          this.trtcComponent.subscribeRemoteAudio({ userID: data.userID })
        }
        // 如果不订阅就不会自动播放音频
        // this.trtcComponent.subscribeRemoteAudio({ userID: data.userID })
      })
      // 远端用户取消推送音频
      this.trtcComponent.on(TRTC_EVENT.REMOTE_AUDIO_REMOVE, (event) => {
        console.log('* room REMOTE_AUDIO_REMOVE', event, this.trtcComponent.getRemoteUserList())
      })
      this.trtcComponent.on(TRTC_EVENT.IM_MESSAGE_RECEIVED, (event) => {
        console.log('* room IM_MESSAGE_RECEIVED', event)
      })
    }
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
