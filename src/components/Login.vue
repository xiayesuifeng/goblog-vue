<template>
  <mdc-layout-app>

    <mdc-toolbar slot="toolbar">
      <!--  toolbar markup here -->
    </mdc-toolbar>

    <main>
      <mdc-fab style="margin: 10px" icon="arrow_back" @click="$router.back()"></mdc-fab>
      <mdc-card class="card">
        <mdc-card-media src="/static/welcome_card.jpg" :height="200">
          <mdc-card-title style="color: white">登录</mdc-card-title>
        </mdc-card-media>
        <mdc-card-text>
          <mdc-textfield style="width: 100%" type="password" v-model="password" label="密码"/>
        </mdc-card-text>
        <mdc-card-actions>
          <mdc-card-action-button @click="onLogin">登录</mdc-card-action-button>
        </mdc-card-actions>
      </mdc-card>
      <mdc-snackbar ref="snackbar"/>
    </main>
  </mdc-layout-app>
</template>

<script>
  export default {
    name: "login",
    data(){
      return {
        password:""
      }
    },
    methods: {
      onLogin() {
        if (this.password===''){
          this.$refs.snackbar.show({ message: '请输入密码!' })
          return
        }
        this.$http.post('/api/login', "password="+this.password, {headers: {'Content-Type': 'application/x-www-form-urlencoded'}}).then(r => {
          console.log(r.data)
          if (r.data.token === '') {
            this.$refs.snackbar.show({message: '密码错误!'})
            return
          }
          window.localStorage.setItem('token', r.data.token)
          this.$router.back()
        })
      }
    }
  }
</script>

<style scoped>
  .card {
    margin: 200px auto auto;
    width: 500px;
    background: white;
  }
</style>
