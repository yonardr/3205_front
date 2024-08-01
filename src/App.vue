<template>
  <div class="min-h-screen bg-gray-100 flex items-center justify-center p-6">
    <div class="bg-white p-8 rounded-lg shadow-lg max-w-md w-full">
      <h1 class="text-2xl font-bold mb-6 text-gray-700">User Search</h1>
      <form @submit.prevent="submitForm" class="space-y-4">
        <div>
          <label for="email" class="block text-gray-600 font-medium">Email:</label>
          <input
              type="email"
              v-model="email"
              required
              class="w-full mt-1 p-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
        </div>
        <div>
          <label for="number" class="block text-gray-600 font-medium">Number:</label>
          <input
              v-mask="'##-##-##'"
              v-model="number"
              class="w-full mt-1 p-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
        </div>
        <button
            type="submit"
            class="w-full py-2 px-4 bg-blue-600 text-white font-semibold rounded-lg shadow-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500"
        >
          Submit
        </button>
      </form>
      <div v-if="loading" class="mt-4 text-gray-600">Loading...</div>
      <div v-if="error" class="mt-4 text-red-600">{{ error }}</div>
      <ul v-if="searchPerformed && users.length > 0" class="mt-4 space-y-2">
        <li
            v-for="user in users"
            :key="user.email + user.number"
            class="p-4 bg-gray-50 border border-gray-200 rounded-lg shadow-sm"
        >
          {{ user.email }} - {{ user.number }}
        </li>
      </ul>
      <div v-else-if="searchPerformed && !loading && !error && users.length === 0" class="mt-4 text-gray-600">No users found</div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import { ref } from 'vue';

export default {
  setup() {
    const email = ref('');
    const number = ref('');
    const users = ref([]);
    const loading = ref(false);
    const error = ref('');
    const searchPerformed = ref(false);
    let cancelTokenSource;

    const submitForm = async () => {
      if (cancelTokenSource) {
        cancelTokenSource.cancel("Operation canceled due to new request.");
      }

      users.value = [];
      loading.value = true;
      error.value = '';
      searchPerformed.value = false;
      cancelTokenSource = axios.CancelToken.source();

      try {
        const response = await axios.get('http://localhost:3000/users', {
          params: {
            email: email.value,
            number: number.value.replace(/-/g, ''),
          },
          cancelToken: cancelTokenSource.token,
        });
        users.value = response.data;
      } catch (err) {
        if (axios.isCancel(err)) {
          console.log('Request canceled:', err.message);
          error.value = err.message;
        } else if (err.response) {
          error.value = err.response.data.message;
        } else {
          console.error(err);
        }
      } finally {
        loading.value = false;
        cancelTokenSource = null;
        searchPerformed.value = true;
      }
    };

    return {
      email,
      number,
      users,
      loading,
      error,
      searchPerformed,
      submitForm,
    };
  }
};
</script>

<style>
/* Добавьте стили по желанию */
</style>
