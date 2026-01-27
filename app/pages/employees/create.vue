<script setup>
const router = useRouter()
const isSaving = ref(false)

// Form data base sa iyong design: First Name, Last Name, Email, Department
const form = ref({
  firstname: '',
  lastname: '',
  email: '',
  position: '',
  department: null
})

// Kunin ang listahan ng Departments mula sa Strapi para sa dropdown
const { data: departments } = await useFetch('http://localhost:1337/api/departments')

const saveEmployee = async () => {
  await $fetch('http://localhost:1337/api/employees', {
    method: 'POST',
    body: { 
      data: {
        firstname: form.value.firstname,
        lastname: form.value.lastname,
        email: form.value.email,
        position: form.value.position, // Siguraduhing may position field ka rin
        department: form.value.department
      } 
    }
  })
  router.push('/employees')
}
</script>

<template>
  <v-container class="fill-height d-flex justify-center align-center">
    <v-card width="600" color="#1E1E1E" theme="dark" class="pa-6 rounded-lg">
      <v-card-title class="text-h5 mb-4">Add New Employee</v-card-title>
      
      <v-form @submit.prevent="saveEmployee">
        <v-text-field 
          v-model="form.firstname" 
          label="First Name" 
          variant="filled" 
          class="mb-2"
        ></v-text-field>

        <v-text-field 
          v-model="form.lastname" 
          label="Last Name" 
          variant="filled" 
          class="mb-2"
        ></v-text-field>

        <v-text-field 
          v-model="form.email" 
          label="Email" 
          variant="filled" 
          class="mb-2"
        ></v-text-field>

        <v-select
          v-model="form.position"
          :items="position?.data || []"
          item-title="attributes.position"
          item-value="id"
          label="Select Position"
          variant="filled"
          class="mb-6"
        ></v-select>

        <v-btn 
          type="submit" 
          block 
          height="50" 
          color="grey-darken-3" 
          class="text-none"
          :loading="isSaving"
        >
          Save Employee
        </v-btn>
      </v-form>
    </v-card>
  </v-container>
</template>