<template>
  <mdc-layout-app>

    <mdc-toolbar slot="toolbar">
      <!--  toolbar markup here -->
    </mdc-toolbar>

    <main>
      <mdc-fab style="margin: 10px" icon="arrow_back" @click="onBack"></mdc-fab>
      <mdc-card class="card">
        <mdc-card-media src="/static/welcome_card.jpg" height="200">
          <mdc-card-title style="color: white"></mdc-card-title>
        </mdc-card-media>
        <mdc-card-text>
          <div class="editor">
            <mdc-textfield class="article-tag" v-model="tag" label="Tag"/>
            <mdc-textfield class="article-title" v-model="title" label="标题"/>
            <div id="editormd">
              <textarea id="editormd-markdown-doc" style="display:none;"></textarea>
              <textarea id="editormd-html-code" style="display:none;"></textarea>
            </div>
          </div>
        </mdc-card-text>
      </mdc-card>
      <mdc-fab class="fab-done" icon="done" @click="onSubmit"></mdc-fab>
      <mdc-snackbar ref="snackbar" style="color: red"/>
    </main>
  </mdc-layout-app>
</template>

<script>
  import $script from 'scriptjs';

  export default {
    name: "ArticleEditor",
    data() {
      return {
        tag: "",
        title: "",
        oldTitle:"",
        context:"",
        token : window.localStorage.getItem('token')
      }
    },
    created() {
      this.readData()

      this.initEditor();

    },
    methods: {
      fetchScript: function (url) {
        return new Promise((resolve) => {
          $script(url, () => {
            resolve();
          });
        });
      },
      initEditor() {
        (async () => {
          await this.fetchScript('/static/editor.md/jquery.min.js');
          await this.fetchScript('/static/editor.md/editormd.min.js');
          this.$nextTick(() => {
            let editor = window.editormd('editormd', {
              width: "100%",
              height: 640,
              syncScrolling: "single",
              path: "/static/editor.md/lib/",
              saveHTMLToTextarea: true,
              emoji: true,

            });
            editor.on('load', () => {
              setTimeout(() => {
                this.editorLoaded = true;
                editor.setMarkdown(this.context);
              }, 0);
            });
            editor.on('change', () => {
              this.context = editor.getMarkdown()
            });
            this.editor = editor;
          });
        })();
      },
      onSubmit(){
        if (this.title === ''){
          this.$refs.snackbar.show({message: '请输入标题!'})
          return
        } else if (this.tag === ''){
          this.$refs.snackbar.show({message: '请输入Tag!'})
          return
        } else if (this.context === ''){
          this.$refs.snackbar.show({message: '请输入内容!'})
          return
        }

        if (this.$route.path === '/edit') {
          console.log(this.title)
          this.$http.put("/api/article/edit", {
            token: this.token,
            oldName:this.oldTitle,
            name: this.title,
            tag: this.tag,
            context: this.context
          }).then(r => {
            console.log(r.data)
            if (r.data.status == 'no authorized') {
              this.$refs.snackbar.show({message: '请登录!'})
              this.saveData()
              this.$router.push({path: "/login"})
            } else if (r.data.status != 'success') {
              this.$refs.snackbar.show({message: '提交失败!'})
            } else {
              this.$router.back()
            }
          })
        }else {
          this.$http.post("/api/article/new", {
            token: this.token,
            name: this.title,
            tag: this.tag,
            context: this.context
          }).then(r => {
            console.log(r.data)
            if (r.data.status == 'no authorized') {
              this.$refs.snackbar.show({message: '请登录!'})
              this.saveData()
              this.$router.push({path: "/login"})
            } else if (r.data.status != 'success') {
              this.$refs.snackbar.show({message: '提交失败!'})
            } else {
              this.$router.push({path: "/"})
            }
          })
        }
      },
      saveData(){
        let article = {
          oldName:this.oldTitle,
          name: this.title,
          tag: this.tag,
          context: this.context
        }
        window.localStorage.setItem('postdata',JSON.stringify(article))
      },
      readData(){
        let data = window.localStorage.getItem('postdata')
        if (data === null) {
          return
        }
        let article = JSON.parse(data)
        this.title = article.name
        this.oldTitle = article.oldName
        this.context = article.context
        this.tag = article.tag

        window.localStorage.removeItem("postdata")
      },
      onBack(){
        if (this.$route.path === '/edit') {
          this.$router.back()
        }else {
          this.$router.push({path:"/"})
        }
      }
    },
  }
</script>

<style scoped>
  .card {
    margin-right: 10%;
    margin-left: 10%;
    margin-top: 5%;
  }

  .fab-done {
    position: fixed;
    right: 40px;
    bottom: 40px;
  }

  .editor {
    margin: 0px 20px 20px 20px;
  }

  .article-tag {
    width: 14%;
  }

  .article-title {
    width: 85.6%;
  }
</style>
