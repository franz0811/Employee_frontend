<template>
  <v-container fluid>
    <v-app>
      <!-- Navigation Drawer -->
      <v-navigation-drawer v-model="drawer">
        <v-list>
          <v-list-item
            prepend-avatar="https://img.freepik.com/free-vector/vector-education-logo_779267-2083.jpg?semt=ais_hybrid&w=740&q=80"
            subtitle="admin@gmailcom"
            title="Administrator"
          ></v-list-item>
        </v-list>

        <v-divider></v-divider>

        <v-list density="compact" nav>
          <v-list-item
            to="/dashboard"
            prepend-icon="mdi-view-dashboard"
            title="Dashboard"
            value="myfiles"
          ></v-list-item>
          <v-list-item
            to="/inventory"
            prepend-icon="mdi-playlist-edit"
            title="Inventory"
            value="shared"
          ></v-list-item>
          <v-list-item
            to="/category"
            prepend-icon="mdi-shape-plus"
            title="Categories"
            value="starred"
          ></v-list-item>
          <v-list-item
            to="/users"
            prepend-icon="mdi-account-multiple"
            title="Users"
            value="starred"
          ></v-list-item>
        </v-list>
        <v-container class="d-flex flex-column justify-end" style="height: 65%;">
          <v-btn
          color="red"
          to="/"
          >
            Log out
          </v-btn>
        </v-container>
      </v-navigation-drawer>
      <!-- Appbar -->

      <v-app-bar :elevation="2">
        <template v-slot:prepend>
          <v-app-bar-nav-icon
            @click.stop="drawer = !drawer"
          ></v-app-bar-nav-icon>
        </template>

        <v-app-bar-title>Inventory</v-app-bar-title>
      </v-app-bar>
      <!-- Main Content -->

      <v-main>
        <div>
          <slot />
        </div>
      </v-main>
    </v-app>
  </v-container>
</template>

<script setup>
const drawer = ref(true);
</script>

<style></style>
