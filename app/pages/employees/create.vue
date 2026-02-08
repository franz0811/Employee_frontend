<script setup>
import { ref } from 'vue';
const router = useRouter()
const isSaving = ref(false)
const isDeleting = ref(false) // State for delete loading

const form = ref({
  firstname: '',
  lastname: '',
  email: '',
  position: '', 
})

// --- SAVE LOGIC ---
const saveEmployee = async () => {
  if (!form.value.firstname || !form.value.lastname || !form.value.email) {
    alert("Please fill in all required fields.");
    return;
  }

  isSaving.value = true;
  try {
    await $fetch('http://localhost:1337/api/employees', {
      method: 'POST',
      body: { 
        data: {
          firstname: form.value.firstname,
          lastname: form.value.lastname,
          email: form.value.email,
          position: form.value.position,
        } 
      }
    });
    router.push('/employees');
  } catch (error) {
    const errorMessage = error.data?.error?.message || "Unknown error occurred";
    alert(`Failed to save: ${errorMessage}`);
  } finally {
    isSaving.value = false;
  }
}

// --- DELETE LOGIC ---
// Note: You need the 'id' of the employee to delete them
const deleteEmployee = async (id) => {
  if (!id) return;
  
  if (confirm("Are you sure you want to delete this employee?")) {
    isDeleting.value = true;
    try {
      await $fetch(`http://localhost:1337/api/employees/${id}`, {
        method: 'DELETE',
      });
      alert("Employee deleted successfully!");
      router.push('/employees');
    } catch (error) {
      console.error("Delete Error:", error);
      alert("Failed to delete. Check if you have 'delete' permissions in Strapi.");
    } finally {
      isDeleting.value = false;
    }
  }
}
</script>

<template>
  <v-container class="fill-height d-flex justify-center align-center">
    <v-card width="600" color="#1E1E1E" theme="dark" class="pa-6 rounded-lg">
      <v-card-title class="text-h5 mb-4">Employee Details</v-card-title>
      
      <v-form @submit.prevent="saveEmployee">
        <v-text-field 
          v-model="form.firstname" 
          label="First Name" 
          variant="filled" 
          class="mb-2"
          required
        ></v-text-field>

        <v-text-field 
          v-model="form.lastname" 
          label="Last Name" 
          variant="filled" 
          class="mb-2"
          required
        ></v-text-field>

        <v-text-field 
          v-model="form.email" 
          label="Email" 
          variant="filled" 
          class="mb-2"
          type="email"
          required
        ></v-text-field>

        <v-text-field
          v-model="form.position"
          label="Position (e.g. Developer, Manager)"
          variant="filled"
          class="mb-6"
        ></v-text-field>

        <v-btn 
          type="submit" 
          block 
          height="50" 
          color="success" 
          class="text-none mb-3"
          :loading="isSaving"
        >
          Save Employee
        </v-btn>

        <v-btn 
          v-if="router.currentRoute.value.params.id"
          block 
          height="50" 
          color="error" 
          variant="outlined"
          class="text-none"
          :loading="isDeleting"
          @click="deleteEmployee(router.currentRoute.value.params.id)"
        >
          Delete Employee
        </v-btn>
      </v-form>
    </v-card>
  </v-container>
</template>