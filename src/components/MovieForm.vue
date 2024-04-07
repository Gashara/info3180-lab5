<template>
    <div>
      <form @submit.prevent="saveMovie" id="movieForm" enctype="multipart/form-data">
        <label for="title" class="movie-title">Movie Title</label><br>
        <input type="text" name="title" class="movie-input" /><br>
    
        <label for="description" class="movie-title">Movie Description</label><br>
        <input type="text" name="description" class="description-input" /><br>
    
        <label for="poster" class="movie-title">Movie Image</label><br>
        <input type="file" name="poster" id="image-file" class="upload-button" /><br><br>
    
        <input type="submit" value="Submit" class="submit-button">
      </form>
      
      <!-- Success message -->
      <div v-if="successMessage" class="success-message">
        {{ successMessage }}
      </div>
  
      <!-- Error messages -->
      <div v-if="errorMessage.length > 0" class="error-messages">
        <div v-for="error in errorMessage" :key="error">{{ error }}</div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from "vue";
  
  const csrf_token = ref("");
  const successMessage = ref("");
  const errorMessage = ref([]);
  
  onMounted(() => {
      getCsrfToken();
  });
  
  const saveMovie = () => {
      const movieForm = document.getElementById('movieForm');
      const form_data = new FormData(movieForm);
      
      fetch("/api/v1/movies", {
        method: 'POST',
        body: form_data,
        headers: {
          'X-CSRFToken': csrf_token.value
        }
      })
      .then(response => {
        if (response.ok) {
          return response.json().then(data => {
            // Display success message
            successMessage.value = data.message;
            errorMessage.value = []; // Clear any previous error messages
          });
        } else {
          return response.json().then(data => {
            // Display error messages
            errorMessage.value = data.errors;
            successMessage.value = ""; // Clear any previous success message
          });
        }
      })
      .catch(error => {
        console.error(error);
        errorMessage.value = ['An error occurred. Please try again.'];
        successMessage.value = ""; // Clear any previous success message
      });
  };
  
  const getCsrfToken = () => {
      fetch('/api/v1/csrf-token')
      .then(response => response.json())
      .then(data => {
        console.log(data);
        csrf_token.value = data.csrf_token;
      })
      .catch(error => {
        console.error(error);
        errorMessage.value = ['Failed to fetch CSRF token.'];
      });
  };
  </script>
  
  <style>
  /* Add CSS styles for success and error messages */
  .success-message {
      color: green;
  }
  
  .error-messages {
      color: red;
  }
  
  form {
      padding: 10px;
      margin: 20px;
  }
  
  .movie-title {
      font-size: 28px;
      font-weight: bold;
      margin-bottom: 10px;
  }
  
  .movie-input, .description-input {
      padding: 10px;
      font-size: 16px;
      border: 2px solid #ccc;
      border-radius: 5px;
      margin-bottom: 20px;
      width: 25%;
  }
  
  .description-input {
      width: 45%;
  }
  
  .upload-button {
      padding: 10px;
      background-color: DodgerBlue;
      color: white;
      border: none;
      border-radius: 5px;
      font-size: 16px;
      margin-right: 10px;
  }
  
  .submit-button {
      padding: 10px;
      background-color: #008CBA;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      font-size: 16px;
  }
  </style>
  