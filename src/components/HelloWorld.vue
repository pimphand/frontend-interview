<script setup>
import { ref, onMounted } from "vue";

// Form state
const form = ref({
  id: null, // Added to track the product ID during update
  name: "",
  description: "",
  price: 0,
  stock: 0,
});

// Products state
const products = ref([]);
const isUpdating = ref(false); // Flag to toggle between add and update modes

let url = "https://backend.dmpt.my.id/api/products";
// Fetch products from the backend
const fetchProducts = async () => {
  try {
    const response = await fetch(url);
    products.value = await response.json();
  } catch (error) {
    console.error("Error fetching products:", error);
  }
};

// Add a product
const addProduct = async () => {
  try {
    const response = await fetch(url, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(form.value),
    });
    const data = await response.json();
    if (response.ok) {
      products.value.push({ ...form.value, id: data.productId });
      resetForm();
    }
  } catch (error) {
    console.error("Error adding product:", error);
  }
};

// Update a product
const updateProduct = async () => {
  try {
    const response = await fetch(`${url}/${form.value.id}`, {
      method: "PUT",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(form.value),
    });
    if (response.ok) {
      const updatedProductIndex = products.value.findIndex(
        (product) => product.id === form.value.id
      );
      if (updatedProductIndex !== -1) {
        products.value[updatedProductIndex] = { ...form.value };
      }
      resetForm();
    }
  } catch (error) {
    console.error("Error updating product:", error);
  }
};

// Delete a product
const deleteProduct = async (id) => {
  try {
    const response = await fetch(`${url}/${id}`, {
      method: "DELETE",
    });
    if (response.ok) {
      products.value = products.value.filter((product) => product.id !== id);
    }
  } catch (error) {
    console.error("Error deleting product:", error);
  }
};

// Edit a product
const editProduct = (product) => {
  form.value = { ...product }; // Populate form with product data
  isUpdating.value = true; // Enable update mode
};

// Reset form
const resetForm = () => {
  form.value = {
    id: null,
    name: "",
    description: "",
    price: 0,
    stock: 0,
  };
  isUpdating.value = false; // Reset to add mode
};

// Fetch products on mount
onMounted(fetchProducts);
</script>

<template>
  <div class="p-6 max-w-4xl mx-auto">
    <!-- Form Section -->
    <h2 class="text-3xl font-extrabold mb-6 text-blue-700">
      {{ isUpdating ? "Update Product" : "Add Product" }}
    </h2>
    <form
      @submit.prevent="isUpdating ? updateProduct() : addProduct()"
      class="bg-gray-50 shadow-lg rounded-lg p-6 mb-6"
    >
      <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
        <!-- Product Name -->
        <div>
          <label
            for="name"
            class="block text-sm font-medium text-gray-700 mb-1"
          >
            Product Name
          </label>
          <input
            type="text"
            id="name"
            v-model="form.name"
            class="w-full border border-gray-300 rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
            placeholder="Enter product name"
            required
          />
        </div>

        <!-- Description -->
        <div>
          <label
            for="description"
            class="block text-sm font-medium text-gray-700 mb-1"
          >
            Description
          </label>
          <textarea
            id="description"
            v-model="form.description"
            class="w-full border border-gray-300 rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
            placeholder="Enter product description"
            required
          ></textarea>
        </div>

        <!-- Price -->
        <div>
          <label
            for="price"
            class="block text-sm font-medium text-gray-700 mb-1"
          >
            Price
          </label>
          <input
            type="number"
            id="price"
            v-model.number="form.price"
            class="w-full border border-gray-300 rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
            placeholder="Enter price"
            required
          />
        </div>

        <!-- Stock -->
        <div>
          <label
            for="stock"
            class="block text-sm font-medium text-gray-700 mb-1"
          >
            Stock
          </label>
          <input
            type="number"
            id="stock"
            v-model.number="form.stock"
            class="w-full border border-gray-300 rounded-md px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
            placeholder="Enter stock"
            required
          />
        </div>
      </div>

      <!-- Submit Button -->
      <div class="mt-4 text-right">
        <button
          type="submit"
          class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-md focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500"
        >
          {{ isUpdating ? "Update Product" : "Add Product" }}
        </button>
        <button
          v-if="isUpdating"
          type="button"
          @click="resetForm"
          class="ml-4 bg-gray-500 hover:bg-gray-700 text-white font-bold py-2 px-4 rounded-md focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
        >
          Cancel
        </button>
      </div>
    </form>

    <!-- Table Section -->
    <h2 class="text-3xl font-extrabold mb-6 text-blue-700">Products List</h2>
    <table class="table-auto w-full bg-white shadow-md rounded-lg">
      <thead>
        <tr class="bg-gray-200 text-black">
          <th class="px-4 py-2">ID</th>
          <th class="px-4 py-2">Name</th>
          <th class="px-4 py-2">Description</th>
          <th class="px-4 py-2">Price</th>
          <th class="px-4 py-2">Stock</th>
          <th class="px-4 py-2">Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="product in products"
          :key="product.id"
          class="border-t hover:bg-gray-100"
        >
          <td class="text-black px-4 py-2 text-center">{{ product.id }}</td>
          <td class="text-black px-4 py-2">{{ product.name }}</td>
          <td class="text-black px-4 py-2">{{ product.description }}</td>
          <td class="text-black px-4 py-2 text-center">{{ product.price }}</td>
          <td class="text-black px-4 py-2 text-center">{{ product.stock }}</td>
          <td class="text-black px-4 py-2 text-center">
            <button
              @click="editProduct(product)"
              class="bg-yellow-500 hover:bg-yellow-700 text-white font-bold py-1 px-2 rounded mr-2"
            >
              Edit
            </button>
            <button
              @click="deleteProduct(product.id)"
              class="bg-red-500 hover:bg-red-700 text-white font-bold py-1 px-2 rounded"
            >
              Delete
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped>
/* Add subtle table row hover effects */
.table-auto tbody tr:hover {
  transition: background-color 0.2s ease-in-out;
}
</style>
