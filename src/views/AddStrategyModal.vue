<template>
  <div
    class="inset-0 bg-black bg-opacity-50 flex justify-center items-center z-50 relative"
  >
    <div class="bg-white p-6 rounded shadow-md w-96">
      <h2 class="text-lg font-bold mb-4 text-[#B48D3E]">Add New Category {{ isLoading }}</h2>

      <!-- Category Name Input -->
      <input
        v-model="strategyName"
        type="text"
        placeholder="Enter category name"
        class="w-full border border-gray-300 p-2 rounded mb-4"
      />

      <!-- Category Icon Input -->
      <input
        type="file"
        accept="image/*"
        @change="handleIconUpload"
        class="w-full border border-gray-300 p-2 rounded mb-4"
      />

      <!-- Preview -->
      <div v-if="iconPreview" class="mb-4">
        <img
          :src="iconPreview"
          alt="Preview"
          class="h-10 w-10 object-contain mx-auto"
        />
      </div>

      <div class="flex justify-end gap-2">
        <button @click="$emit('close')" class="px-4 py-2 rounded bg-gray-300">
          Cancel
        </button>
        <button
          @click="submitStrategy"
          class="px-4 py-2 rounded bg-[#B48D3E] text-white disabled:opacity-60 flex items-center justify-center"
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
          <span>{{ isLoading ? "Adding..." : "Add" }}</span>
        </button>
      </div>
    </div>

    <!-- Loader Overlay -->
    <div
      v-if="isLoading"
      class="absolute inset-0 bg-black bg-opacity-40 flex items-center justify-center z-50"
    >
      <div
        class="h-10 w-10 rounded-full border-4 border-white border-t-transparent animate-spin"
      ></div>
    </div>
  </div>
</template>

<script setup>
import { ref, defineEmits } from "vue";

const emit = defineEmits(["add-strategy", "close"]);

const strategyName = ref("");
const iconPreview = ref(null);
const isLoading = ref(false);

function handleIconUpload(event) {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = () => {
      iconPreview.value = reader.result;
    };
    reader.readAsDataURL(file);
  }
}

async function submitStrategy() {
  console.log(isLoading.value)
  if (!strategyName.value.trim()) return;

  isLoading.value = true;

  try {
    // Simulate async task
    console.log(isLoading.value)
    await new Promise((resolve) => setTimeout(resolve, 1000));

    emit("add-strategy", {
      name: strategyName.value.trim(),
      icon: iconPreview.value,
    });

    emit("close");
    strategyName.value = "";
    iconPreview.value = null;
  } catch (error) {
    console.error("Error submitting strategy:", error);
  } finally {
    isLoading.value = false;
  }
}
</script>
