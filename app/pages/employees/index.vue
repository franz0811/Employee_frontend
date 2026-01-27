<script setup>
import { ref, computed, onMounted } from 'vue';
import { useFetch } from '#app';

// Kukunin ang listahan mula sa Strapi (Section 2.4 - Retrieve)
const { data: employees, refresh, error, pending } = useFetch(
  'http://localhost:1337/api/employees?populate=*',
  {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
    },
  }
);

const headers = [
  { title: 'First Name', key: 'firstname' },
  { title: 'Last Name', key: 'lastname' },
  { title: 'Position', key: 'position' },
  { title: 'Email', key: 'email' },
];

const items = computed(() => {
  console.log('Employees data:', employees.value); // Debugging: Check the raw data

  return employees.value?.data?.map((employee) => {
      console.log('Processing employee:', employee); // Debugging: Check each employee object

      const { firstname = '', lastname = '', position = '', email = '' } = employee || {};
      return {
        id: employee.id,
        firstname,
        lastname,
        position,
        email,
      };
    }) || [];
});

onMounted(() => {
  refresh();
});

const deleteItem = async (itemId) => {
  try {
    await $fetch(`http://localhost:1337/api/employees/${itemId}`, {
      method: 'DELETE',
    });
    message.value = 'Employee deleted successfully!';
    refresh(); // Refresh the employee list
  } catch (err) {
    console.error('Error deleting employee:', err);
    message.value = 'Failed to delete employee.';
  }
};

const message = ref(''); // For displaying messages
</script>

<template>
  <v-container>
    <div class="d-flex justify-space-between align-center mb-6">
      <h2 class="text-h4">Employee Directory</h2>
      <v-btn color="primary" prepend-icon="mdi-plus" to="/employees/create">
        Add Employee
      </v-btn>
    </div>

    <v-card border elevation="0">
      <v-data-table
        :headers="headers"
        :items="items"
        hover
        :loading="pending"
      >
        <template v-slot:loading>
          <v-progress-linear indeterminate color="primary"></v-progress-linear>
        </template>
        <template v-slot:no-data>
          <v-alert type="warning" title="No data available."></v-alert>
        </template>
        <template v-slot:error>
          <v-alert type="error" title="An error occurred while loading data."></v-alert>
        </template>
        <template v-slot:item.actions="{ item }">
          <v-icon
            size="small"
            class="me-2"
            @click="$router.push(`/employees/edit/${item.id}`)"
          >
            mdi-pencil
          </v-icon>
          <v-icon
            size="small"
            @click="deleteItem(item.id)"
          >
            mdi-delete
          </v-icon>
        </template>
      </v-data-table>
    </v-card>

    <v-alert v-if="error" type="error">
      {{ error.message }}
    </v-alert>
    <v-alert v-if="message" type="success">
      {{ message }}
    </v-alert>
  </v-container>
</template>