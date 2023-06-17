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

<script setup>
  import { ref, onMounted } from 'vue';

  const posts = ref([])
  const title = ref('')
  const body = ref('')
  const post_id = ref(0)
  const isEditing = ref(false)
  const API_URL = 'http://localhost:3000/posts';

  onMounted(async() => {
    const res = await fetch(API_URL)
    posts.value = await res.json()
  })

  const createPost = async() => {
    const res = await fetch(API_URL, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        title: title.value,
        body: body.value
      })
    })

    const data = await res.json()

    posts.value.push(data)
    title.value = ''
    body.value = ''
    post_id.value = 0
  }

  const editPost = (post) => {
    isEditing.value = true
    title.value = post.title
    body.value = post.body
    post_id.value = post.id

    window.scrollTo({
      top: 0,
      behavior: 'smooth'
    })
  } 

  const updatePost = async() => {
    const res = await fetch(`${API_URL}/${post_id.value}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        title: title.value,
        body: body.value
      })
    });

    const updatedPost = await res.json()

    const postsValue = posts.value


    // 為了防止資料更動後，Vue 畫面上並未更新，因為 Vue 不覺得資料有變動(記憶體位置未改變)
    // 對後端資料 CRUD 時，建議使用 shallow copy
    const dummyPostsValue = [...postsValue] // 變更記憶體位置
    dummyPostsValue.map(post => {
      if (post.id === updatedPost.id) {
        const index = dummyPostsValue.indexOf(post)
        dummyPostsValue[index] = updatedPost
      }
    });
    posts.value = dummyPostsValue // 把新記憶體位置的資料重新帶入
    title.value = ''
    body.value = ''
    post_id.value = 0
    isEditing.value = false
  }

  const cancelEdit = () => {
    title.value = ''
    body.value = ''
    post_id.value = ''
    isEditing.value = false
  }

  const deletePost = async(id) => {
    await fetch(`${API_URL}/${id}`, {
      method: 'DELETE'
    })

    posts.value = posts.value.filter(post => post.id !== id)
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