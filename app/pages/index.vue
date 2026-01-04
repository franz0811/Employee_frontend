<template>
  <v-container fluid class="d-flex align-center justify-center" style="height: 100vh;">
    <v-card
      class="mx-auto pa-12 pb-8"
      elevation="2"
      rounded="lg"
    >
    <v-img
      class="ma-2"
      max-width="228"
      src="https://cdn.vuetifyjs.com/docs/images/logos/vuetify-logo-v3-slim-text-light.svg"
    ></v-img>
      <h1 class="text-medium-emphasis pb-6">Inventory Login</h1>

      <v-text-field
        density="compact"
        placeholder="Email address"
        prepend-inner-icon="mdi-email-outline"
        variant="outlined"
      ></v-text-field>  

      <v-text-field
        :append-inner-icon="visible ? 'mdi-eye-off' : 'mdi-eye'"
        :type="visible ? 'text' : 'password'"
        density="compact"
        placeholder="Enter your password"
        prepend-inner-icon="mdi-lock-outline"
        variant="outlined"
        @click:append-inner="visible = !visible"
      ></v-text-field>

      <div class="text-subtitle-1 text-medium-emphasis">

        <a
          class="text-caption text-decoration-none text-blue mt-0 pt-0"
          href="#"
          rel="noopener noreferrer"
          target="_blank"
        >
          Forgot login password?</a>
      </div>

      <v-btn
        class="mb-8"
        color="primary"
        size="large"
        variant="tonal"
        block
        to="/dashboard"
      >
        Log In
      </v-btn>

      <v-card-text class="text-center">
        <a
          class="text-blue text-decoration-none"
          href="#"
          rel="noopener noreferrer"
          target="_blank"
        >
          Sign up now <v-icon icon="mdi-chevron-right"></v-icon>
        </a>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script setup>
  definePageMeta({
    layout: false,
  })
  const visible = ref(false)

</script>

<style></style>
