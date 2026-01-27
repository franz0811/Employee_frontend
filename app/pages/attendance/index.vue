<script setup>
import { ref, onMounted } from 'vue';
import { useFetch } from '#app';

// Kukunin ang listahan ng lahat ng employees para sa dropdown
const { data: employees, pending: employeesPending, error: employeesError, refresh: refreshEmployees } = useFetch(
  'http://localhost:1337/api/employees?populate=*',
  {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
    },
  }
);

const selectedEmployee = ref(null);
const isLogging = ref(false);
const message = ref('');
const hasCheckedIn = ref(false); // Track check-in status

const handleCheckIn = async () => {
  if (!selectedEmployee.value) {
    message.value = 'Piliin muna ang iyong pangalan!';
    return;
  }

  isLogging.value = true;
  const now = new Date();
  const formattedDate = now.toISOString().split('T')[0]; // Extract date in YYYY-MM-DD format
  const formattedTime = now.toLocaleTimeString('en-US', { hour12: false }); // Extract time in HH:mm:ss format

  try {
    const { error: checkInError } = await $fetch('http://localhost:1337/api/attendances', {
      method: 'POST',
      body: {
        data: {
          date: formattedDate,
          checkIn: formattedTime,
          employee: selectedEmployee.value,
        },
      },
    });

    if (checkInError) {
      console.error('Error during check-in:', checkInError);
      message.value = `Check-in failed: ${checkInError.message || 'Unknown error'}`; // Provide a default
      return;
    }

    message.value = 'Check-in Successful!';
    hasCheckedIn.value = true;
    selectedEmployee.value = null;
    await refreshEmployees(); // Refresh employee list
  } catch (err) {
    console.error('Error during check-in:', err);
    message.value = 'Error: Siguraduhing may "create" permission ang Attendance sa Strapi.';
  } finally {
    isLogging.value = false;
  }
};

const handleCheckOut = async () => {
  if (!selectedEmployee.value) {
    message.value = 'Piliin muna ang iyong pangalan!';
    return;
  }

  isLogging.value = true;
  const now = new Date();
  const formattedDate = now.toISOString().split('T')[0]; // Extract date in YYYY-MM-DD format
  const formattedTime = now.toLocaleTimeString('en-US', { hour12: false }); // Extract time in HH:mm:ss format

  let attendanceRecords = null;
  let attendanceError = null;

  try {
    const response = await useFetch(
      () => `http://localhost:1337/api/attendances?filters[date][$eq]=${formattedDate}&filters[employee][$eq]=${selectedEmployee.value}&populate=*`,
      {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
        },
      }
    );

    if (response.error && response.error.value) {
      attendanceError = response.error.value;
      console.error('Error fetching attendance records:', attendanceError);
      message.value = `Error fetching attendance records: ${attendanceError.message || 'Unknown error'}`; // Provide a default
      return;
    }

    attendanceRecords = response.data;

    if (attendanceRecords && attendanceRecords.value && attendanceRecords.value.length > 0) {
      const attendanceRecordId = attendanceRecords.value[0].id;

      const { error: checkOutError } = await $fetch(`http://localhost:1337/api/attendances/${attendanceRecordId}`, {
        method: 'PUT',
        body: {
          data: {
            checkOut: formattedTime,
          },
        },
      });

      if (checkOutError) {
        console.error('Error during check-out:', checkOutError);
        message.value = `Check-out failed: ${checkOutError.message || 'Unknown error'}`; // Provide a default
        return;
      }

      message.value = 'Check-out Successful!';
      hasCheckedIn.value = false;
      selectedEmployee.value = null;
      await refreshEmployees(); // Refresh employee list
    } else {
      message.value = 'Error: No check-in record found for today.';
    }
  } catch (error) {
    console.error('Error during check-out:', error);
    message.value = 'Error: Could not perform check-out.';
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
    <v-card width="500" class="pa-10 rounded-xl" elevation="4">
      <v-card-title class="text-center text-h5 font-weight-bold mb-6">
        Attendance Terminal
      </v-card-title>

      <v-select
        v-model="selectedEmployee"
        label="Piliin ang iyong Pangalan"
        :items="employees?.data?.map(employee => ({
          id: employee.id,
          name: `${employee.firstname || ''} ${employee.lastname || ''}`
        })) || []"
        item-value="id"
        item-title="name"
        variant="outlined"
        :loading="employeesPending"
        :error="employeesError"
      >
      </v-select>

      <v-btn
        v-if="!hasCheckedIn"
        block
        size="x-large"
        color="success"
        @click="handleCheckIn"
        :loading="isLogging"
        class="font-weight-bold py-4 mt-2"
        style="height: 60px; font-size: 1.2rem;"
      >
        CONFIRM CHECK-IN
      </v-btn>

      <v-btn
        v-else
        block
        size="x-large"
        color="error"
        @click="handleCheckOut"
        :loading="isLogging"
        class="font-weight-bold py-4 mt-2"
        style="height: 60px; font-size: 1.2rem;"
      >
        CONFIRM CHECK-OUT
      </v-btn>

      <v-alert v-if="message" class="mt-4" :type="hasCheckedIn ? 'success' : 'info'" variant="tonal" closable>
        {{ message }}
      </v-alert>

      <v-alert v-if="employeesError" class="mt-4" type="error" variant="tonal" closable>
        {{ employeesError?.message || 'Failed to fetch employees.' }}
      </v-alert>
    </v-card>
  </v-container>
</template>