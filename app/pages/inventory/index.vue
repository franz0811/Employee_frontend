<template>
  <v-container fluid>
    <v-card title="Inventory" flat elevation="2" class="pa-4">
      <template v-slot:text>
        <v-text-field
          v-model="search"
          label="Search"
          prepend-inner-icon="mdi-magnify"
          variant="outlined"
          hide-details
          single-line
        ></v-text-field>
      </template>

      <!-- Selectable Category -->
      <v-select
        v-model="selectedCategory"
        clearable
        label="Select Category"
        :items="category.data"
        item-title="category_name"
        item-value="id"
        class="mx-4"
        variant="outlined"
      ></v-select>

      <v-data-table
        :headers="headers"
        :items="filteredInventories"
        :search="search"
      ></v-data-table>
    </v-card>
  </v-container>
</template>

<script setup>
// const { data: inventory, error } = await useFetch(
//   "http://localhost:1337/api/inventories"
// );
const { data: category, cerror } = await useFetch(
  "http://localhost:1337/api/categories"
);

const { data: inventory, error: invError } = await useFetch(
  "http://localhost:1337/api/inventories?populate=category"
);

// console.log(inventory.value);

// const categoryOptions = category.value.data.map((data) => ({
//   title: data.category_name,
//   value: data.id,
// }));

// const categoryOptions = computed(() => {
//   // const cats = category.value?.data || [];
//   return category.value.data.map((cat) => ({
//     title: cat.category_name,
//     value: cat.id,
//   }));
// });

// console.log(categoryOptions.value);

// console.log(inventory.value.data[0].product_name);
const selectedCategory = ref(null);
const search = ref("");

// console.log(!selectedCategory.value);

const headers = [
  { key: "product_name", title: "Product Name" },
  { key: "description", title: "Description" },
  { key: "quantity", title: "Quantity" },
  { key: "condition", title: "Condition" },
  { key: "location", title: "Location" },
  // { key: "category_name", title: "Category" },
];

const filteredInventories = computed(() => {
  const items = inventory.value.data;
  if (!selectedCategory.value) return items;
  return items.filter((item) => {
    const itemCatId = item.category.id;
    return itemCatId === selectedCategory.value;
  });
});

// console.log(inventory.value?.data);
</script>

<style></style>
