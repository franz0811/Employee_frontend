<template>
  <v-container fluid>
    <ClientOnly>
      <v-row justify="center">
        <v-col cols="12" md="8">
          <v-card class="pa-6 rounded-lg" elevation="3">
            <v-card-title class="text-h5 font-weight-bold mb-4 text-center">
              File a Leave Request
            </v-card-title>

            <v-form @submit.prevent="submitLeave">
              <v-select
                v-model="selectedEmployee"
                label="Piliin ang iyong Pangalan"
                :items="employees?.data || []"
                item-value="id"
                variant="outlined"
                :loading="pending"
                :clearable="true"
              >
                <template v-slot:item="{ props, item }">
                  <v-list-item
                    v-bind="props"
                    :title="`${item.raw?.attributes?.firstname || ''} ${item.raw?.attributes?.lastname || ''}`"
                  ></v-list-item>
                </template>
                <template v-slot:selection="{ item }">
                  {{ item.raw?.attributes?.firstname }} {{ item.raw?.attributes?.lastname }}
                </template>
              </v-select>

              <v-select
                v-model="leaveType"
                label="Leave Type"
                :items="['Sick Leave', 'Vacation Leave', 'Emergency Leave']"
                variant="outlined"
                class="mt-4"
                :clearable="true"
              ></v-select>

              <v-text-field
                v-model="reason"
                label="Reason"
                variant="outlined"
                class="mt-4"
              ></v-text-field>

              <v-menu
                ref="startDateMenu"
                v-model="startDateMenuOpen"
                :close-on-content-click="false"
                transition="scale-transition"
                offset-y
                min-width="auto"
              >
                <template v-slot:activator="{ on, attrs }">
                  <v-text-field
                    v-model="startDate"
                    label="Start Date"
                    prepend-icon="mdi-calendar"
                    readonly
                    v-bind="attrs"
                    v-on="on"
                    variant="outlined"
                    class="mt-4"
                  ></v-text-field>
                </template>
                <v-date-picker
                  v-model="startDate"
                  @input="startDateMenuOpen = false"
                ></v-date-picker>
              </v-menu>

              <v-menu
                ref="endDateMenu"
                v-model="endDateMenuOpen"
                :close-on-content-click="false"
                transition="scale-transition"
                offset-y
                min-width="auto"
              >
                <template v-slot:activator="{ on, attrs }">
                  <v-text-field
                    v-model="endDate"
                    label="End Date"
                    prepend-icon="mdi-calendar"
                    readonly
                    v-bind="attrs"
                    v-on="on"
                    variant="outlined"
                    class="mt-4"
                  ></v-text-field>
                </template>
                <v-date-picker
                  v-model="endDate"
                  @input="endDateMenuOpen = false"
                ></v-date-picker>
              </v-menu>

              <v-btn
                type="submit"
                color="orange-darken-2"
                block
                size="large"
                :loading="isSending"
                class="mt-6"
              >
                SUBMIT REQUEST
              </v-btn>

              <!-- Snackbar for notifications -->
              <v-snackbar
                v-model="snackbar.show"
                :color="snackbar.color"
                timeout="3000"
              >
                {{ snackbar.text }}
              </v-snackbar>
            </v-form>
          </v-card>
        </v-col>
      </v-row>
    </ClientOnly>
  </v-container>
</template>

<script setup>
import { ref } from 'vue';
// Pinilit nating maging CLIENT-SIDE ONLY ang fetch para iwas Hydration Mismatch
const { data: employees, pending } = await useAsyncData(
  'employees',
  () => $fetch('http://localhost:1337/api/employees'),
  { server: false, default: () => ({ data: [] }) } // Provide a default
);

const selectedEmployee = ref(null);
const leaveType = ref(null);
const reason = ref(''); // Add reason ref
const startDate = ref(new Date().toISOString().substr(0, 10)); // Add start date ref
const endDate = ref(new Date().toISOString().substr(0, 10)); // Add end date ref
const isSending = ref(false);
const startDateMenuOpen = ref(false);
const endDateMenuOpen = ref(false);

// Reactive snackbar properties
const snackbar = ref({
  show: false,
  text: '',
  color: '',
});

const showSnackbar = (text, color) => {
  snackbar.value.show = true;
  snackbar.value.text = text;
  snackbar.value.color = color;
};

const submitLeave = async () => {
  if (!selectedEmployee.value || !leaveType.value || !reason.value || !startDate.value || !endDate.value) {
    return showSnackbar('Paki-pili ang lahat ng field.', 'warning');
  }

  isSending.value = true;
  try {
    const response = await $fetch('http://localhost:1337/api/leave-requests', {
      method: 'POST',
      body: {
        data: {
          employee: selectedEmployee.value, // ID ito ng employee
          startDate: startDate.value,
          endDate: endDate.value,
          reason: reason.value, // Use reason.value
          statuss: 'Pending', // Palitan ang 'pending' sa 'Pending'
        },
      },
    });

    if (response.error) {
      console.error('Submission Error Details:', response.error);
      return showSnackbar(`Error: ${response.error.message}`, 'error');
    }

    showSnackbar('Tagumpay! Na-save na ang iyong Leave Request.', 'success');
    selectedEmployee.value = null;
    leaveType.value = null;
    reason.value = '';
    startDate.value = new Date().toISOString().substr(0, 10);
    endDate.value = new Date().toISOString().substr(0, 10);
  } catch (error) {
    console.error('Submission Error Details:', error);
    // Check if the error has a response and data
    if (error.response && error.response._data) {
      const strapiError = error.response._data.error;
      showSnackbar(`Error: ${strapiError.message} - ${strapiError.details}`, 'error');
    } else {
      showSnackbar('Error: May mali sa format ng data o permissions sa Strapi.', 'error');
    }
  } finally {
    isSending.value = false;
  }
};
</script>
