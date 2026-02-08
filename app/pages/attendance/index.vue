<script setup>
import { ref, computed } from 'vue';

// 1. Inayos na Fetch: Nagdagdag tayo ng filter para tanging logs na may valid employee lang ang lalabas
const { data: attendance, refresh } = await useFetch('http://localhost:1337/api/attendances?populate=*&filters[employee][id][$notNull]=true')
const search = ref('');

const headers = [
  { title: 'Date', key: 'date' },
  { title: 'Employee Name', key: 'employeeName' },
  { title: 'Check In', key: 'checkIn' },
  { title: 'Check Out', key: 'checkOut' },
  { title: 'Status', key: 'statuss' },
  { title: 'Actions', key: 'actions', sortable: false }, // Bagong column para sa delete
];

// Function para mag-delete ng log entry
const deleteLog = async (id) => {
  if (confirm('Sigurado ka bang gusto mong burahin ang attendance log na ito?')) {
    try {
      await $fetch(`http://localhost:1337/api/attendances/${id}`, {
        method: 'DELETE',
      });
      refresh(); // I-refresh ang listahan pagkatapos burahin
    } catch (err) {
      alert('Error deleting log. Check Strapi permissions.');
    }
  }
};

const items = computed(() => {
  if (!attendance.value?.data) return [];

  return attendance.value.data.map((record) => {
    const emp = record.employee; 
    const fullname = emp ? `${emp.firstname} ${emp.lastname}`.trim() : 'No Employee Linked';

    return {
      id: record.documentId || record.id, // Gamitin ang documentId para sa Strapi v5
      date: record.date || '',
      employeeName: fullname, 
      checkIn: record.checkIn || record.checkin || '--:--',
      checkOut: record.checkOut || record.checkout || '--:--',
      statuss: record.statuss || record.status || 'PRESENT'
    };
  });
});
</script>

<template>
  <v-container>
    <div class="d-flex justify-space-between align-center mb-6">
      <h2 class="text-h4 font-weight-bold">
        <v-icon icon="mdi-clock-outline" class="me-2" color="success"></v-icon>
        Attendance Logs
      </h2>
      <v-btn color="success" prepend-icon="mdi-monitor" to="/attendance/terminal">
        Open Terminal
      </v-btn>
    </div>

    <v-card border elevation="0" class="rounded-lg">
      <v-card-title class="pa-4">
        <v-text-field
          v-model="search"
          prepend-inner-icon="mdi-magnify"
          label="Search logs..."
          variant="outlined"
          density="compact"
          hide-details
        ></v-text-field>
      </v-card-title>

      <v-data-table
        :headers="headers"
        :items="items"
        :search="search"
        hover
      >
        <template v-slot:item.checkIn="{ value }">
          <v-chip size="small" color="blue-lighten-4" variant="flat" class="font-weight-bold">
            {{ value }}
          </v-chip>
        </template>

        <template v-slot:item.checkOut="{ value }">
          <v-chip size="small" :color="value !== '--:--' ? 'orange-lighten-4' : 'grey-lighten-3'" variant="flat">
            {{ value }}
          </v-chip>
        </template>

        <template v-slot:item.statuss="{ value }">
          <v-chip 
            size="small" 
            :color="value === 'PRESENT' || value === 'ON TIME' ? 'success' : 'warning'" 
            variant="tonal" 
            class="text-uppercase"
          >
            {{ value }}
          </v-chip>
        </template>

        <template v-slot:item.actions="{ item }">
          <v-btn
            icon="mdi-delete-outline"
            size="small"
            variant="text"
            color="error"
            @click="deleteLog(item.id)"
          ></v-btn>
        </template>
      </v-data-table>
    </v-card>
  </v-container>
</template>