<script setup>
import { ref } from 'vue'
import api from '../lib/axios'

const email = ref('')
const password = ref('')
const user = ref(null)
const errorMessage = ref('')

const handleLogin = async () => {
  try {
    errorMessage.value = ''
    
    // 1. Request the CSRF cookie from Laravel Sanctum
    await api.get('/sanctum/csrf-cookie')
    
    // 2. Perform the login request via the proxy route
    await api.post('/login', {
      email: email.value,
      password: password.value
    })
    
    // 3. Fetch the authenticated user data to confirm success
    const response = await api.get('/api/user')
    user.value = response.data
  } catch (error) {
    if (error.response && error.response.data.message) {
      errorMessage.value = error.response.data.message
    } else {
      errorMessage.value = 'An error occurred during authentication.'
    }
  }
}
</script>

<template>
  <div class="login-container">
    <h2>Laravel Sanctum Auth Test</h2>
    
    <div v-if="user">
      <p>Success! Welcome, <strong>{{ user.name }}</strong> ({{ user.email }})</p>
    </div>
    
    <form v-else @submit.prevent="handleLogin">
      <div>
        <label>Email:</label>
        <input type="email" v-model="email" required />
      </div>
      <div>
        <label>Password:</label>
        <input type="password" v-model="password" required />
      </div>
      <button type="submit">Login</button>
      <p v-if="errorMessage" style="color: red;">{{ errorMessage }}</p>
    </form>
  </div>
</template>