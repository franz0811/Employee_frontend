<script setup>
import { ref, onMounted } from 'vue';

// 1. Fetch Employees mula sa Strapi
const { data: employees, error } = await useFetch('http://localhost:1337/api/employees');

// Debugger para makita sa F12 > Console kung ano ang dumating na data
onMounted(() => {
  if (employees.value) {
    console.log("Employees Data from Strapi:", employees.value.data);
  }
  if (error.value) {
    console.error("Fetch Error:", error.value);
  }
});

const selectedEmployee = ref(null);
const startDate = ref(new Date().toISOString().split('T')[0]);
const endDate = ref(new Date().toISOString().split('T')[0]);
const leaveReason = ref('');
const isSubmitting = ref(false);
const submissionMsg = ref('');

// 2. Submit Leave Request
const handleLeaveSubmit = async () => {
  if (!selectedEmployee.value || !leaveReason.value) {
    submissionMsg.value = "Pakikumpleto ang lahat ng fields.";
    return;
  }

  isSubmitting.value = true;
  submissionMsg.value = "";

  try {
    await $fetch('http://localhost:1337/api/leave-requests', {
      method: 'POST',
      body: {
        data: {
          employee: selectedEmployee.value, // Match sa 'employee' field mo
          startDate: startDate.value,
          endDate: endDate.value,
          reason: leaveReason.value,
          statuss: 'Pending' // Tugma sa 'statuss' field sa Strapi schema mo
        }
      }
    });
    submissionMsg.value = "Leave request submitted successfully!";
    selectedEmployee.value = null;
    leaveReason.value = '';
  } catch (err) {
    submissionMsg.value = "Error: Siguraduhing naka-SAVE ang Public 'create' permission.";
    console.error(err);
  } finally {
    isSubmitting.value = false;
  }
};
</script>

<template>
  <v-container fluid class="d-flex align-center justify-center" style="min-height: 100vh;">
    <v-card width="500" class="pa-8 rounded-xl" elevation="10">
      <v-card-title class="text-center text-h5 font-weight-bold mb-6">
        File a Leave Request
      </v-card-title>

      <v-select
        v-model="selectedEmployee"
        label="Piliin ang iyong Pangalan"
        :items="employees?.data?.map(emp => ({
          title: `${emp.firstname} ${emp.lastname}`,
          value: emp.documentId 
        })) || []"
        item-title="title"
        item-value="value"
        variant="outlined"
        class="mb-2"
        :loading="!employees && !error"
        :placeholder="employees?.data?.length === 0 ? 'Walang mahanap na Published Employee' : 'Pumili sa listahan'"
        no-data-text="Walang Employee na nahanap sa database"
      ></v-select>

      <v-row>
        <v-col cols="6">
          <v-text-field v-model="startDate" label="Start Date" type="date" variant="outlined"></v-text-field>
        </v-col>
        <v-col cols="6">
          <v-text-field v-model="endDate" label="End Date" type="date" variant="outlined"></v-text-field>
        </v-col>
      </v-row>

      <v-textarea 
        v-model="leaveReason" 
        label="Reason" 
        variant="outlined" 
        rows="3"
        placeholder="Bakit ka mag-le-leave?"
      ></v-textarea>

      <v-btn 
        block 
        size="x-large" 
        color="primary" 
        @click="handleLeaveSubmit" 
        :loading="isSubmitting" 
        class="mt-2"
      >
        SUBMIT REQUEST
      </v-btn>

      <v-btn 
        block 
        variant="outlined" 
        color="secondary" 
        class="mt-4" 
        to="/leavelist"
        prepend-icon="mdi-format-list-bulleted"
      >
        VIEW LEAVE STATUS
      </v-btn>

      <v-alert 
        v-if="submissionMsg" 
        class="mt-6" 
        :type="submissionMsg.includes('successfully') ? 'success' : 'error'" 
        variant="tonal"
      >
        {{ submissionMsg }}
      </v-alert>
    </v-card>
  </v-container>
</template>

<style scoped>
</style>