<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input"/><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="submit" class="button save-button">Save</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <strong>{{ article.title }}</strong> <br />
        <p>{{ article.content }}</p>
        <button @click="edit(article)" class="button edit-button">Edit</button>
        <button @click="remove(article.id)" class="button delete-button">Delete</button>
      </li>
    </ul>
    <button @click="load" class="button load-button">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: '',
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        let url, method;
        if (form.id === null) {
          const lastArticle = articles.value[articles.value.length - 1];
          const newId = lastArticle ? parseInt(lastArticle.id) + 1 : 1;
          form.id = newId.toString(); 
          url = 'http://localhost:3000/articles/';
          method = 'post';
        } else {
          url = 'http://localhost:3000/articles/' + form.id;
          method = 'put';
        }
        const response = await axios[method](url, form);
        articles.value = articles.value.map((article) =>
          article.id === response.data.id ? response.data : article
        );
        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.log('Error saving article: ', error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete('http://localhost:3000/articles/${id}');
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, remove, load };
  },
};
</script>