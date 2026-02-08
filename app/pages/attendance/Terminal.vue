<script setup>
import { ref, onMounted } from 'vue';
import { useFetch } from '#app';

// 1. Load Employees
const { data: employees, pending: employeesPending, refresh: refreshEmployees } = useFetch(
  'http://localhost:1337/api/employees',
  {
    method: 'GET',
    headers: { 'Content-Type': 'application/json' },
  }
);

const selectedEmployee = ref(null);
const isLogging = ref(false);
const message = ref('');
const hasCheckedIn = ref(false); 

// 2. Handle Check-In
const handleCheckIn = async () => {
  if (!selectedEmployee.value) {
    message.value = 'Piliin muna ang iyong pangalan!';
    return;
  }

  isLogging.value = true;
  const now = new Date();
  const formattedDate = now.toISOString().split('T')[0]; 
  // TIME FORMAT FIX: Ginawa nating standard HH:mm:ss.000 para sa Strapi
  const formattedTime = now.toTimeString().split(' ')[0] + ".000"; 

  try {
    await $fetch('http://localhost:1337/api/attendances', {
      method: 'POST',
      body: {
        data: {
          date: formattedDate,
          checkIn: formattedTime,
          employee: selectedEmployee.value, 
          statuss: 'Present' // Sinunod natin ang valid enum value
        },
      },
    });

    message.value = 'Check-in Successful!';
    hasCheckedIn.value = true;
    selectedEmployee.value = null;
  } catch (err) {
    console.error('Error details:', err.data);
    const detail = err.data?.error?.message || 'Check-in Failed.';
    message.value = `${detail}. Siguraduhin na ang API permissions ay naka-save.`;
  } finally {
    isLogging.value = false;
  }
};

// 3. Handle Check-Out
const handleCheckOut = async () => {
  if (!selectedEmployee.value) {
    message.value = 'Piliin muna ang iyong pangalan!';
    return;
  }

  isLogging.value = true;
  const now = new Date();
  const formattedDate = now.toISOString().split('T')[0];
  const formattedTime = now.toTimeString().split(' ')[0] + ".000";

  try {
    const response = await $fetch(
      `http://localhost:1337/api/attendances?filters[date][$eq]=${formattedDate}&filters[employee][documentId][$eq]=${selectedEmployee.value}`
    );

    if (response.data && response.data.length > 0) {
      const docId = response.data[0].documentId;

      await $fetch(`http://localhost:1337/api/attendances/${docId}`, {
        method: 'PUT',
        body: {
          data: { checkOut: formattedTime },
        },
      });

      message.value = 'Check-out Successful!';
      hasCheckedIn.value = false;
      selectedEmployee.value = null;
    } else {
      message.value = 'No record found today. Mag-check-in ka muna.';
    }
  } catch (error) {
    console.error('Check-out error:', error);
    message.value = 'Check-out Failed.';
  } finally {
    isLogging.value = false;
  }
};

onMounted(() => {
  refreshEmployees();
});
</script>

<template>
  <v-container class="fill-height d-flex justify-center align-center">
    <v-card width="550" class="pa-8 rounded-xl" elevation="10">
      <v-card-title class="text-center text-h5 font-weight-bold mb-6">
        Attendance Terminal
      </v-card-title>

      <v-select
        v-model="selectedEmployee"
        label="Select Your Name"
        :items="employees?.data?.map(emp => ({
          docId: emp.documentId, 
          name: `${emp.firstname} ${emp.lastname}`
        })) || []"
        item-value="docId" 
        item-title="name"
        variant="outlined"
      ></v-select>

      <v-row class="mt-2">
        <v-col cols="6">
          <v-btn
            block
            size="x-large"
            color="success"
            @click="handleCheckIn"
            :loading="isLogging"
            style="height: 80px;"
            elevation="4"
          >
            <v-icon start>mdi-login</v-icon>
            CHECK-IN
          </v-btn>
        </v-col>

        <v-col cols="6">
          <v-btn
            block
            size="x-large"
            color="error"
            @click="handleCheckOut"
            :loading="isLogging"
            style="height: 80px;"
            elevation="4"
          >
            <v-icon start>mdi-logout</v-icon>
            CHECK-OUT
          </v-btn>
        </v-col>
      </v-row>

      <v-alert v-if="message" class="mt-6" :type="message.includes('Successful') ? 'success' : 'error'" variant="tonal">
        {{ message }}
      </v-alert>

      <v-btn to="/attendance" variant="text" block class="mt-4" prepend-icon="mdi-arrow-left">
        View Logs
      </v-btn>
    </v-card>
  </v-container>
</template>