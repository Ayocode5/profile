<template>
  <section :key="routeChangesIdentifier">
    <page-header>
      <template slot="options">
        <div class="dropdown">
          <a
            href="#"
            class="nav-link pr-1"
            role="button"
            data-toggle="dropdown"
            aria-haspopup="true"
            aria-expanded="false"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 24 24"
              width="25"
              class="icon-dots-horizontal"
            >
              <path
                class="fill-primary"
                fill-rule="evenodd"
                d="M5 14a2 2 0 1 1 0-4 2 2 0 0 1 0 4zm7 0a2 2 0 1 1 0-4 2 2 0 0 1 0 4zm7 0a2 2 0 1 1 0-4 2 2 0 0 1 0 4z"
              />
            </svg>
          </a>
          <div
            v-if="isReady"
            class="dropdown-menu dropdown-menu-right"
            aria-labelledby="actionDropdownMenu"
          >
            <a
              v-if="hasAccess"
              :href="`/${CanvasUI.canvasPath}/posts/${post.id}/edit`"
              class="dropdown-item"
            >
              Edit post
            </a>
            <a
              :href="`/${CanvasUI.canvasPath}/stats/${post.id}`"
              class="dropdown-item"
            >
              View stats
            </a>
          </div>
        </div>
      </template>
    </page-header>

    <div v-if="isReady" class="mt-5">
      <div
        class="
          col-10
          offset-1
          col-sm-10
          offset-sm-1
          col-md-10
          offset-md-1
          col-lg-8
          offset-lg-2
          col-xl-8
          offset-xl-2
          col-xxl-8
          offset-xxl-2
        "
      >
        <h1>{{ post.title }}</h1>

        <div class="row row-cols-auto media pt-1 pb-5">
          <div class="ml-3">
            <router-link
              :to="{ name: 'show-user', params: { id: post.user.id } }"
            >
              <img
                :src="post.user.avatar || post.user.default_avatar"
                class="mr-1 rounded-circle shadow-inner"
                style="width: 50px"
                :alt="post.user.name"
              />
            </router-link>
          </div>

          <div class="col">
            <div class="media-body">
              <p class="my-0">
                <span>
                  <router-link
                    :to="{ name: 'show-user', params: { id: post.user.id } }"
                    class="text-decoration-none"
                  >
                    {{ post.user.name }}
                  </router-link>
                </span>
                <span v-if="post.topic.length">
                  in
                  <router-link
                    :to="{
                      name: 'show-topic',
                      params: { slug: post.topic[0].slug },
                    }"
                    class="text-decoration-none"
                  >
                    {{ post.topic[0].name }}
                  </router-link>
                </span>
              </p>
              <span class="text-secondary"
                >{{ moment(post.published_at).format("MMM D, Y") }} —
                {{ post.read_time }}</span
              >
            </div>
          </div>
        </div>

        <img
          v-if="post.featured_image"
          :src="post.featured_image"
          class="pt-4 img-fluid w-100"
          :alt="post.featured_image_caption"
          :title="post.featured_image_caption"
        />

        <p
          v-if="post.featured_image && post.featured_image_caption"
          class="text-muted text-center featured-image-caption"
          v-html="post.featured_image_caption"
        />

        <div
          class="
            post-content
            position-relative
            align-items-center
            overflow-y-visible
            mt-4
          "
        >
          <div v-html="post.body" />
        </div>

        <div v-if="post.tags.length" class="mt-5">
          Tags:
          <div v-for="tag in post.tags" :key="tag.id" class="badge p-2 my-1" :class="getRandomColour()" style="margin-right: 2px">
            <router-link
              :to="{ name: 'show-tag', params: { slug: tag.slug } }" class="text-decoration-none text-white"
            >
              {{ tag.name }}
            </router-link>
          </div>
        </div>

        <div
          v-if="post.meta.canonical_link"
          class="
            post-content
            position-relative
            align-items-center
            overflow-y-visible
            font-arial
          "
        >
          <hr />
          <p class="text-center font-italic mb-5">
            This post was originally published on
            <a
              :href="post.meta.canonical_link"
              target="_blank"
              rel="noopener"
              >{{ parseURL(post.meta.canonical_link).host }}</a
            >
          </p>
        </div>
      </div>

      <!-- Comment Reply Component -->
      <comment-reply-component :post_id="post.id" :post_slug="post.slug" />

      <!-- Related Posts Component -->
      <div v-if="post.topic.length > 0">
        <related-posts-component
          :related_params="{
            topic: post.topic[0].id,
            current_post_id: post.id,
          }"
        />
      </div>

      <!-- Popular Posts Component -->

      <div
        class="
          col-10
          offset-1
          col-sm-10
          offset-sm-1
          col-md-10
          offset-md-1
          col-lg-8
          offset-lg-2
          col-xl-8
          offset-xl-2
          col-xxl-8
          offset-xxl-2
          mb-5
        "
      >
        <popular-posts-component />
      </div>

      <!-- Footer Component -->
      <PageFooterComponent />
    </div>
  </section>
