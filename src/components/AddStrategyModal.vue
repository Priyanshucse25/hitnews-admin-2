<!-- sidebar category modal -->
<template>
  <div
    class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
  >
    <div class="bg-white p-6 rounded shadow-md w-96">
      <h2 class="text-xl font-bold mb-4 text-[#B48D3E]">Add Category</h2>
      
      <!-- Category Name Input -->
      <input
        v-model="strategyName"
        type="text"
        placeholder="Enter Category name"
        class="border px-3 py-2 w-full rounded mb-4 text-slate-900"
      />
      
      <!-- Icon Upload -->
      <label class="block mb-4">
        <span class="text-sm font-medium text-gray-700">Choose Icon</span>
        <input
          type="file"
          accept="image/*"
          @change="handleIconUpload"
          class="mt-1 block w-full text-sm text-gray-500"
        />
      </label>
      
      <!-- Icon Preview -->
      <div v-if="iconPreview" class="mb-4">
        <p class="text-sm text-gray-600">Preview:</p>
        <img :src="iconPreview" class="h-12 w-12 object-contain" />
      </div>
      
      <!-- Error or Success Message -->
      <div
        v-if="message"
        :class="messageType === 'error' ? 'text-red-600' : 'text-green-600'"
        class="mb-4"
      >
        <p>{{ message }}</p>
      </div>
      
      <!-- Buttons -->
      <div class="flex justify-end gap-2">
        <button
          @click="$emit('close')"
          class="bg-gray-300 text-black px-4 py-2 rounded"
        >
          Cancel
        </button>
        <button
          @click="submit"
          class="bg-[#B48D3E] text-white px-4 py-2 rounded flex items-center justify-center"
          :disabled="isLoading"
        >
          <svg
            v-if="isLoading"
            class="animate-spin h-4 w-4 mr-2 text-white"
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
          >
            <circle
              class="opacity-25"
              cx="12"
              cy="12"
              r="10"
              stroke="currentColor"
              stroke-width="4"
            ></circle>
            <path
              class="opacity-75"
              fill="currentColor"
              d="M4 12a8 8 0 018-8v4a4 4 0 00-4 4H4z"
            ></path>
          </svg>
          <span>{{ isLoading ? 'Adding...' : 'Add' }}</span>
        </button>
      </div>
    </div>
    
    <!-- Loader Overlay -->
    <div
      v-if="isLoading"
      class="absolute inset-0 bg-black bg-opacity-40 flex items-center justify-center z-50"
    >
      <div class="h-10 w-10 rounded-full border-4 border-white border-t-transparent animate-spin"></div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import { useCategoryStore } from "@/stores/categoryStore";
const emit = defineEmits(["close", "add-strategy"]);
const categoryStore = useCategoryStore();

const strategyName = ref("");
const iconPreview = ref(null);
const selectedFile = ref(null); // store the actual file
const message = ref(""); // To show success or error messages
const messageType = ref(""); // 'error' or 'success'
const isLoading = ref(false); // Add isLoading ref

function handleIconUpload(e) {
  const file = e.target.files[0];
  if (file) {
    selectedFile.value = file;
    const reader = new FileReader();
    reader.onload = () => {
      iconPreview.value = reader.result;
    };
    reader.readAsDataURL(file);
  }
}

const submit = async () => {
  if (!strategyName.value || !selectedFile.value) {
    message.value = "Please provide both a name and an icon.";
    messageType.value = "error";
    return;
  }
  
  isLoading.value = true; // Set loading to true when starting submission

  try {
    const formData = new FormData();
    formData.append("name", strategyName.value);
    formData.append("image", selectedFile.value);
    
    const response = await axios.post(
      "https://backend-5gsq.onrender.com/api/categories",
      formData,
      {
        headers: {
          "Content-Type": "multipart/form-data",
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
      }
    );
    
    await categoryStore.getCategories();
    
    const newCategory = response?.data?.category || {};
    
    emit("add-strategy", {
      name: newCategory.name,
      image: newCategory.image, // Adjust if backend uses a different property name
    });
    
    // Reset form
    strategyName.value = "";
    iconPreview.value = null;
    selectedFile.value = null;
    
    message.value = "Category created successfully!";
    messageType.value = "success";
    
    emit("close");
  } catch (err) {
    console.error(
      "Error creating category:",
      err.response?.data || err.message
    );
    message.value = "Error creating category. Please try again.";
    messageType.value = "error";
  }finally {
    isLoading.value = false;
  }
};
</script>