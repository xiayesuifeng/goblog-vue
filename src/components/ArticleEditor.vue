<template>
  <mdc-layout-app>

    <mdc-toolbar slot="toolbar">
      <!--  toolbar markup here -->
    </mdc-toolbar>

    <main>
      <mdc-fab style="margin: 10px" icon="arrow_back" to="/"></mdc-fab>
      <mdc-card class="card">
        <mdc-card-media src="/static/welcome_card.jpg" height="200">
          <mdc-card-title style="color: white"></mdc-card-title>
        </mdc-card-media>
        <mdc-card-text>
          <div class="editor">
            <mdc-textfield class="article-tag" v-model="tag" label="Tag"/>
            <mdc-textfield class="article-title" v-model="title" label="标题"/>
            <div id="editormd">
              <textarea id="editormd-markdown-doc" v-model="context" style="display:none;"></textarea>
              <textarea id="editormd-html-code" style="display:none;"></textarea>
            </div>
          </div>
        </mdc-card-text>
      </mdc-card>
      <mdc-fab class="fab-done" icon="done" @click="onSubmit"></mdc-fab>
      <mdc-snackbar ref="snackbar"/>
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
        context:""
      }
    },
    created() {
      if (this.$route.path === '/edit') {
        this.tag = this.$route.post("tag")
        this.title = this.$route.post("title")
      }

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
                this.initData && editor.setMarkdown(this.initData);
              }, this.initDataDelay);
            });
            this.onchange && editor.on('change', () => {
              let html = editor.getPreviewedHTML();
              this.onchange({
                markdown: editor.getMarkdown(),
                html: html,
                text: window.$(html).text()
              });
            });
            this.editor = editor;
          });
        })();
      },
      onSubmit(){

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
