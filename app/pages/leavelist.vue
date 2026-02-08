<script setup>
// Kunin ang data at i-filter para sa 'Pending' status gamit ang tamang field name 'statuss'
const { data: pendingLeaves, refresh } = await useFetch('http://localhost:1337/api/leave-requests?filters[statuss][$eq]=Pending&populate=*');

// Function para sa pag-update ng status mula Pending patungong Approved o Rejected
const updateStatus = async (docId, newStatus) => {
  try {
    // Ginagamit ang documentId para sa PUT request sa Strapi v5
    await $fetch(`http://localhost:1337/api/leave-requests/${docId}`, {
      method: 'PUT',
      body: {
        data: {
          statuss: newStatus // Ito ay magiging 'Approved' o 'Rejected'
        }
      }
    });
    // I-refresh ang table para mawala na sa listahan ang inaksyong request
    refresh();
  } catch (err) {
    console.error("Update Error:", err);
    alert("Error: Siguraduhing naka-SAVE ang Public 'update' permission sa Strapi.");
  }
};
</script>

<template>
  <v-container fluid class="pa-8">
    <v-btn to="/leaves" prepend-icon="mdi-arrow-left" variant="text" class="mb-4">Back to Request</v-btn>
    
    <v-card class="rounded-xl" elevation="4">
      <v-card-title class="pa-6 font-weight-bold d-flex align-center text-h5">
        <v-icon start color="warning" size="large">mdi-clock-outline</v-icon>
        Pending Leave Requests
        <v-spacer></v-spacer>
        <v-btn icon="mdi-refresh" variant="tonal" @click="refresh"></v-btn>
      </v-card-title>

      <v-table hover>
        <thead>
          <tr>
            <th class="text-subtitle-1 font-weight-bold">Employee</th>
            <th class="text-subtitle-1 font-weight-bold">Start Date</th>
            <th class="text-subtitle-1 font-weight-bold">End Date</th>
            <th class="text-subtitle-1 font-weight-bold">Reason</th>
            <th class="text-subtitle-1 font-weight-bold text-center">Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="leave in pendingLeaves?.data" :key="leave.id">
            <td class="font-weight-medium">
              {{ leave.employee?.firstname }} {{ leave.employee?.lastname }}
            </td>
            <td>{{ leave.startDate }}</td>
            <td>{{ leave.endDate }}</td>
            <td>{{ leave.reason }}</td>
            <td class="text-center">
              <v-btn 
                icon="mdi-check" 
                color="success" 
                variant="tonal" 
                size="small" 
                class="mr-2"
                title="Approve"
                @click="updateStatus(leave.documentId, 'Approved')"
              ></v-btn>

              <v-btn 
                icon="mdi-close" 
                color="error" 
                variant="tonal" 
                size="small" 
                title="Reject"
                @click="updateStatus(leave.documentId, 'Rejected')"
              ></v-btn>
            </td>
          </tr>
          <tr v-if="pendingLeaves?.data?.length === 0">
            <td colspan="5" class="text-center py-8 text-grey-darken-1 italic">
              Walang pending na request sa ngayon.
            </td>
          </tr>
        </tbody>
      </v-table>
    </v-card>
  </v-container>
</template>