<template>
  <mdc-layout-app>

    <mdc-toolbar slot="toolbar">
      <!--  toolbar markup here -->
    </mdc-toolbar>

    <main>
      <mdc-card class="card">
        <mdc-card-media src="/static/welcome_card.jpg" :height="200">
          <mdc-card-title style="color: white">安装</mdc-card-title>
        </mdc-card-media>
        <mdc-card-text>
          <mdc-textfield style="width: 100%" type="text" v-model="name" label="博客名"/>
          <mdc-textfield style="width: 100%" type="password" v-model="password" label="密码"/>
          <mdc-select v-model="driver" label="数据库">
            <mdc-option>mysql</mdc-option>
          </mdc-select>
          <mdc-textfield style="width: 100%" type="text" v-model="address" label="数据库地址"/>
          <mdc-textfield style="width: 100%" type="text" v-model="port" label="数据库端口"/>
          <mdc-textfield style="width: 100%" type="text" v-model="db_name" label="数据库名"/>
          <mdc-textfield style="width: 100%" type="text" v-model="db_username" label="数据库用户名"/>
          <mdc-textfield style="width: 100%" type="password" v-model="db_password" label="数据库密码"/>
        </mdc-card-text>
        <mdc-card-actions>
          <mdc-card-action-button @click="onInstall">安装</mdc-card-action-button>
        </mdc-card-actions>
      </mdc-card>
      <mdc-snackbar ref="snackbar"/>
    </main>
  </mdc-layout-app>
</template>

<script>
    export default {
      name: "install",
      data(){
        return{
          name:'goblog',
          password:'',
          driver:'mysql',
          address:'127.0.0.1',
          port:'3306',
          db_name:'goblog',
          db_username:'root',
          db_password:''
        }
      },
      methods:{
          onInstall(){
            this.$http.post("/api/install",{
              name:this.name,
              password:this.password,
              database:{
                driver:this.driver,
                address:this.address,
                port:this.port,
                dbname:this.db_name,
                username:this.db_username,
                password:this.db_password
              }
            }).then(r=>{
              if (r.data === '安装完成') {
                this.$router.push({path:'/'})
              }else{
                this.$refs.snackbar.show({message: '安装失败!'})
              }
            })
          }
      }
    }
</script>

<style scoped>
  .card {
    margin: 100px auto auto;
    width: 500px;
    background: white;
  }
</style>
