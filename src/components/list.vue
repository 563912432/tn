<template>
  <div class="list">
    <v-header></v-header>
    <div class="content" v-loading.body="loading">
      <div class="menu">
        <router-link to="/" class='menu-title'><img src="../assets/back.png" alt="返回"> 返回
        </router-link>
        <span>{{ category }}</span>
        <router-link to="/" class='menu-title'><img src="../assets/home.png" alt="首页"> 首页</router-link>
      </div>
      <div class="wrapper">
        <el-row :gutter="10">
          <el-col :span="8" v-for="(value, index) in info" :key="value.id" class="w-col">
            <div class="w-content" @click="detail($event)">
              <el-card class="box-card" v-html="value.title" :id="value.id" :cid="value.cid">
              </el-card>
            </div>
          </el-col>
        </el-row>
      </div>
    </div>
    <v-footer></v-footer>
  </div>
</template>
<script>
  import footer from '@/components/footer'
  import header from '@/components/header'

  const Host = '/Api/Pinyin/'
  export default {
    name: 'list',
    components: {
      'v-footer': footer,
      'v-header': header
    },
    data () {
      return {
        category: '',
        cid: '',
        title: '',
        info: '',
        loading: false
      }
    },
    created () {
      let formdata = new FormData()
      formdata.append('cid', this.$route.params.cid)
      this.loading = true
      let url = Host + 'category'
      this.post(url, formdata, res => {
        this.loading = false
        if (res.status) {
          this.category = res.info['category']
          this.info = res.info['list']
        } else {
          console.log(res.info)
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
      detail (el) {
        let id = el.target.getAttribute('id')
        this.$router.push({path: '/d/' + id})
      }
    }
  }
</script>
<style scoped>
  @font-face {
    /* font-properties */
    font-family: kaiti;
    src:url('/static/font/simkai.ttf')
  }
  .list {
    flex: 1;
    display: flex;
    flex-direction: column;
  }

  .list .content {
    flex: 1;
    display: flex;
    flex-direction: column;
  }

  .list .content .menu {
    display: flex;
    height: 40px;
    align-items: center;
    justify-content: space-between;
    padding: 0 20px;
    font-size: 15px;
    color: #fff;
    background-color: #45a0e1;
  }

  .list .content .menu .menu-title {
    font-size: 13px;
    color: #fff;
    text-decoration: none;
  }
  .list .content .menu .menu-title img{
    width: 13px;
    height: 13px;
    margin-bottom: -1px;
  }

  .list .content .wrapper {
    font-family: 'kaiti';
    /*font-family: 'pinyin';*/
    flex: 1;
    overflow-y: auto;
    text-align: center;
    padding: 0 15px 10px 15px;
  }

  .list .content .wrapper .w-col {
    padding: 5px 0 0 0;
  }

  .list .content .wrapper .w-content {
    font-size: 22px;
    color: #45a0e2;
    font-weight: 600;
  }
  .list .content .wrapper .w-content .box-card{
    border-radius: 10px;
    border-color: #45a0e1;
    box-shadow: 1px 2px 2px #4095d1;
    color: #45a0e2;;
    margin-top: 10px;
    padding: 10px 0 8px 0;
  }
</style>
