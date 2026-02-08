<script setup>
import { ref, computed, onMounted } from 'vue';
import { useFetch } from '#app';

// 1. Destructure refresh correctly from useFetch
const { data: employees, refresh, error, pending } = await useFetch(
  'http://localhost:1337/api/employees',
  {
    method: 'GET',
    server: false 
  }
);

const search = ref('');
const snackbar = ref({ show: false, text: '', color: '' });

const headers = [
  { title: 'First Name', key: 'firstname' },
  { title: 'Last Name', key: 'lastname' },
  { title: 'Position', key: 'position' },
  { title: 'Email', key: 'email' },
  { title: 'Actions', key: 'actions', sortable: false, align: 'end' },
];

// 2. Map both id and documentId (Strapi v5 requirement)
const items = computed(() => {
  return employees.value?.data?.map((employee) => {
    return {
      id: employee.id,
      documentId: employee.documentId, // Crucial for Strapi v5
      firstname: employee.firstname || '',
      lastname: employee.lastname || '',
      position: employee.position || '',
      email: employee.email || '',
    };
  }) || [];
});

const showMessage = (text, color = 'success') => {
  snackbar.value = { show: true, text, color };
};

// 3. Updated Delete Function using documentId
const deleteItem = async (item) => {
  // Use documentId if available, otherwise fallback to id
  const targetId = item.documentId || item.id;

  if (!confirm(`Are you sure you want to delete ${item.firstname}?`)) return;

  try {
    await $fetch(`http://localhost:1337/api/employees/${targetId}`, {
      method: 'DELETE',
    });
    
    showMessage('Employee deleted successfully!');
    
    // 4. Force a fresh fetch from the server
    await refresh(); 
  } catch (err) {
    console.error('Error deleting employee:', err);
    // Alert specific error message if Forbidden
    const errMsg = err.data?.error?.message || 'Failed to delete. Check Permissions.';
    showMessage(errMsg, 'error');
  }
};
</script>

<template>
  <v-container>
    <v-data-table
      :headers="headers"
      :items="items"
      :search="search"
      hover
      :loading="pending"
    >
      <template v-slot:top>
        <v-toolbar flat color="transparent">
          <v-toolbar-title class="text-h6 font-weight-bold">
            Employees List
          </v-toolbar-title>
          
          <v-spacer></v-spacer>

          <v-text-field
            v-model="search"
            prepend-inner-icon="mdi-magnify"
            label="Search Employees"
            single-line
            hide-details
            variant="outlined"
            density="compact"
            class="me-4"
            style="max-width: 300px;"
          ></v-text-field>

          <v-btn
            color="success"
            prepend-icon="mdi-plus"
            variant="elevated"
            @click="$router.push('/employees/create')"
          >
            Add Employee
          </v-btn>
        </v-toolbar>
        <v-divider></v-divider>
      </template>

      <template v-slot:item.actions="{ item }">
        <v-btn
          icon="mdi-pencil"
          variant="text"
          color="blue"
          size="small"
          class="me-2"
          @click="$router.push(`/employees/edit/${item.documentId || item.id}`)"
        ></v-btn>
        
        <v-btn
          icon="mdi-delete"
          variant="text"
          color="red"
          size="small"
          @click="deleteItem(item)" 
        ></v-btn>
      </template>
    </v-data-table>

    <v-snackbar v-model="snackbar.show" :color="snackbar.color">
      {{ snackbar.text }}
    </v-snackbar>
  </v-container>
</template>