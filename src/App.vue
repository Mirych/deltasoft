<template>
<div class="container">
  <h1>Посты</h1>
  <div class="row">
    <div class="col s6">
      <div class="input-field">
        <input 
          v-model="searchQuery"      
          type="search" 
          
        >
        <label for="search">Поиск по названию</label>
      </div>
    </div>
    <div class="col s6">
      <div class="input-field">
        <my-select 
        v-model="selectedSort"
        :options="sortOptions"
        />
      </div>
    </div>
  </div>
  <div v-if="!isPostsLoading">      
    <div class="card" v-for="post in sortedAndSearchedPosts" :key="post">
      <div class="card-content hoverable teal lighten-5">        
        <span class="card-title activator grey-text text-darken-4">{{ post.title }}
          <i class="right small material-icons">arrow_drop_down</i></span>                
        <p>{{post.name}}</p>
      </div>
      <div class="card-reveal teal lighten-3">
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
  <ul class="pagination">
      <li class="disabled"><a href="#!"><i class="material-icons">chevron_left</i></a></li>      
      <li 
        v-for="pageNumber in totalPages" 
        :key="pageNumber" 
        class="waves-effect" 
        :class="{
          'active teal lighten-3': page === pageNumber
        }"
      >
        <a @click="changePage(pageNumber)">{{ pageNumber }}</a>
      </li>
      <li class="waves-effect"><a href="#!"><i class="material-icons">chevron_right</i></a></li> 
  </ul> 
</div>
</template>

<script>
/* eslint-disable */
import axios from 'axios'
import materialize from 'materialize-css'
import MySelect from './components/MySelect.vue'

export default {
  components: { MySelect },
  name: 'App',
  data() {
    return {
      posts: [],
      cachedUsers: [],
      isPostsLoading: false,
      searchQuery: '',
      page: 1,
      limit: 10,
      totalPages: 0,
      selectedSort: '',
      sortOptions: [
        {value: 'title', name: 'По названию'},
        {value: 'body', name: 'По содержимому'}
      ]
    }
  },
  methods: {
    async fetchPosts() {
      try {
        this.isPostsLoading = true;          
        const postsResponse = await axios.get('https://jsonplaceholder.typicode.com/posts', {
          params: {
            _page: this.page,
            _limit: this.limit
          }
        }); 
        this.totalPages = Math.ceil(postsResponse.headers['x-total-count'] / this.limit);
        this.posts = postsResponse.data;       
        for (let index = 0; index < this.posts.length; index++) {
          const element = this.posts[index];
          let userId = element.userId;
          let user = await this.getUser(userId);   
          this.posts[index].name = user.name;                         
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
      }

      return user;
    },
    changePage(pageNumber) {
      this.page = pageNumber;
      this.fetchPosts();
    }
  },
  mounted() {
    this.fetchPosts();
    var elems = document.querySelectorAll('select');
    var instances = M.FormSelect.init(elems);
  },
  computed: {
    sortedPosts() {
      return [...this.posts].sort((post1, post2) => post1[this.selectedSort]?.localeCompare(post2[this.selectedSort]))
    },
    sortedAndSearchedPosts() {
      return this.sortedPosts.filter(post => post.title.toLowerCase().includes(this.searchQuery.toLowerCase()))
    }   
  },
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
