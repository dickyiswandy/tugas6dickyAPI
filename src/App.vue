<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-toolbar-title>API Project</q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page class="q-pa-md">
        <q-form @submit.prevent="save" class="q-gutter-md">
          <q-input v-model="form.title" label="Title" filled />
          <q-input v-model="form.content" label="Content" type="textarea" filled autogrow />
          <q-btn type="submit" label="Save" color="primary" />
        </q-form>

        <q-list bordered class="q-mt-md">
          <q-item v-for="article in articles" :key="article.id" clickable>
            <q-item-section>
              <q-item-label>{{ article.title }}</q-item-label>
              <q-item-label caption>{{ article.content }}</q-item-label>
            </q-item-section>
            <q-item-section side>
              <q-btn icon="edit" @click="edit(article)" flat round color="primary" />
              <q-btn icon="delete" @click="deleteArticle(article.id)" flat round color="negative" />
            </q-item-section>
          </q-item>
        </q-list>

        <q-btn @click="load" label="Load" class="q-mt-md" />
      </q-page>
    </q-page-container>

    <q-footer class="q-pa-md">
      <p>&copy; 2024 Integrasi Dengan API. All rights reserved.</p>
      <p>Contact Person: Dicky Iswandy</p>
    </q-footer>
  </q-layout>
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue';
import axios from 'axios';
import { QForm, QInput, QBtn, QList, QItem, QItemSection, QItemLabel, QLayout, QHeader, QToolbar, QToolbarTitle, QPageContainer, QPage, QFooter } from 'quasar';

const API_KEY = '$2a$10$VBm/nsEOxSQ6r9cDubxr6euumsm1nMeebIcVIXey5/CsSsNfexKl2';  // Replace with your JSONBin API key
const BIN_ID = '665ae50aacd3cb34a8510830';  // Replace with your JSONBin Bin ID

const form = reactive({
  id: null,
  title: '',
  content: '',
});

const articles = ref([]);

async function load() {
  try {
    const response = await axios.get(`https://api.jsonbin.io/v3/b/${BIN_ID}/latest`, {
      headers: {
        'X-Master-Key': API_KEY,
      }
    });
    if (Array.isArray(response.data.record)) {
      articles.value = response.data.record;
    } else {
      console.error('Unexpected data format:', response.data.record);
    }
  } catch (error) {
    console.error('Error loading articles', error);
  }
}

async function save() {
  try {
    console.log("Saving article:", form);
    let updatedArticles = Array.isArray(articles.value) ? [...articles.value] : [];
    if (form.id) {
      // Update existing article
      const index = updatedArticles.findIndex(article => article.id === form.id);
      if (index !== -1) {
        updatedArticles[index] = { ...form };
      }
    } else {
      // Create new article
      form.id = Date.now().toString(); // Generate a simple unique ID
      updatedArticles.push({ ...form });
    }
    console.log("Updated articles:", updatedArticles);
    await axios.put(`https://api.jsonbin.io/v3/b/${BIN_ID}`, { record: updatedArticles }, {
      headers: {
        'Content-Type': 'application/json',
        'X-Master-Key': API_KEY,
        'X-Bin-Versioning': 'false'  // Disable versioning if not needed
      }
    });
    articles.value = updatedArticles;
    resetForm(); // Reset the form
  } catch (error) {
    console.error('Error saving article', error);
  }
}

function edit(article) {
  form.id = article.id;
  form.title = article.title;
  form.content = article.content;
}

async function deleteArticle(id) {
  try {
    if (Array.isArray(articles.value)) {
      const updatedArticles = articles.value.filter(article => article.id !== id);
      await axios.put(`https://api.jsonbin.io/v3/b/${BIN_ID}`, { record: updatedArticles }, {
        headers: {
          'Content-Type': 'application/json',
          'X-Master-Key': API_KEY,
          'X-Bin-Versioning': 'false'  // Disable versioning if not needed
        }
      });
      articles.value = updatedArticles;
    } else {
      console.error('articles.value is not an array:', articles.value);
    }
  } catch (error) {
    console.error('Error deleting article', error);
  }
}

function resetForm() {
  form.id = null;
  form.title = '';
  form.content = '';
}

onMounted(load);
</script>

<style>
/* Add your custom styles here, if needed */
</style>
