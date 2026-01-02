<template>
  <v-container fluid>
    <v-card title="Category" flat elevation="2">
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

      <v-btn @click="createDialog = true" color="primary" class="ma-4">
        Create Category
      </v-btn>

<v-data-table
:headers="headers"
:items="categories"
:search="search"
>
<template #item.actions="{ item }">
  <v-menu location="start top">
    <template #activator="{ props }">
      <v-btn
        icon="mdi-dots-vertical"
        variant="text"
        v-bind="props"
      />
    </template>

    <v-list>
      <v-list-item
        title="Edit"
        @click="editCategory(item)"
      />
      <v-list-item
        title="Delete"
        @click="deleteCategory(item)"
      />
    </v-list>
  </v-menu>
</template>
</v-data-table>

<v-dialog
      v-model="createDialog"
      width="auto"
    >
      <v-card
        max-width="500"
        prepend-icon="mdi-pencil"
        title="Add Category"
      >

        <v-card-text>
          <v-form>
            <v-text-field
            v-model="CategoryName"
              label="Category Name*"
              required
            ></v-text-field>

            <v-textarea
              v-model="CategoryDescription"
              label="Description*"
              required
            ></v-textarea>
          </v-form>
        </v-card-text>

        <template v-slot:actions>
          <v-btn
            class="ms-auto"
            text="Create"
            @click="createCategory()"
          ></v-btn>
        </template>
      </v-card>
    </v-dialog>
    </v-card>

    <v-snackbar v-model="snackbar" :color="snackbarColor">
      {{ snackbarText }}</v-snackbar
    >

  </v-container>
</template>

<script setup>
// const { data: category, error } = await useFetch(
//   "http://localhost:1337/api/categories"
// );
const categories = ref([]);
// console.log(category.data);
const search = ref("");
const createDialog = ref(false);

const CategoryName = ref();
const CategoryDescription = ref();
const snackbar = ref(false);
const snackbarColor = ref("");
const snackbarText = ref();

const headers = [
  { key: "category_name", title: "Category Name" },
  { key: "description", title: "Description" },
  { key: "createdAt", title: "Created At" },
  { title: "", key: "actions" },
];
const getCategories = async () => {
  try {
    const res = await $fetch("http://localhost:1337/api/categories");
    categories.value = res.data;
  } catch (err) {
    console.log(err);
  }
};


const createCategory = async () => {
  // alert("Create", item);

  try {
    let payload = {
      category_name: CategoryName.value,
      description: CategoryDescription.value,
    };
    const res = await useFetch("http://localhost:1337/api/categories", {
      method: "POST",
      body: {
        data: payload,
      },
    });
    if (res) {
      createDialog.value = false;
      // console.log("Successfully Created", categoryName.value);
      snackbarColor.value = "green";
      snackbarText.value = "Successfully Created!";
      snackbar.value = true;
      getCategories();
    } else {
      alert("Error creating category");
    }

  } catch (error) {
    console.log(error);
  }
  
};

const editCategory = (item) => {
  // alert("Edit", item);
  
};

const deleteCategory = async (item) => {
  // alert("Delete", item);
  try {
    const res = await useFetch(
      `http://localhost:1337/api/categories/${item.documentId}`,
      {
        method: "DELETE",
      }
    );

    console.log("Successfully Deleted!");
    snackbarColor.value = "green";
    snackbarText.value = "Successfully Deleted!";
    snackbar.value = true;
    getCategories();
  } catch (err) {
    console.log(err);
  }
};

onMounted(() => {
  getCategories();
});

</script>

<style></style>
