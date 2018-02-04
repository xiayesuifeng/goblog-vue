<template>
  <mdc-layout-app>

    <mdc-toolbar slot="toolbar">
      <!--  toolbar markup here -->
    </mdc-toolbar>

    <main>
      <mdc-fab style="margin: 10px" icon="arrow_back" to="/"></mdc-fab>
      <mdc-card class="card">
        <mdc-card-media src="/assets/welcome_card.jpg" :height="150" dark>
          <mdc-card-title v-text="article.Name"></mdc-card-title>
        </mdc-card-media>
        <div>
          <span v-text="'创建于'+ unixToTime(article.CreateTime,'yyyy-MM-dd')"></span>
          <span v-text="'最后编辑于'+ unixToTime(article.EditTime,'yyyy-MM-dd')"></span>
          <span style="margin-right: 0; margin-left: 60%" v-text="article.Tag"></span>
        </div>
        <mdc-card-text v-html="context"></mdc-card-text>
      </mdc-card>
    </main>

  </mdc-layout-app>
</template>

<script>
    export default {
        name: "Article",
      data(){
          return {
            article:{},
            context:''
          }
      },
      created(){
        this.article = JSON.parse(window.localStorage.getItem('article'))

        this.$http.get('/api/article/uuid/'+this.article.Uuid)
          .then(r=>{
            this.context = r.data
          })
      },
      methods: {
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
        }
      }
    }
</script>

<style scoped>
  .card{
    margin-right: 10%;
    margin-left: 10%;
    margin-top: 200px;
  }
</style>
