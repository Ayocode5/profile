<template>
  <section>
    <page-header />

    <div v-if="isReady">
      <div :key="routeChangesIdentifier">
        <JumbotronComponent />
        <div class="col-10 offset-1 col-sm-10 offset-sm-1 col-xl-8 offset-xl-2 col-lg-10 offset-lg-1 col-md-10 offset-md-1 mt-3">
          <main role="main" class="mt-5">
            <div>
              <h4 class="my-4 border-bottom mt-5 pb-2">
                <span class="border-bottom border-dark pb-2"
                  >Search For : {{ $route.params.keyword }}</span
                >
              </h4>

              <div :key="`${index}-${post.id}`" v-for="(post, index) in posts">
                <router-link
                  :to="{ name: 'show-post', params: { slug: post.slug } }"
                  class="text-decoration-none"
                >
                  <div class="card mb-4">
                    <div class="card-body px-0">
                      <div
                        class="container-fluid d-lg-inline-flex align-items-center"
                      >
                        <div
                          v-if="post.featured_image"
                          class="col-12 col-lg-3 p-0"
                        >
                          <img
                            :src="post.featured_image"
                            :alt="post.featured_image_caption"
                            class="card-img-top rounded w-100"
                          />
                        </div>
                        <section
                          class="col-12 mt-3 mt-lg-0 px-0 px-lg-3"
                          :class="post.featured_image ? 'col-lg-9' : ''"
                        >
                          <h5 class="card-title text-truncate mb-0">
                            {{ post.title }}
                          </h5>
                          <p class="card-text text-truncate">
                            {{ post.summary }}
                          </p>
                          <p class="card-text mb-0 text-secondary">
                            {{ post.user.name }}
                            <span v-if="post.topic.length">
                              in {{ post.topic[0].name }}
                            </span>
                          </p>
                          <p class="card-text text-secondary">
                            {{ moment(post.published_at).format("MMM D, Y") }} —
                            {{ post.read_time }}
                          </p>
                        </section>
                      </div>
                    </div>
                  </div>
                </router-link>
              </div>

              <infinite-loading spinner="spiral" @infinite="searchPost">
                <span slot="no-more" />
                <div slot="no-results" class="text-left">
                  <div class="my-5">
                    <p class="lead text-center text-muted mt-5">
                      You have no published posts
                    </p>
                    <p class="lead text-center text-muted mt-1">
                      Write on the go with our mobile-ready app!
                    </p>
                  </div>
                </div>
              </infinite-loading>

              <div v-if="noMatchPost">
                <span slot="no-more" />
                <div slot="no-results" class="text-left">
                  <div class="my-5">
                    <p class="lead text-center text-muted mt-5">
                      No matched posts found! 
                    </p>
                  </div>
                </div>
              </div>
            </div>
          </main>
        </div>
      </div>
    </div>
    <!-- <Footer /> -->
    <PageFooterComponent />
  </section>
</template>

<script>
import InfiniteLoading from "vue-infinite-loading";
import NProgress from "nprogress";
import PageHeader from "../components/PageHeaderComponent";
import JumbotronComponent from "../components/JumbotronComponent";
// import Footer from "../components/Footer";
import PageFooterComponent from "../components/PageFooterComponent";
import isEmpty from "lodash/isEmpty";

export default {
  name: "search-posts",

  components: {
    InfiniteLoading,
    PageHeader,
    JumbotronComponent,
    PageFooterComponent
    // Footer,
  },

  metaInfo() {
    return {
      title: "ayocode | posts",
    };
  },

  data() {
    return {
      page: 1,
      posts: [],
      isReady: false,
      noMatchPost: false,
      routeChangesIdentifier: 0,
    };
  },

  async created() {
    await Promise.all([this.searchPost()]);
    this.isReady = true;
    NProgress.done();
  },

  methods: {
    searchPost($state) {
      if ($state) {
        return this.request()
          .get("api/posts", {
            params: {
              search: this.$route.params.keyword,
              page: this.page,
            },
          })
          .then(({ data }) => {
            if (!isEmpty(data) && !isEmpty(data.data)) {
              this.page += 1;
              this.posts.push(...Object.values(data.data));
              $state.loaded();
            } else {
              $state.complete();
            }

            if (isEmpty($state)) {
              NProgress.inc();
            }
          })
          .catch((err) => {
            if (err.response.status == 404) {
              this.noMatchPost = true;
              $state.loaded();
              $state.complete();
            } else if (err.response.status == 500) {
              // $state.loaded();
              $state.complete();
            }
          });
      }
    },
  },

  watch: {
    "$route.params.keyword": async function () {
      this.page = 1;
      this.posts = [];
      this.noMatchPost = false;

      await Promise.all([this.searchPost()]);
      this.isReady = true;
      NProgress.done();

      this.routeChangesIdentifier += 1;
    },
  },
};
</script>

<style scoped>
/* .card {
  border: none;
  background: rgb(255,255,255);
background: linear-gradient(90deg, rgba(255,255,255,1) 0%, rgba(247,247,247,1) 50%, rgba(242,242,242,1) 100%);
} */

</style>