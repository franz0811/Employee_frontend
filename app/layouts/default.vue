<template>
  <v-container fluid class="pa-0">
    <v-app>
      <v-navigation-drawer v-model="drawer" color="grey-lighten-4">
        <v-list>
          <v-list-item
            prepend-avatar="https://png.pngtree.com/png-clipart/20250128/original/pngtree-human-resources-specialist-3d-icon-isolated-on-a-white-background-symbolizing-png-image_20173414.png"
            subtitle="admin@gmail.com"
            title="HR Administrator"
          ></v-list-item>
        </v-list>

        <v-divider></v-divider>

        <v-list density="compact" nav>
          <v-list-item
            to="/dashboard"
            prepend-icon="mdi-view-dashboard"
            title="Dashboard"
          ></v-list-item>

          <v-list-item
            to="/employees"
            prepend-icon="mdi-account-group"
            title="Employees"
          ></v-list-item>

          <v-list-item
            to="/attendance"
            prepend-icon="mdi-clock-outline"
            title="Attendance"
          ></v-list-item>

          <v-list-item
            to="/leaves"
            prepend-icon="mdi-calendar-blank"
            title="Leave Requests"
          ></v-list-item>
        </v-list>

        <v-container class="d-flex flex-column justify-end" style="height: 10%;">
          <v-btn
            color="red-darken-1"
            variant="flat"
            block
            to="/login"
          >
            Log out
          </v-btn>
        </v-container>
      </v-navigation-drawer>

      <v-app-bar :elevation="1">
        <template v-slot:prepend>
          <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
        </template>
        <v-app-bar-title>HR Management System</v-app-bar-title>
      </v-app-bar>

      <v-main>
        <v-container fluid>
          <slot />
        </v-container>
      </v-main>
    </v-app>
  </v-container>
</template>

<script setup>
const drawer = ref(true);
</script>