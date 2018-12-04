<template>
  <div class="hello">
    <v-header :url="info.topVideo"></v-header>
    <div class="content">
      <ul v-loading.body="loading">
        <li v-for="(item, index) in msg"
            :class="[index % 2 === 0?'blue':'white']"
            @click="click($event)"
            :cid="item.id"
            :title="item.title">
          <i class="el-icon-arrow-right" style="font-weight: bold;color: #adadad"></i>　{{ item.title }}
        </li>
      </ul>
      <!--顶图广告位-->
      <div class="img-container" v-if="info.adInfo && info.adInfo.thumb">
        <img :src="info.adInfo.thumb?host + 'Uploads/' + info.adInfo.thumb:''" alt="" @click="topAdClick(info.adInfo.type, info.adInfo.video_url, info.adInfo.content, 'top')">
      </div>
      <!--推荐广告位-->
      <div class="tuijian-container" v-if="info.adInfo && bottomAd[0] && bottomAd[0].thumb1 !== ''">
        <div class="tuijian-parent" v-for="(item, index) in bottomAd" :key="index">
          <div class="tuijian-image">
            <img :src="item.thumb1?host + 'Uploads/' +item.thumb1:''" alt="" @click="tuijianClick(item.bottom_type, item.contentActivity, item.top_href_video, index)">
          </div>
          <div class="tuijian-title">

            {{item.adTitle}}
          </div>
        </div>
        <div style="clear: both"></div>
      </div>
    </div>
    <!--logo-->
    <div v-if="info.adInfo && openSimple && info.adInfo.logo" class="logo">
      <div class='b-button' @click="topAdClick(info.adInfo.type, info.adInfo.video_url, info.adInfo.content, 'logo')">
        <img :src="host + 'Uploads/' +info.adInfo.logo" alt="">
      </div>
      <div class="btnClose" @click="bClose">X</div>
    </div>
    <v-footer></v-footer>
  </div>
</template>

<script>
  import footer from '@/components/footer'
  import header from '@/components/header'
  import {mapState} from 'vuex'

  const Host = '/Api/Pinyin/'

  export default {
    name: 'HelloWorld',
    components: {
      'v-footer': footer,
      'v-header': header
    },
    computed: {
      timeNow () {
        return Date.parse(new Date())
      },
      ...mapState(['adCate', 'adType'])
    },
    data () {
      return {
        host: '/',
        cardId: '', // 卡片id
        uid: '', // 用户标识
        info: {
          topAdUrl: '',
          adInfo: {
            thumb: '',
            logo: '',
            content: '',
            ad: ''
          }
        }, // 全部信息
        msg: '',
        bottomAd: [],
        isParent: true,
        loading: false,
        openSimple: false
      }
    },
    created () {
      this.loading = true
      let url = Host + 'tuina'
      this.post(url, '', res => {
        this.loading = false
        if (res.status) {
          this.info = res.info
          this.cardId = this.info.card_id
          // 取localStorage
          let uid = window.localStorage.getItem('user')
          if (uid) {
            this.uid = uid
          } else {
            this.uid = this.info.uid
            window.localStorage.setItem('user', this.uid)
          }
          if (res.info.adInfo && res.info.adInfo.content) {
            this.$store.commit('saveTopContent', res.info.adInfo.content)
          }
          if (res.info.tree) {
            this.msg = res.info.tree
          }
          if (res.info.adInfo && res.info.adInfo.ad) {
            this.bottomAd = JSON.parse(res.info.adInfo.ad)
          }
        } else {
          console.log(res.info)
        }
      })
    },
    mounted () {
      // 取cookie
      this.openSimple = !this.$cookie.get('tuina')
    },
    methods: {
      // logo的点击事件
      topAdClick (type, hrefVideo, content, tag) {
        switch (parseInt(type)) {
          case 1: // 链接网址
            window.location.href = hrefVideo
            break
          case 2: // 视频代码
            this.$router.push(`/introduce/${hrefVideo}`)
            break
          case 3: // 活动详情
            this.$store.commit('saveTopContent', content)
            this.$router.push(`adDetail/${this.adCate.topAd}`)
            break
          default:
            break
        }
        let formdata = new FormData()
        formdata.append('cardId', this.cardId)
        formdata.append('uid', this.uid)
        if (tag === 'logo') { // logo统计
          formdata.append('tag', this.adType.logo)
        }
        if (tag === 'top') { // top统计
          formdata.append('tag', this.adType.top)
        }
        this.post(`${Host}adStatistic`, formdata, res => {
          console.log(res.info)
        })
      },
      tuijianClick (type, content, hrefVideo, index) {
        switch (parseInt(type)) {
          case 1: // 链接地址
            window.location.href = hrefVideo
            break
          case 2: // 活动详情
            this.$store.commit('saveTuiJianContent', content)
            this.$router.push(`adDetail/${this.adCate.tuijianAd}`)
            break
          default:
            break
        }
        let url = Host + 'adStatistic'
        let formdata = new FormData()
        formdata.append('cardId', this.cardId)
        formdata.append('uid', this.uid)
        let tag = 'bottom' + (index + 1)
        switch (tag) {
          case 'bottom1':
            tag = this.adType.bottom1
            break
          case 'bottom2':
            tag = this.adType.bottom2
            break
          case 'bottom3':
            tag = this.adType.bottom3
            break
          case 'bottom4':
            tag = this.adType.bottom4
            break
        }
        formdata.append('tag', tag)
        this.post(url, formdata, res => {
          console.log(res.info)
        })
      },
      timeTo13 (number10) {
        return parseInt(number10 + '000')
      },
      timeStringToStamp (string) {
        return (new Date(Date.parse(string.replace(/-/g, '/')))).getTime()
      },
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
      click (el) {
        let cid = el.target.getAttribute('cid')
        this.$router.push({path: '/list/' + cid})
      },
      bClose () {
        this.openSimple = false
        this.$cookie.set('tuina', 'Helloworld', parseInt(this.info.adInfo.interval))
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  li {
    list-style-type: none;
  }

  .hello {
    flex: 1;
    display: flex;
    flex-direction: column;
  }

  .hello .content {
    flex: 1;
    display: flex;
    flex-direction: column;
    overflow-y: auto;
    padding-top: 5px;
    z-index: 0;
  }

  .hello .content ul {
    margin: 0;
    padding: 0;
  }

  .hello .content ul li {
    padding: 10px;
  }

  .hello .content .white {
    background-color: #fff;
    margin: 3px;
    border-radius: 3px;
  }

  .hello .content .blue {
    background-color: #b9e2ff;
    margin: 3px;
    border-radius: 3px;
  }
</style>
