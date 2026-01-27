<script setup>
// Kukunin ang bilang ng Employees (Section 2.4 - Retrieve)
const { data: employees } = await useFetch('http://localhost:1337/api/employees')

// Kukunin ang Attendance logs para sa araw na ito (Section 2.4 - Retrieve)
const today = new Date().toISOString().split('T')[0]
const { data: attendance } = await useFetch(`http://localhost:1337/api/attendances?filters[date][$eq]=${today}`)

// Kukunin ang bilang ng Pending Leave Requests (Section 2.4 - Retrieve)
const { data: leaves } = await useFetch('http://localhost:1337/api/leave-requests?filters[statuss][$eq]=Pending')
</script>

<template>
  <v-container fluid>
    <v-row>
      <v-col cols="12" md="4">
        <v-card border elevation="0" class="rounded-lg pa-4">
          <div class="d-flex align-center justify-space-between">
            <div>
              <div class="text-overline mb-1">Total Workforce</div>
              <div class="text-h3 font-weight-bold">{{ employees?.meta?.pagination?.total || 0 }}</div>
            </div>
            <v-icon icon="mdi-account-group" color="primary" size="x-large"></v-icon>
          </div>
        </v-card>
      </v-col>

      <v-col cols="12" md="4">
        <v-card border elevation="0" class="rounded-lg pa-4">
          <div class="d-flex align-center justify-space-between">
            <div>
              <div class="text-overline mb-1">Today's Attendance</div>
              <div class="text-h3 font-weight-bold">{{ attendance?.meta?.pagination?.total || 0 }}</div>
            </div>
            <v-icon icon="mdi-clock-check" color="success" size="x-large"></v-icon>
          </div>
        </v-card>
      </v-col>

      <v-col cols="12" md="4">
        <v-card border elevation="0" class="rounded-lg pa-4">
          <div class="d-flex align-center justify-space-between">
            <div>
              <div class="text-overline mb-1">Pending Leaves</div>
              <div class="text-h3 font-weight-bold">{{ leaves?.meta?.pagination?.total || 0 }}</div>
            </div>
            <v-icon icon="mdi-calendar-alert" color="warning" size="x-large"></v-icon>
          </div>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>