<template>
  <div class="post">

    <!-- header -->
    <div class="post_header mt-4">
      <!-- title -->
      <h3 class="float-left">{{ post.title }}</h3>

      <!-- buttons -->
      <div v-if="typeof post.author !== 'undefined'" class="btn-group float-right" role="group" aria-label="Basic example">
        <router-link
          v-if="$gate.allow('update', 'post', post)"
          class="btn btn-success"
          :to="'/posts/update-post/' + this.post.slug"
        >Update</router-link>

        <template v-if="$gate.allow('publish', 'post', post)">
          <button v-if="post.published == 0" type="button" class="btn btn-info btn-sm" @click="publishPost">publish</button>
          <button v-else type="button" class="btn btn-warning btn-sm" @click="unpublishPost">unpublish</button>
        </template>

        <button v-if="$gate.allow('delete', 'post', post)" type="button" class="btn btn-danger" @click="deletePost">delete</button>
      </div>
      <span class="clearfix d-block mb-3"></span>

      <!-- tags -->
      <div class="tags">
        <span>tags:</span>
        <ul v-if="typeof post.tags !== 'undefined' && post.tags.length !== 0">
          <li v-for="tag in post.tags">
            <router-link class="bg-primary" :to="'/tags/' + tag.slug + '/posts'">
              <font-awesome-icon icon="tag" />
              {{ tag.name }}
            </router-link>
          </li>
        </ul>
        <a v-if="$gate.allow('assignTags', 'post', post)" class="ml-3" href="#" @click.prevent="assignTags">reassign tags...</a>
      </div>
    </div>

    <!-- info -->
    <hr>
    <p class="lead post_info">
      by
      <template v-if="typeof post.author !== 'undefined'">
        <router-link :to="'/users/' + post.author.slug + '/posts'">{{ post.author.name }}</router-link>
      </template>
      <template v-else>
        unknown
      </template>
      <span v-if="post.published == 1" class="float-right"><strong>Published:</strong> {{ post.published_at }}</span>
    </p>
    <hr>

    <!-- Post Content -->
    <div class="post_content">
      <div v-if="caption != 0" class="text-center">
        <img class="img-fluid" :src="caption" alt="post_caption">
        <hr>
      </div>
      <p v-html="post.body"></p>
    </div>
    <hr>

    <div v-if="typeof comments !== 'undefined' && comments.length !== 0" class="comments mt-3 mb-5">
      <h4 class="mb-4">Comments</h4>

      <template v-for="comment in comments">
        <post-comment
          :comment="comment"
        ></post-comment>
      </template>


      <!-- TODO: Comment with nested comments -->
      <!-- <div class="media mb-4">
        <img class="d-flex mr-3 rounded-circle" src="http://placehold.it/50x50" alt="">
        <div class="media-body">
          <h5 class="mt-0">Commenter Name</h5> Cras sit amet nibh libero, in gravida nulla. Nulla vel metus scelerisque ante sollicitudin. Cras purus odio, vestibulum in vulputate at, tempus viverra turpis. Fusce condimentum nunc ac nisi vulputate fringilla. Donec lacinia congue felis
          in faucibus.

          <div class="media mt-4">
            <img class="d-flex mr-3 rounded-circle" src="http://placehold.it/50x50" alt="">
            <div class="media-body">
              <h5 class="mt-0">Commenter Name</h5> Cras sit amet nibh libero, in gravida nulla. Nulla vel metus scelerisque ante sollicitudin. Cras purus odio, vestibulum in vulputate at, tempus viverra turpis. Fusce condimentum nunc ac nisi vulputate fringilla. Donec lacinia congue
              felis in faucibus.
            </div>
          </div>

          <div class="media mt-4">
            <img class="d-flex mr-3 rounded-circle" src="http://placehold.it/50x50" alt="">
            <div class="media-body">
              <h5 class="mt-0">Commenter Name</h5> Cras sit amet nibh libero, in gravida nulla. Nulla vel metus scelerisque ante sollicitudin. Cras purus odio, vestibulum in vulputate at, tempus viverra turpis. Fusce condimentum nunc ac nisi vulputate fringilla. Donec lacinia congue
              felis in faucibus.
            </div>
          </div>

        </div>
      </div> -->
    </div>

  </div>
</template>

<script>
import tagsForm from './tags_form';
import postComment from './post_comment';

export default {
  components: {
    postComment
  },


  computed: {
    post: function () {
      return this.$store.getters['posts/post'];
    },
    comments: function () {
      return this.$store.getters['comments/comments'];
    },
    caption: function () {
      if (!!this.post.caption) {
        return window.Laravel.baseURL + '/storage/posts/' + this.post.caption;
      }
      return 0;
    }
  },


  created: function () {
    this.$store.dispatch('posts/getPost', this.$route.params.post);
    this.$store.dispatch('comments/getPostComments', this.$route.params.post);
  },


  methods: {
    deletePost() {
      this.$store.dispatch('message/update', {
        title: this.post.title,
        body: 'Are you sure you want to delete this Post?',
        class: 'danger',
        confirm: true
      });

      this.$bus.$once('proceed', () => {
        this.$store.dispatch('posts/deletePost', this.post);
        this.$store.dispatch('message/close');
        this.$router.push('/posts');
      });
    },

    publishPost() {
      this.$store.dispatch('posts/publish', this.post);
    },

    unpublishPost() {
      this.$store.dispatch('posts/unpublish', this.post);
    },

    assignTags() {
      this.$store.dispatch('modal/update', {
        action: 'update',
        formData: this.post,
        title: 'Assign tags to the post',
        component: tagsForm
      });
    }
  }
}
</script>
