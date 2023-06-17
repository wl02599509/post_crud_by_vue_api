<template>
  <div>
    <h1>Posts</h1>

    <input type="text" placeholder="Text" v-model="title" class="title-input">
    <input type="text" placeholder="Body" v-model="body" class="body-input">

    <button v-if="isEditing" @click="updatePost">Update</button>
    <button v-if="isEditing" @click="cancelEdit">Cancel</button>

    <button v-else @click="createPost">Create</button>

    <div v-for="post in posts" :key="post.id">
      <h5>{{post.id}}.{{ post.title }}</h5>
      <p>{{ post.body }}</p>
      <button @click="editPost(post)">Edit</button>
      <button @click="deletePost(post.id)">Delete</button>
    </div>
  </div>
</template>

<script>
  export default{
    data () {
      return {
        posts: [],
        title: '',
        body: '',
        post_id: 0,
        isEditing: false,
        API_URL: 'http://localhost:3000/posts'
      }
    },
    methods: {
      async createPost() {
        const res = await fetch(this.API_URL, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            title: this.title,
            body: this.body
          })
        });

        const data = await res.json();

        this.title = '';
        this.body = '';
        this.post_id = 0;
        this.posts.push(data);
      },
      editPost(post) {
        this.isEditing = true
        this.title = post.title
        this.body = post.body
        this.post_id = post.id

        window.scrollTo({
          top: 0,
          behavior: 'smooth'
        })
      },
      async updatePost() {
        const res = await fetch(`${this.API_URL}/${this.post_id}`, {
          method: 'PUT',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            title: this.title,
            body: this.body
          })
        });

        const updatedPost = await res.json()

        const postsValue = this.posts


        // 為了防止資料更動後，Vue 畫面上並未更新，因為 Vue 不覺得資料有變動(記憶體位置未改變)
        // 對後端資料 CRUD 時，建議使用 shallow copy
        const dummyPostsValue = [...postsValue] // 變更記憶體位置
        dummyPostsValue.map(post => {
          if (post.id === updatedPost.id) {
            const index = dummyPostsValue.indexOf(post)
            dummyPostsValue[index] = updatedPost
          }
        });
        this.posts = dummyPostsValue // 把新記憶體位置的資料重新帶入
        this.title = ''
        this.body = ''
        this.post_id = 0
        this.isEditing = false
      },
      cancelEdit() {
        this.title = ''
        this.body = ''
        this.post_id = ''
        this.isEditing = false
      },
      async deletePost(post_id) {
        await fetch(`${this.API_URL}/${post_id}`, {
          method: 'DELETE'
        })

        return this.posts = this.posts.filter(post => post.id !== post_id)
      }
    },
    async mounted() {
      const res = await fetch(this.API_URL)
      this.posts = await res.json()
    }
  }
</script>

<style scoped>
  .title-input {
    width: 100%;
    padding: 12px 20px;
    margin: 8px 0;
    box-sizing: border-box;
    border: 2px solid #ccc;
    background-color: #f8f8f8;
    color: #111;
    border-radius: 4px;
    resize: vertical;
  }

  .body-input {
    width: 100%;
    padding: 12px 20px;
    margin: 8px 0;
    box-sizing: border-box;
    border: 2px solid #ccc;
    background-color: #f8f8f8;
    color: #111;
    border-radius: 4px;
    resize: vertical;
  }
</style>