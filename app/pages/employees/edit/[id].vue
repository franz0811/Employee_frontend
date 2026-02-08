<script setup>
import { ref, watchEffect } from 'vue'

const route = useRoute()
const docId = route.params.id // Kunin ang documentId mula sa URL

// 1. Kunin ang kasalukuyang data ng employee
const { data: employee, pending } = await useFetch(`http://localhost:1337/api/employees/${docId}`)

// 2. I-setup ang form fields kasama ang Position at Email
const form = ref({
  firstname: '',
  lastname: '',
  position: '',
  email: ''
})

// 3. I-populate ang form kapag may laman na ang 'employee' data
watchEffect(() => {
  if (employee.value?.data) {
    const data = employee.value.data
    form.value.firstname = data.firstname || ''
    form.value.lastname = data.lastname || ''
    form.value.position = data.position || ''
    form.value.email = data.email || ''
  }
})

const handleUpdate = async () => {
  try {
    // Magpadala ng PUT request sa Strapi gamit ang documentId
    await $fetch(`http://localhost:1337/api/employees/${docId}`, {
      method: 'PUT',
      body: { data: form.value }
    })
    alert('Employee updated successfully!')
    navigateTo('/employees')
  } catch (err) {
    console.error('Update failed:', err)
    alert('Update failed. Check Strapi permissions for "update".')
  }
}
</script>

<template>
  <v-container>
    <v-btn to="/employees" prepend-icon="mdi-arrow-left" variant="text" class="mb-4">Back to List</v-btn>

    <v-card class="pa-8 rounded-xl mx-auto" max-width="600" elevation="4">
      <v-card-title class="text-h5 font-weight-bold mb-4">
        <v-icon start color="primary">mdi-account-edit</v-icon>
        Edit Employee Profile
      </v-card-title>

      <v-progress-linear v-if="pending" indeterminate color="primary" class="mb-4"></v-progress-linear>

      <v-form v-else @submit.prevent="handleUpdate">
        <v-text-field 
          v-model="form.firstname" 
          label="First Name" 
          variant="outlined" 
          class="mb-2"
        ></v-text-field>

        <v-text-field 
          v-model="form.lastname" 
          label="Last Name" 
          variant="outlined" 
          class="mb-2"
        ></v-text-field>

        <v-text-field 
          v-model="form.position" 
          label="Position" 
          variant="outlined" 
          class="mb-2"
        ></v-text-field>

        <v-text-field 
          v-model="form.email" 
          label="Email Address (Gmail)" 
          variant="outlined" 
          type="email"
          class="mb-4"
        ></v-text-field>

        <v-btn 
          block 
          color="primary" 
          size="large" 
          type="submit" 
          class="font-weight-bold"
        >
          SAVE CHANGES
        </v-btn>
      </v-form>
    </v-card>
  </v-container>
</template>