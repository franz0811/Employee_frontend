<template>
  <v-container fluid class="d-flex align-center justify-center" style="height: 100vh;">
    <v-card class="mx-auto pa-12 pb-8" elevation="2" rounded="lg" width="450">
      <v-img
        class="mx-auto mb-6"
        max-width="150"
        src="https://png.pngtree.com/png-clipart/20250128/original/pngtree-human-resources-specialist-3d-icon-isolated-on-a-white-background-symbolizing-png-image_20173414.png"
      ></v-img>
      
      <h1 class="text-h5 text-center font-weight-bold pb-6">Employee Login</h1>

      <v-text-field
        v-model="email"
        label="Email address"
        prepend-inner-icon="mdi-email-outline"
        variant="outlined"
        density="compact"
      ></v-text-field>

      <v-text-field
        v-model="password"
        label="Password"
        :type="visible ? 'text' : 'password'"
        :append-inner-icon="visible ? 'mdi-eye-off' : 'mdi-eye'"
        @click:append-inner="visible = !visible"
        prepend-inner-icon="mdi-lock-outline"
        variant="outlined"
        density="compact"
      ></v-text-field>

      <v-alert
        v-if="errorMsg"
        type="error"
        variant="tonal"
        density="compact"
        class="mt-2 mb-4"
      >
        {{ errorMsg }}
      </v-alert>

      <v-btn
        block
        color="primary"
        size="large"
        variant="tonal"
        @click="handleLogin"
        :loading="loading"
      >
        Log In
      </v-btn>
    </v-card>
  </v-container>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

definePageMeta({ layout: false })

const router = useRouter()
const email = ref('')
const password = ref('')
const visible = ref(false)
const loading = ref(false)
const errorMsg = ref('')

const handleLogin = async () => {
  if (!email.value || !password.value) {
    errorMsg.value = "Pakilagay ang email at password."
    return
  }
  loading.value = true
  errorMsg.value = ""
  try {
    const response = await $fetch('http://localhost:1337/api/auth/local', {
      method: 'POST',
      body: { identifier: email.value, password: password.value }
    })
    if (response.jwt) {
      localStorage.setItem('auth_token', response.jwt)
      router.push('/dashboard')
    }
  } catch (err) {
    errorMsg.value = "Maling email o password."
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
/* SIGURADUHIN NA EMPTY ITO O WALANG NAKASULAT NA errorMsg DITO */
</style>