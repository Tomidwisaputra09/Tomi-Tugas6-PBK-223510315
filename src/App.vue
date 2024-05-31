<template>
  <div class="container">
    <!-- Form -->
    <form @submit.prevent="save" class="form">
      <input
        type="text"
        v-model="form.title"
        placeholder="Title"
        class="input"
      /><br />
      <textarea
        v-model="form.content"
        placeholder="Content"
        class="textarea"
      ></textarea
      ><br />
      <button type="submit" class="button primary-button">Save</button>
    </form>

    <!-- List of Articles -->
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <div class="article-header">{{ article.title }}</div>
        <div class="article-content">{{ article.content }}</div>
        <div class="article-buttons">
          <button @click="edit(article)" class="button edit-button">
            Edit
          </button>
          <button
            @click="deleteArticle(article.id)"
            class="button delete-button"
          >
            Delete
          </button>
        </div>
      </li>
    </ul>

    <!-- Load Button -->
    <button @click="load" class="button primary-button load-button">
      Load
    </button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from "vue";
import axios from "axios";

export default {
  setup() {
    const form = reactive({
      id: "",
      title: "",
      content: "",
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get("http://localhost:3000/articles");
        articles.value = response.data;
      } catch (error) {
        console.error("Error loading articles:", error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : "http://localhost:3000/articles";
        const method = form.id ? "put" : "post";
        const response = await axios[method](url, {
          title: form.title,
          content: form.content,
        });

        // Assuming successful save, update UI and reset form
        if (form.id) {
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          articles.value.push(response.data);
        }

        // Reset form
        form.id = "";
        form.title = "";
        form.content = "";
      } catch (error) {
        console.error("Error saving article:", error);
      }
    }

    async function deleteArticle(id) {
      try {
        // Remove the article from the local array first
        articles.value = articles.value.filter((article) => article.id !== id);
        // Then send delete request to the server
        await axios.delete(`http://localhost:3000/articles/${id}`);
      } catch (error) {
        console.error("Error deleting article:", error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, deleteArticle, load };
  },
};
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: "Arial", sans-serif;
  background-color: #f0f2f5;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.form {
  margin-bottom: 20px;
  padding: 20px;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.input,
.textarea {
  width: 100%;
  padding: 12px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
  box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.1);
}

.button {
  padding: 12px 20px;
  font-size: 16px;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.primary-button {
  background-color: #007bff;
}

.primary-button:hover {
  background-color: #0056b3;
}

.delete-button {
  background-color: #dc3545;
}

.delete-button:hover {
  background-color: #c82333;
}

.edit-button {
  background-color: #ffc107;
}

.edit-button:hover {
  background-color: #e0a800;
}

.article-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.article-item {
  padding: 15px;
  margin-bottom: 10px;
  background-color: #fff;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  align-items: center;
}

.article-header {
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 5px;
  color: #333;
}

.article-content {
  font-size: 16px;
  margin-bottom: 10px;
  color: #666;
  text-align: center;
}

.article-buttons {
  display: flex;
  gap: 10px;
  justify-content: center;
}
</style>
