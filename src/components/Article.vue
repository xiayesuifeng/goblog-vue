<template>
  <mdc-layout-app>

    <mdc-toolbar slot="toolbar">
      <!--  toolbar markup here -->
    </mdc-toolbar>

    <main>
      <mdc-fab style="margin: 10px" icon="arrow_back" to="/"></mdc-fab>
      <mdc-card class="card">
        <mdc-card-media src="/static/welcome_card.jpg" :height="200">
          <mdc-card-title style="color: white" v-text="article.Name"></mdc-card-title>
        </mdc-card-media>
        <div style="margin: 20px">
          <span v-text="'创建于'+ unixToTime(article.CreateTime,'yyyy-MM-dd')"></span>
          <span v-text="'最后编辑于'+ unixToTime(article.EditTime,'yyyy-MM-dd')"></span>
          <span style="margin-right: 0; margin-left: 60%" v-text="article.Tag"></span>
        </div>
        <mdc-card-text v-html="context"></mdc-card-text>
      </mdc-card>
      <mdc-fab v-if="token" icon="edit" class="fab-edit" @click="onEdit"></mdc-fab>
      <mdc-fab v-if="token" icon="delete" class="fab-del" @click="onDelete"></mdc-fab>
      <mdc-snackbar ref="snackbar"/>
    </main>

  </mdc-layout-app>
</template>

<script>
    export default {
        name: "Article",
      data(){
          return {
            article:{},
            context:'',
            token : window.localStorage.getItem('token'),
            rawMd:''
          }
      },
      created(){
        this.article = JSON.parse(window.localStorage.getItem('article'))

        this.$http.get('/api/article/uuid/'+this.article.Uuid)
          .then(r=>{
            this.context = r.data
          })

        var url = '/api/article/uuid/'+this.article.Uuid
        this.$http.get(url,{
          params:{
            'mode':'raw'
          }
        }).then((response)=>{
          this.rawMd = response.data
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
        },
        onEdit(){
          this.saveData()
          this.$router.push({path:'/edit'})
        },
        onDelete(){
          this.$http.delete("/api/article/del/"+this.article.Name+"?token="+this.token)
            .then(r=>{
              if (r.data.status === 'no authorized') {
                this.$router.push({path:"/login"})
              }else if (r.data.status === 'success') {
                this.$router.push({path:"/"})
              }else{
                this.$refs.snackbar.show({message: '删除失败!'})
              }
            })
        },
        saveData(){
          let article = {
            oldName:this.article.Name,
            name: this.article.Name,
            tag: this.article.Tag,
            context: this.rawMd
          }
          window.localStorage.setItem('postdata',JSON.stringify(article))
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

  .fab-edit {
    position: fixed;
    right: 40px;
    bottom: 110px;
    z-index: 99999999;
  }

  .fab-del {
    position: fixed;
    right: 40px;
    bottom: 40px;
    z-index: 99999999;
  }
</style>
