<template>
<div>
  <div v-if="!isPostsLoading">    
    <div class="card" v-for="post in posts" :key="post.id">
      <div class="card-content hoverable blue lighten-5">
        <span class="card-title activator grey-text text-darken-4">{{ post.title }}
          <i class="right small material-icons">arrow_drop_down</i></span>        
        <!-- <i class="material-icons right">...</i>< -->
        <p>Author</p>
      </div>
      <div class="card-reveal blue lighten-3">
        <span class="card-title grey-text text-darken-4"><i class="material-icons right">close</i></span>
        <p>{{ post.body }}</p>
      </div>
    </div>       
  </div>
  <!-- прелоудер пока грузит посты -->
  <div class="preloader-wrapper big active" v-else>
    <div class="spinner-layer spinner-blue-only">
      <div class="circle-clipper left">
        <div class="circle"></div>
      </div><div class="gap-patch">
        <div class="circle"></div>
      </div><div class="circle-clipper right">
        <div class="circle"></div>
      </div>
    </div>
  </div>  
</div>
</template>

<script>
/* eslint-disable */
import axios from 'axios'
import materialize from 'materialize-css'

export default {
  name: 'App',
  data() {
    return {
      posts: [],
      cachedUsers: [],
      isPostsLoading: false
    }
  },
  methods: {
    async fetchPosts() {
      try {
        this.isPostsLoading = true;          
        const postsResponse = await axios.get('https://jsonplaceholder.typicode.com/posts?_limit=20');        
        this.posts = postsResponse.data;       
        for (let index = 0; index < this.posts.length; index++) {
          const element = this.posts[index];
          let userId = element.userId;
          let user = await this.getUser(userId);          
        }              
      } catch (e) {
        alert('Ошибка')
      } finally {
        this.isPostsLoading = false;  
      }
    },
    async getUser(id) {      
      let user = this.cachedUsers.find(u => u.id == id);
      
      if (user == undefined) {
        user = (await axios.get(`https://jsonplaceholder.typicode.com/users/${id}`)).data;        
        this.cachedUsers.push(user);                      
        console.log("Fetched from server")
      } else {
        console.log("Fetched from cache")
      }

      return user;
    }
  },
  mounted() {
    this.fetchPosts();
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.md-progress-spinner {
    margin: 24px;
}
</style>
