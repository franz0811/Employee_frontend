<template>
  <v-container fluid>
    <v-card title="Category" flat elevation="2">

      <!-- Search -->
      <template #text>
        <v-text-field
          v-model="search"
          label="Search"
          prepend-inner-icon="mdi-magnify"
          variant="outlined"
          hide-details
          single-line
        />
      </template>

      <!-- Create Button -->
      <v-btn color="primary" class="ma-4" @click="createDialog = true">
        Create Category
      </v-btn>

      <!-- Data Table -->
      <v-data-table
        :headers="headers"
        :items="categories"
        :search="search"
      >
        <-- Alternative UI design for Edit and Delete option -->
          <-- <template v-slot:item.actions="{ item }">
      <v-btn
        variant="text"
        icon
        @click="openEditDialog(item)"
      >
        <v-icon>mdi-pencil</v-icon>
      </v-btn>

      <v-btn variant="text" icon @click="deleteCategory(item)">
        <v-icon>mdi-delete</v-icon>
      </v-btn>
    </template> -->
        <-- Actual better UI Design for Edit and Delete option-->
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
                @click="openEditDialog(item)"
              />
              <v-list-item
                title="Delete"
                @click="deleteCategory(item)"
              />
            </v-list>
          </v-menu>
        </template> 
      </v-data-table>

    </v-card>

    <!-- CREATE CATEGORY DIALOG -->
    <v-dialog v-model="createDialog" width="500">
      <v-card prepend-icon="mdi-plus" title="Add Category">
        <v-card-text>
          <v-form>
            <v-text-field
              v-model="CategoryName"
              label="Category Name*"
              required
            />
            <v-textarea
              v-model="CategoryDescription"
              label="Description*"
              required
            />
          </v-form>
        </v-card-text>

        <v-card-actions>
          <v-spacer />
          <v-btn color="primary" @click="createCategory">Create</v-btn>
          <v-btn variant="flat" color="red" @click="createDialog = false">Cancel</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- EDIT CATEGORY DIALOG -->
    <v-dialog v-model="editDialog" width="500">
      <v-card prepend-icon="mdi-pencil" title="Edit Category">
        <v-card-text>
          <v-form>
            <v-text-field
              v-model="editedCategory.category_name"
              label="Category Name*"
              required
            />
            <v-textarea
              v-model="editedCategory.description"
              label="Description*"
              required
            />
          </v-form>
        </v-card-text>

        <v-card-actions>
          <v-spacer />
          <v-btn color="primary" @click="updateCategory">Save</v-btn>
          <v-btn variant="flat" color="red"  @click="editDialog = false">Cancel</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Snackbar -->
    <v-snackbar v-model="snackbar" :color="snackbarColor">
      {{ snackbarText }}
    </v-snackbar>

  </v-container>
</template>

<script setup>
/* ------------------ STATE ------------------ */
const categories = ref([]);
const search = ref("");

const createDialog = ref(false);
const editDialog = ref(false);

const CategoryName = ref("");
const CategoryDescription = ref("");

const editedCategory = ref({
  documentId: null,
  category_name: "",
  description: "",
});

const snackbar = ref(false);
const snackbarColor = ref("");
const snackbarText = ref("");

/* ------------------ TABLE HEADERS ------------------ */
const headers = [
  { key: "category_name", title: "Category Name" },
  { key: "description", title: "Description" },
  { key: "createdAt", title: "Created At" },
  { key: "actions", title: "", sortable: false },
];

/* ------------------ FETCH ------------------ */
const getCategories = async () => {
  try {
    const res = await $fetch("http://localhost:1337/api/categories");
    categories.value = res.data;
  } catch (err) {
    console.log(err);
  }
};

/* ------------------ CREATE ------------------ */
const createCategory = async () => {
  try {
    await $fetch("http://localhost:1337/api/categories", {
      method: "POST",
      body: {
        data: {
          category_name: CategoryName.value,
          description: CategoryDescription.value,
        },
      },
    });

    createDialog.value = false;
    CategoryName.value = "";
    CategoryDescription.value = "";

    snackbarColor.value = "green";
    snackbarText.value = "Category created successfully!";
    snackbar.value = true;

    getCategories();
  } catch (err) {
    console.log(err);
  }
};

/* ------------------ EDIT ------------------ */
const openEditDialog = (item) => {
  editedCategory.value = {
    documentId: item.documentId,
    category_name: item.category_name,
    description: item.description,
  };
  editDialog.value = true;
};

const updateCategory = async () => {
  try {
    await $fetch(
      `http://localhost:1337/api/categories/${editedCategory.value.documentId}`,
      {
        method: "PUT",
        body: {
          data: {
            category_name: editedCategory.value.category_name,
            description: editedCategory.value.description,
          },
        },
      }
    );

    editDialog.value = false;

    snackbarColor.value = "green";
    snackbarText.value = "Category updated successfully!";
    snackbar.value = true;

    getCategories();
  } catch (err) {
    console.log(err);
  }
};

/* ------------------ DELETE ------------------ */
const deleteCategory = async (item) => {
  try {
    await $fetch(
      `http://localhost:1337/api/categories/${item.documentId}`,
      { method: "DELETE" }
    );

    snackbarColor.value = "green";
    snackbarText.value = "Category deleted successfully!";
    snackbar.value = true;

    getCategories();
  } catch (err) {
    console.log(err);
  }
};

/* ------------------ INIT ------------------ */
onMounted(() => {
  getCategories();
});
</script>

<style scoped>
</style>
