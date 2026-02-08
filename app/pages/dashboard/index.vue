<script setup>
// 1. Kukunin ang bilang ng Employees
const { data: employees } = await useFetch('http://localhost:1337/api/employees')

// 2. Kukunin ang Attendance logs para sa araw na ito
const today = new Date().toISOString().split('T')[0]
const { data: attendance } = await useFetch(`http://localhost:1337/api/attendances?filters[date][$eq]=${today}`)

// 3. Kukunin ang PENDING Leave Requests
const { data: pendingLeaves } = await useFetch('http://localhost:1337/api/leave-requests?filters[statuss][$eq]=Pending&populate=*')

// 4. Kukunin ang APPROVED Leave Requests para sa Listahan
const { data: approvedLeaves } = await useFetch('http://localhost:1337/api/leave-requests?filters[statuss][$eq]=Approved&populate=*')

// 5. Kukunin ang REJECTED Leave Requests para sa Listahan
const { data: rejectedLeaves } = await useFetch('http://localhost:1337/api/leave-requests?filters[statuss][$eq]=Rejected&populate=*')
</script>

<template>
  <v-container fluid>
    <v-row>
      <v-col cols="12" md="4">
        <v-card border elevation="0" class="rounded-lg pa-4" hover to="/employees">
          <div class="d-flex align-center justify-space-between">
            <div>
              <div class="text-overline mb-1">Total Workforce</div>
              <div class="text-h4 font-weight-bold">{{ employees?.meta?.pagination?.total || 0 }}</div>
            </div>
            <v-icon icon="mdi-account-group" color="primary" size="large"></v-icon>
          </div>
        </v-card>
      </v-col>

      <v-col cols="12" md="4">
        <v-card border elevation="0" class="rounded-lg pa-4" hover to="/attendance">
          <div class="d-flex align-center justify-space-between">
            <div>
              <div class="text-overline mb-1">Today's Attendance</div>
              <div class="text-h4 font-weight-bold">{{ attendance?.meta?.pagination?.total || 0 }}</div>
            </div>
            <v-icon icon="mdi-clock-check" color="success" size="large"></v-icon>
          </div>
        </v-card>
      </v-col>

      <v-col cols="12" md="4">
        <v-card border elevation="0" class="rounded-lg pa-4" hover to="/leavelist">
          <div class="d-flex align-center justify-space-between">
            <div>
              <div class="text-overline mb-1">Pending Requests</div>
              <div class="text-h4 font-weight-bold">{{ pendingLeaves?.meta?.pagination?.total || 0 }}</div>
            </div>
            <v-icon icon="mdi-calendar-clock" color="warning" size="large"></v-icon>
          </div>
        </v-card>
      </v-col>
    </v-row>

    <v-row class="mt-6">
      <v-col cols="12" md="6">
        <v-card class="rounded-xl overflow-hidden" border elevation="2">
          <v-toolbar color="success" density="compact">
            <v-icon start class="ml-4">mdi-check-circle</v-icon>
            <v-toolbar-title class="font-weight-bold">Approved Leave Requests</v-toolbar-title>
          </v-toolbar>
          
          <v-table density="comfortable" hover>
            <thead>
              <tr>
                <th class="font-weight-bold">Employee</th>
                <th class="font-weight-bold">Dates</th>
                <th class="font-weight-bold">Status</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="leave in approvedLeaves?.data" :key="leave.id">
                <td class="font-weight-medium">
                  {{ leave.employee?.firstname }} {{ leave.employee?.lastname }}
                </td>
                <td class="text-caption">
                  {{ leave.startDate }} to {{ leave.endDate }}
                </td>
                <td>
                  <v-chip color="success" size="x-small" variant="flat">APPROVED</v-chip>
                </td>
              </tr>
              <tr v-if="approvedLeaves?.data?.length === 0">
                <td colspan="3" class="text-center py-4 text-grey italic">No approved requests yet.</td>
              </tr>
            </tbody>
          </v-table>
        </v-card>
      </v-col>

      <v-col cols="12" md="6">
        <v-card class="rounded-xl overflow-hidden" border elevation="2">
          <v-toolbar color="error" density="compact">
            <v-icon start class="ml-4">mdi-close-circle</v-icon>
            <v-toolbar-title class="font-weight-bold">Rejected Leave Requests</v-toolbar-title>
          </v-toolbar>
          
          <v-table density="comfortable" hover>
            <thead>
              <tr>
                <th class="font-weight-bold">Employee</th>
                <th class="font-weight-bold">Reason</th>
                <th class="font-weight-bold">Status</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="leave in rejectedLeaves?.data" :key="leave.id">
                <td class="font-weight-medium">
                  {{ leave.employee?.firstname }} {{ leave.employee?.lastname }}
                </td>
                <td class="text-truncate" style="max-width: 150px;">
                  {{ leave.reason }}
                </td>
                <td>
                  <v-chip color="error" size="x-small" variant="flat">REJECTED</v-chip>
                </td>
              </tr>
              <tr v-if="rejectedLeaves?.data?.length === 0">
                <td colspan="3" class="text-center py-4 text-grey italic">No rejected requests yet.</td>
              </tr>
            </tbody>
          </v-table>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>