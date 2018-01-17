<template>
  <div class="hello">
    <v-header></v-header>
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
    </div>
    <v-footer></v-footer>
  </div>
</template>

<script>
  import footer from '@/components/footer'
  import header from '@/components/header'

  const Host = '/Api/Pinyin/'

  export default {
    name: 'HelloWorld',
    components: {
      'v-footer': footer,
      'v-header': header
    },
    data () {
      return {
        msg: '',
        isParent: true,
        loading: false
      }
    },
    created () {
      this.loading = true
      let url = Host + 'tuina'
      this.post(url, '', res => {
        this.loading = false
        if (res.status) {
          this.msg = res.info
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
      click (el) {
        let cid = el.target.getAttribute('cid')
        this.$router.push({path: '/list/' + cid})
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
