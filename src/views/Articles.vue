<template>
<div class="articles-wrapper fluid-container">
  <h1 class="title global-title" v-if="articles.length > 1">
    Articles
  </h1>
  <div class="vert-spacer" v-else>
  </div>
  <div class="articles" v-if="articles.length > 0">
    <div class="article"
      v-for="article in articles"
      :key="article.id"
      @click="toArticle(article.id)">
      <h3>{{article.title}}</h3>
      <p>{{article.body}}</p>
      <em v-if="article.date">{{new Date(article.date).toLocaleDateString("at")}}</em>
      <div class="controls">
        <v-btn @click="back" class="back-button" v-if="articles.length === 1">
          Back
        </v-btn>
        <v-btn class="update" dark
          @click.stop="addAricleOpen(article.id, article.title, article.slug, article.body)">
            Update
          </v-btn>
        <v-btn class="delete" dark
        @click.stop="deleteArticle(article.id)">
          delete
        </v-btn>
      </div>
    </div>
  </div>
  <div class="articles" v-if="articles.length === 0 && !loading">
    <h3>No articles found</h3>
  </div><div class="articles" v-if="articles.length === 0 && loading">
    <h3>Loading...</h3>
  </div>
  <v-btn class="mx-2 add-article" fab dark large @click="addAricleOpen" title="Add article">
    <v-icon dark>mdi-pencil</v-icon>
  </v-btn>
  <add-article @refetch="fetchArticles"></add-article>
</div>
</template>

<script>
const AddArticle = () => import('../components/common/AddArticle.vue');
export default {
  name: 'articles',
  components: {
    AddArticle,
  },
  data() {
    return {
      articles: [],
      loading: null,
    };
  },
  methods: {
    async fetchArticles() {
      this.loading = 'Loading...';
      const url = !this.$route.params.id ? `${process.env.VUE_APP_API}/articles/` : `${process.env.VUE_APP_API}/articles/${this.$route.params.id}/`;
      const response = await fetch(url);
      const data = await response.json();
      if (data.length) {
        this.articles = data;
      } else {
        this.articles = [data];
      }
      this.loading = null;
      console.log(this.loading);
    },
    async deleteArticle(id) {
      try {
        const response = await fetch(`${process.env.VUE_APP_API}/articles/${id}/`, { method: 'DELETE' });
        console.log(response);
        this.fetchArticles();
      // eslint-disable-next-line
      }
      catch (error) {
        console.log(error);
      // eslint-disable-next-line
      }
    },
    toArticle(id) {
      // this.$router.push({ path: `/article/${id}` });
      if (this.$route.params.id !== id) {
        this.$router.push({ name: 'articles', params: { id } });
        this.fetchArticles();
      }
    },
    back() {
      this.$router.go(-1);
    },
    addAricleOpen(id, title, slug, body) {
      this.$modal.show('add-article', {
        id,
        title,
        slug,
        body,
      });
    },
  },
  beforeMount() {
    this.fetchArticles();
  },
  watch: {
    $route: {
      handler() {
        this.fetchArticles();
      },
      deep: true,
    },
  },
};
</script>

<style lang="scss">
.articles-wrapper {
  .title {
    color: $theme;
    margin: 50px 0 30px 0;
  }

  .vert-spacer {
    margin: 30px 0;
  }

  .articles {
    .article {
      margin-bottom:40px;
      padding: 10px;
      cursor: pointer;
      position: relative;
      transition: all 0.2s linear;
      &:hover {
        background: #f0f0f0;
        border-radius: 4px;
        box-shadow: 0 0 6px rgba(0, 0, 0, 0.23);
      }
      &::after {
        content: "";
        border-bottom: 1px solid #f0f0f0;
        width: 100%;
        left: 0;
        bottom: -20px;
        position: absolute;
      }

      .controls {
        display:flex;
        flex-wrap: nowrap;
        justify-content: flex-end;
        margin: 20px 0 10px 0;

        .back-button {
          background-color: $text !important;
          color: $text-inverse;
          border-radius: 4px;
          border: none;
          font-size:0.875em;
          cursor: pointer;
          transition: all 0.2s linear;
          margin-right: 20px;
          width: 93px;

          &:hover {
            background-color: lighten($text, 15%) !important;
          }
        }

        .update {
          background-color: $confirm !important;
        }
        .delete {
          background-color: $alert !important;
          margin-left:20px;
        }
      }
    }
    h3, p {
      color: $text;
      font-weight: 400;
    }

    p {
      text-overflow: ellipsis;
      white-space: wrap;
      overflow: hidden;
    }

    em {
      color: $gray;
      font-size: 0.75em;
      width: 100%;
      display: flex;
      justify-content: flex-end;
    }
  }

  .add-article {
    background-color: $theme !important;
    position: fixed;
    bottom: 50px;
    left: calc(100% - 90px);
  }
}
</style>