</template>

<script>
import NProgress from "nprogress";
import PageHeader from "../components/PageHeaderComponent";
import PopularPostsComponent from "../components/PopularPostsComponent";
import RelatedPostsComponent from "../components/RelatedPostsComponent";
import CommentReplyComponent from "../components/CommentReplyComponent";
import PageFooterComponent from "../components/PageFooterComponent";
import hljs from "highlight.js";
import mediumZoom from "medium-zoom";

export default {
  name: "show-post",

  components: {
    PageHeader,
    PopularPostsComponent,
    RelatedPostsComponent,
    CommentReplyComponent,
    PageFooterComponent,
  },

  metaInfo() {
    return {
      title: this.post?.meta?.title,
      meta: [
        { name: "description", content: this.post?.meta?.description },
        { property: "og:title", content: this.post?.meta?.title },
        { property: "og:image", content: this.post?.featured_image },
        { property: "og:description", content: this.post?.meta?.description },
        { name: "twitter:card", content: "summary" },
        { name: "twitter:title", content: this.post?.meta?.title },
        { name: "twitter:description", content: this.post?.meta?.description },
        { name: "twitter:image", content: this.post?.featured_image },
      ],
    };
  },

  data() {
    return {
      uri: this.$route.params.slug,
      page: 1,
      post: null,
      isReady: false,
      routeChangesIdentifier: 0,
      colours: ["danger", "info", "primary", "secondary", "success", "warning"],
    };
  },

  computed: {
    hasAccess() {
      if (this.CanvasUI.user) {
        return (
          this.CanvasUI.user.id === this.post.user.id ||
          this.isAdmin ||
          this.isEditor
        );
      } else {
        return false;
      }
    },
  },

  async created() {
    await Promise.all([this.fetchPost()]);

    // Hack since vue-meta doesn't seem to like canonical tags
    if (this.post?.meta?.canonical_link != null) {
      let link = document.createElement("link");
      link.rel = "canonical";
      link.href = this.post.meta.canonical_link;
      document.head.appendChild(link);
    }

    this.isReady = true;
    NProgress.done();
  },

  updated() {
    document.querySelectorAll(".embedded_image img").forEach((image) => {
      mediumZoom(image);
    });

    document.querySelectorAll("pre").forEach((block) => {
      hljs.highlightBlock(block);
    });
  },

  beforeRouteLeave(to, from, next) {
    // Hack to remove the canonical tag when you navigate away
    document.querySelector('link[rel="canonical"]')?.remove();
    next();
  },

  methods: {
    fetchPost() {
      return this.request()
        .get(`/api/posts/${this.uri}`)
        .then(({ data }) => {
          this.post = data;
          NProgress.inc();
        })
        .catch(() => {
          NProgress.done();
        });
    },

    randomColor() {
      let colors = ["#ff0000", "#00ff00", "#0000ff"];
      let random_color = colors[Math.floor(Math.random() * colors.length)];
      document.getElementById("title").style.color = random_color;
    },

    getRandomColour() {
      return (
        "bg-" + this.colours[Math.floor(Math.random() * this.colours.length)]
      );
    },
  },

  watch: {
    "$route.params.slug": async function () {
      this.uri = this.$route.params.slug;
      this.page = 1;
      this.post = null;
      this.isReady = false;

      await Promise.all([this.fetchPost()]);

      this.isReady = true;
      this.routeChangesIdentifier += 1;
      NProgress.done();
    },
  },
};
</script>
<style lang="scss">
pre.ql-syntax {
  background-color: #f5f5f5;
  overflow: none;
}

.ql-editor pre {
  white-space: none;
}

.post-content pre {
  border-radius: 3px;
}

.post-content {
  // margin: 1.5em 0 0 0 !important;
  font-family: "Roboto" sans-serif !important;
  margin: 15px 0 0 0 !important;
}
</style>