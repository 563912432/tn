<template>
  <div class="detail">
    <div class="back" @click="close"><i class="el-icon-error" style="color: #ffffff;font-size: 30px"></i></div>
    <div v-show="show" id="showVideo"></div>
    <div v-show="!show" class="img">
      <img :src="host + 'Uploads/'+ img" alt="" @click="zoomIn" id="img">
    </div>
  </div>
</template>
<script>
  import polyvObject from 'polyvObject'

  export default {
    name: 'detail',
    data () {
      return {
        host: '/',
        player: null,
        vid: null,
        cid: '',
        img: null,
        show: true,
        zoom: false
      }
    },
    mounted () {
      this.loading = true
      let formdata = new FormData()
      formdata.append('id', this.$route.params.id)
      let url = this.host + 'Api/Pinyin/cardDetail'
      this.post(url, formdata, res => {
        this.loading = false
        if (res.status) {
          this.vid = res.info.video
          this.cid = res.info.cid
          this.img = res.info.thumb
          if (this.vid) {
            this.show = true
            let videoHeight = 200
            let height = window.screen.height
            let viewHeight = window.innerHeight
            let titleHeight = height - viewHeight
            let marginTop = (height - videoHeight) / 2 - titleHeight
            document.getElementById('showVideo').style.marginTop = marginTop + 'px'
            this.player = polyvObject('#showVideo').videoPlayer({
              'width': '100%',
              'height': videoHeight,
              'vid': this.vid
            })
            this.player.j2s_resumeVideo()
          } else if (this.img) {
            this.show = false
          }
        }
      })
    },
    methods: {
      post (url, data, fn) {         // datat应为'a=a1&b=b1'这种字符串格式，在jq里如果data为对象会自动将对象转成这种字符串格式
        let obj = new XMLHttpRequest()
        obj.open('POST', url, true)
        obj.onreadystatechange = function () {
          if (+obj.readyState === 4 && (+obj.status === 200 || +obj.status === 304 || +obj.status === 0)) {  // 304未修改
            fn.call(this, JSON.parse(obj.responseText))
          }
        }
        obj.send(data)
      },
      close () {
        // 关闭视频，关闭页面
        if (this.vid) {
          this.player.j2s_pauseVideo()
          this.player = null
        }
        this.$router.push({path: '/list/' + this.cid})
      },
      zoomIn (e) {
        if (this.zoom) {
          e.target.style.width = '100%'
          e.target.style.marginTop = '60px'
          this.zoom = false
        } else {
          e.target.style.width = '200%'
          e.target.style.marginTop = '0'
          this.zoom = true
        }
      }
    }
  }
</script>
<style scoped>
  .detail {
    flex: 1;
    background-color: #212121;
    display: flex;
    z-index: 0;
  }
  .detail .back{
    background-color: #212121;
    border-radius: 20px;
    position: absolute;
    left: 10px;
    top: 10px;
    padding: 1px;
    z-index: 1;
  }
  .detail #showVideo {
    width: 100%;
    height: auto;
    z-index: 11;
  }
  .detail .img{
    position: relative;
    width: 100%;
    height: 90%;
  }
  .detail .img img{
    margin-top: 60px;
    width: 100%;
    height: auto;
  }
</style>
