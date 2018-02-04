<template>
  <mdc-layout-app>

    <mdc-toolbar slot="toolbar">
      <!--  toolbar markup here -->
    </mdc-toolbar>

    <mdc-drawer slot="drawer" class="drawer">
      <div class="header">
      <img src="@/assets/logo.png" height="192" width="192">
        <span v-text="blogName"></span>
        <mdc-button v-if="token" @click="logout">注销</mdc-button>
      </div>
      <mdc-drawer-list permanent>
        <mdc-drawer-item active to="/">All</mdc-drawer-item>
        <mdc-drawer-item v-for="tag in tags" :to="/tag/+tag">{{ tag }}</mdc-drawer-item>
      </mdc-drawer-list>
    </mdc-drawer>

    <main>
        <mdc-card class="card" v-for="article in articles">
          <mdc-card-header :title="article.Name" :subtitle="article.CreateTime">
            <mdc-card-action-button  @click="catArticle(article)">{{ article.Name }}</mdc-card-action-button>
            <br>
            <span v-text="'创建于 ' + unixToTime(article.CreateTime,'yyyy-MM-dd')"></span>
          </mdc-card-header>
          <mdc-card-text v-html="article.Desc"></mdc-card-text>
          <mdc-card-actions>
            <mdc-card-action-button @click="$router.push({path:'/tag/'+article.Tag })">{{ article.Tag }}</mdc-card-action-button>
          </mdc-card-actions>
        </mdc-card>
      <mdc-fab v-if="token" icon="add" class="fab-add" @click="$router.push({path:'/add'})"></mdc-fab>
    </main>

  </mdc-layout-app>
</template>

<script>
  export default {
    name: 'Home',
    data() {
      return {
        tags: [],
        articles: [],
        blogName :'',
        token : window.localStorage.getItem('token')
      }
    },
    created() {
      this.$http.get('/api/tags')
        .then((response) => {
          this.tags = response.data.tags
        })

      this.getBlogName()

      this.getArticle()
    },
    watch:{
      '$route':'getArticle'
    },
    methods: {
      getArticle() {
        var url
        if (this.$route.params.tag === undefined) {
          url = '/api/tag'
        }else {
          url = '/api/tag/' + this.$route.params.tag
        }
        this.$http.get(url)
          .then((response) => {
            this.articles = response.data.articles
            for (let i=0;i<this.articles.length;i++){
              var url = '/api/article/uuid/'+this.articles[i].Uuid
              this.$http.get(url,{
                params:{
                  'mode':'description'
                }
              }).then((response)=>{
                this.$set(this.articles[i],'Desc',response.data)
              })
            }
          })
      },
      getBlogName(){
        this.$http.get('/api/name')
          .then((response)=>{
            this.blogName = response.data
          })
      },
      catArticle(article){
        window.localStorage.setItem('article',JSON.stringify(article))
        this.$router.push({path:'/article/'+article.Id})
      },
      unixToTime(value,fmt){
        let date = new Date(value);
        if (/(y+)/.test(fmt)) {
          fmt = fmt.replace(RegExp.$1, (date.getFullYear() + '').substr(4 - RegExp.$1.length));
        }
        let o = {
          'M+': date.getMonth() + 1,
          'd+': date.getDate(),
          'h+': date.getHours(),
          'm+': date.getMinutes(),
          's+': date.getSeconds()
        };
        for (let k in o) {
          if (new RegExp(`(${k})`).test(fmt)) {
            let str = o[k] + '';
            fmt = fmt.replace(RegExp.$1, (RegExp.$1.length === 1) ? str : ('00' + str).substr(str.length));
          }
        }
        return fmt;
      },
      logout(){
        window.localStorage.setItem('token','')
        this.token=''
      }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

  .card{
    margin: 40px;
  }
  .header{
    height: 239px;
    width: 239px;
    background-color: cornflowerblue;
  }

.drawer {
  text-align: center;
}
.fab-add {
  position: absolute;
  right: 40px;
  bottom: 40px;
  z-index: 99999999;
}
</style>
