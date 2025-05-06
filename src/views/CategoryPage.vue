<template>
  <div class="h-screen flex flex-col overflow-hidden">
    <div class="p-4 sm:p-6 h-full flex flex-col">
      <!-- Header -->
      <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-4 gap-4">
        <h2 class="text-xl sm:text-2xl font-bold text-[#B48D3E]">
          Category: {{ categoryName }}
        </h2>
        <div class="flex flex-col sm:flex-row gap-2 w-full sm:w-auto">
          <button @click="showModal = true"
            class="bg-[#B48D3E] text-white px-4 py-2 rounded hover:opacity-90 w-full sm:w-auto">
            Add Content
          </button>
          <button @click="toggleBanner" :disabled="isLoadingBanners"
            class="bg-[#B48D3E] text-white px-4 py-2 rounded hover:opacity-90 w-full sm:w-auto flex items-center justify-center">
            <span v-if="!isLoadingBanners">Banner</span>
            <svg v-else class="animate-spin h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v4l3-3-3-3v4a8 8 0 000 16v-4l-3 3 3 3v-4a8 8 0 01-8-8z"></path>
            </svg>
          </button>
        </div>
      </div>

      <!-- Banner Section -->
      <div v-if="showBanners" class="bg-gray-100 p-4 rounded-lg mb-6 max-h-[500px] overflow-y-auto">
        <h3 class="text-lg sm:text-xl font-bold text-[#B48D3E] mb-4">Banners</h3>
        <div v-if="isLoadingBanners" class="flex justify-center py-8">
          <svg class="animate-spin h-8 w-8 text-[#B48D3E]" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v4l3-3-3-3v4a8 8 0 000 16v-4l-3 3 3 3v-4a8 8 0 01-8-8z"></path>
          </svg>
        </div>
        <div v-else class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
          <div v-for="(banner, index) in banners" :key="banner._id || index" class="bg-white rounded-lg shadow p-4">
            <img :src="banner.image" alt="Banner" class="h-40 w-full object-cover rounded" />
            <p class="mt-2 text-sm break-words">{{ banner.link }}</p>
            <div class="flex gap-2 mt-2">
              <button @click="openEditBannerModal(banner)" class="text-sm text-[#B48D3E] hover:text-green-600">
                Edit
              </button>
              <button @click="openDeleteBannerModal(banner)" class="text-sm text-red-600 hover:text-red-800">
                Delete
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Cards Section -->
      <div class="flex-1 overflow-y-auto">
        <!-- Loading state for content -->
        <div v-if="isLoadingContent" class="flex justify-center items-center h-64">
          <svg class="animate-spin h-12 w-12 text-[#B48D3E]" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v4l3-3-3-3v4a8 8 0 000 16v-4l-3 3 3 3v-4a8 8 0 01-8-8z"></path>
          </svg>
        </div>
        
        <!-- Content grid -->
        <div v-else class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
          <div v-for="(item, index) in paginatedContent" :key="item._id" @click="toggleCard(item)"
            class="bg-white rounded-xl p-4 shadow text-[#B48D3E] cursor-pointer transition-all duration-300 ease-in-out"
            :class="selectedCardId === item._id ? 'max-h-full' : 'max-h-[400px] overflow-hidden'">
            <h3 class="font-bold text-lg line-clamp-2">
              {{ item.title }}
            </h3>
            <img :src="item.image" alt="Content image" class="h-40 w-full object-cover my-2 rounded" />
            <p class="text-sm text-gray-600" v-html="item.content"
              :class="selectedCardId === item._id ? '' : 'line-clamp-3'"></p>
            <div class="mt-4 border-t pt-3 text-sm text-gray-700 space-y-1">
              <div class="flex justify-between">
                <span class="font-medium">Category:</span>
                <span class="text-[#B48D3E] italic">
                  {{ item.category?.name || item.category || "No Category" }}
                </span>
              </div>
              <div class="flex justify-between">
                <span class="font-medium">Author:</span>
                <span class="text-[#B48D3E] italic">
                  {{ item.author?.username || item.author || "No Author" }}
                </span>
              </div>
              <div class="flex justify-end gap-2 mt-2">
                <button @click.stop="editCard(item)" class="text-blue-500 hover:underline text-sm">Edit</button>
                <button @click.stop="deleteCard(item)" :disabled="isDeletingCard === item._id" class="text-red-500 hover:underline text-sm flex items-center">
                  <span v-if="isDeletingCard !== item._id">Delete</span>
                  <svg v-else class="animate-spin h-4 w-4 text-red-500" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                    <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                    <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v4l3-3-3-3v4a8 8 0 000 16v-4l-3 3 3 3v-4a8 8 0 01-8-8z"></path>
                  </svg>
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Pagination -->
      <div class="mt-6 flex flex-wrap justify-center gap-2">
        <button @click="changePage(page - 1)" :disabled="page === 1 || isLoadingPage"
          class="px-3 py-1 rounded border text-[#B48D3E] border-[#B48D3E] hover:bg-[#B48D3E] hover:text-white disabled:opacity-50">
          Prev
        </button>
        <button v-for="p in totalPages" :key="p" @click="changePage(p)" :disabled="isLoadingPage" :class="[
          page === p
            ? 'bg-[#B48D3E] text-white'
            : 'text-[#B48D3E] border-[#B48D3E] hover:bg-[#B48D3E] hover:text-white',
          'px-3 py-1 rounded border'
        ]">
          {{ p }}
        </button>
        <button @click="changePage(page + 1)" :disabled="page === totalPages || isLoadingPage"
          class="px-3 py-1 rounded border text-[#B48D3E] border-[#B48D3E] hover:bg-[#B48D3E] hover:text-white disabled:opacity-50">
          Next
        </button>
        
        <!-- Pagination loader -->
        <div v-if="isLoadingPage" class="flex items-center ml-2">
          <svg class="animate-spin h-5 w-5 text-[#B48D3E]" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v4l3-3-3-3v4a8 8 0 000 16v-4l-3 3 3 3v-4a8 8 0 01-8-8z"></path>
          </svg>
        </div>
      </div>

      <!-- Modals (Add/Edit Content, Edit Banner, Delete Confirmation) -->
      <!-- Add/Edit Content Modal -->
      <div v-if="showModal"
        class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 overflow-auto">
        <div
          class="bg-white p-4 sm:p-6 rounded-xl w-full max-w-lg max-h-[90vh] text-[#B48D3E] relative overflow-y-auto">
          <h3 class="text-xl font-bold mb-4">
            {{ editingIndex !== null ? "Edit Content" : "Add Content" }}
          </h3>

          <label class="block mb-2">Title</label>
          <input v-model="form.title" type="text" class="w-full px-4 py-2 border rounded mb-4" />

          <label class="block mb-2">Image</label>
          <input type="file" @change="handleImage" class="w-full mb-4" />
          <div v-if="isUploadingImage" class="flex items-center mb-4">
            <svg class="animate-spin h-5 w-5 mr-2 text-[#B48D3E]" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v4l3-3-3-3v4a8 8 0 000 16v-4l-3 3 3 3v-4a8 8 0 01-8-8z"></path>
            </svg>
            <span class="text-sm">Uploading image...</span>
          </div>

          <label class="block mb-2">Content</label>
          <div class="mb-4 border rounded">
            <QuillEditor 
              v-model:content="form.content" 
              toolbar="full"
              contentType="html" 
              :options="editorOptions"
            />
          </div>

          <label class="block mb-2">Category</label>
          <div class="relative mb-4" ref="categoryDropdownRef">
            <input v-model="searchQuery" @focus="showCategoryList = true" placeholder="Search Category"
              class="w-full px-4 py-2 border rounded" />
            <ul v-if="showCategoryList"
              class="absolute bg-white border w-full mt-1 z-10 rounded max-h-40 overflow-y-auto shadow">
              <li v-for="(cat, index) in filteredCategories" :key="index" @click.prevent="selectCategory(cat)"
                class="px-4 py-2 hover:bg-gray-200 cursor-pointer">
                {{ cat }}
              </li>
            </ul>
          </div>

          <div class="flex justify-end gap-2">
            <button @click="cancelEdit"
              class="px-4 py-2 border border-[#B48D3E] rounded text-[#B48D3E] hover:bg-[#B48D3E] hover:text-white">
              Cancel
            </button>
            <button 
              v-if="!isLoading"
              @click="saveContent" 
              class="px-4 py-2 bg-[#B48D3E] text-white rounded hover:opacity-90"
            >
              Save
            </button>
            <button 
              v-else 
              disabled 
              class="px-4 py-2 bg-[#B48D3E] text-white rounded flex items-center justify-center"
            >
              <svg class="animate-spin h-5 w-5 mr-2 text-white" xmlns="http://www.w3.org/2000/svg" fill="none"
                  viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor"
                        stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor"
                      d="M4 12a8 8 0 018-8v4l3-3-3-3v4a8 8 0 000 16v-4l-3 3 3 3v-4a8 8 0 01-8-8z"></path>
              </svg>
              Saving...
            </button>
          </div>
        </div>
      </div>

      <!-- Edit Banner Modal -->
      <div v-if="isEditBannerModalVisible"
        class="fixed inset-0 bg-gray-500 bg-opacity-50 flex items-center justify-center z-50 p-4">
        <div class="bg-white p-6 rounded-lg w-full max-w-md overflow-auto">
          <h3 class="text-xl font-bold text-[#B48D3E] mb-4">Edit Banner</h3>
          <label class="block text-sm mb-1">Banner Image</label>
          <input type="file" @change="handleFileChange" accept="image/*" />
          <div v-if="isUploadingBannerImage" class="flex items-center mt-2">
            <svg class="animate-spin h-5 w-5 mr-2 text-[#B48D3E]" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v4l3-3-3-3v4a8 8 0 000 16v-4l-3 3 3 3v-4a8 8 0 01-8-8z"></path>
            </svg>
            <span class="text-sm">Uploading image...</span>
          </div>
          <label class="block text-sm mt-4 mb-1">Banner Link</label>
          <input v-model="currentBannerToEdit.link" type="text"
            class="mb-4 p-2 border border-gray-300 rounded w-full" />
          <div class="flex justify-end space-x-2">
            <button @click="closeEditBannerModal" class="bg-gray-400 text-white px-4 py-2 rounded">
              Cancel
            </button>
            <button @click="saveBanner" :disabled="isSavingBanner" class="bg-[#B48D3E] text-white px-4 py-2 rounded flex items-center justify-center">
              <span v-if="!isSavingBanner">Save</span>
              <span v-else class="flex items-center">
                <svg class="animate-spin h-5 w-5 mr-2" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                  <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                  <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v4l3-3-3-3v4a8 8 0 000 16v-4l-3 3 3 3v-4a8 8 0 01-8-8z"></path>
                </svg>
                Saving...
              </span>
            </button>
          </div>
        </div>
      </div>

      <!-- Delete Confirmation Modals -->
      <div v-if="isDeleteBannerModalVisible"
        class="fixed inset-0 bg-gray-500 bg-opacity-50 flex items-center justify-center z-50 p-4">
        <div class="bg-white p-6 rounded-lg w-full max-w-md">
          <h3 class="text-xl font-bold text-red-600 mb-4">Are you sure?</h3>
          <p class="mb-4">Do you want to delete this banner?</p>
          <div class="flex justify-end gap-2">
            <button @click="closeDeleteBannerModal" class="bg-gray-400 text-white px-4 py-2 rounded">Cancel</button>
            <button @click="confirmDeleteBanner" :disabled="isDeletingBanner" class="bg-red-600 text-white px-4 py-2 rounded flex items-center justify-center">
              <span v-if="!isDeletingBanner">Delete</span>
              <span v-else class="flex items-center">
                <svg class="animate-spin h-5 w-5 mr-2" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                  <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                  <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v4l3-3-3-3v4a8 8 0 000 16v-4l-3 3 3 3v-4a8 8 0 01-8-8z"></path>
                </svg>
                Deleting...
              </span>
            </button>
          </div>
        </div>
      </div>

      <div v-if="showDeleteModal"
        class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
        <div class="bg-white p-6 rounded-lg w-full max-w-md">
          <h3 class="text-xl font-bold text-red-600 mb-4">Are you sure?</h3>
          <p class="mb-4">Do you want to delete this content?</p>
          <div class="flex justify-end gap-2">
            <button @click="cancelDelete"
              class="px-4 py-2 border border-[#B48D3E] rounded text-[#B48D3E] hover:bg-[#B48D3E] hover:text-white">
              Cancel
            </button>
            <button @click="confirmDelete()" :disabled="isDeleting" class="px-4 py-2 bg-red-600 text-white rounded hover:opacity-90 flex items-center justify-center">
              <span v-if="!isDeleting">Delete</span>
              <span v-else class="flex items-center">
                <svg class="animate-spin h-5 w-5 mr-2" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                  <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                  <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v4l3-3-3-3v4a8 8 0 000 16v-4l-3 3 3 3v-4a8 8 0 01-8-8z"></path>
                </svg>
                Deleting...
              </span>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onBeforeUnmount } from "vue";
import { useRoute } from "vue-router";
import { QuillEditor } from "@vueup/vue-quill";
import "@vueup/vue-quill/dist/vue-quill.snow.css";
import { useCategoryStore } from "@/stores/categoryStore";
import axios from "axios";

// Refs & States
const route = useRoute();
const categoryName = ref(decodeURIComponent(route.params.name || ""));
const categoryStore = useCategoryStore();

const allContent = ref([]);
const banners = ref([]);
const showModal = ref(false);
const showDeleteModal = ref(false);
const indexToDelete = ref(null);
const editingIndex = ref(null);
const form = ref({
  title: "",
  image: "",
  content: "",
  category: "",
  author: "",
});
const searchQuery = ref("");
const showCategoryList = ref(false);
const categoryDropdownRef = ref(null);

// Loading States
const isLoading = ref(false);
const isLoadingContent = ref(false);
const isLoadingBanners = ref(false);
const isLoadingPage = ref(false);
const isDeleting = ref(false);
const isDeletingCard = ref(null); // Will store the ID of the card being deleted
const isDeletingBanner = ref(false);
const isUploadingImage = ref(false);
const isUploadingBannerImage = ref(false);
const isSavingBanner = ref(false);

// Quill Editor Options
const editorOptions = {
  modules: {
    toolbar: [
      ['bold', 'italic', 'underline', 'strike'],        
      ['blockquote', 'code-block'],
      [{ 'header': 1 }, { 'header': 2 }],               
      [{ 'list': 'ordered' }, { 'list': 'bullet' }],
      [{ 'script': 'sub' }, { 'script': 'super' }],     
      [{ 'indent': '-1' }, { 'indent': '+1' }],         
      [{ 'direction': 'rtl' }],                         
      [{ 'size': ['small', false, 'large', 'huge'] }],  
      [{ 'header': [1, 2, 3, 4, 5, 6, false] }],
      [{ 'color': [] }, { 'background': [] }],          
      [{ 'font': [] }],
      [{ 'align': [] }],
      ['link', 'image', 'video'],                       
      ['clean']                                         
    ]
  },
  placeholder: 'Write your content here...',
  theme: 'snow'
};

// Banners
const isEditBannerModalVisible = ref(false);
const isDeleteBannerModalVisible = ref(false);
const currentBannerToEdit = ref({ image: "", link: "" });
const bannerToDeleteId = ref(null);

const showBanners = ref(false);

// Computed
const categories = computed(() => categoryStore.categories.map((c) => c.name));
const filteredCategories = computed(() => {
  const query = searchQuery.value.toLowerCase();
  return categories.value.filter((cat) => cat.toLowerCase().includes(query));
});

const handleFileChange = async (e) => {
  const file = e.target.files[0];
  if (file) {
    isUploadingBannerImage.value = true;
    try {
      currentBannerToEdit.value.newFile = file;
      // Any additional processing if needed
    } finally {
      isUploadingBannerImage.value = false;
    }
  }
};

const selectedCardId = ref(null);

function toggleCard(item) {
  selectedCardId.value = selectedCardId.value === item._id ? null : item._id;
}

// Watch route param
watch(
  () => route.params.name,
  (newVal) => {
    categoryName.value = decodeURIComponent(newVal || "");
    page.value = 1;
    fetchContent(); // Refetch content when category changes
  }
);

// Category Dropdown
const selectCategory = (cat) => {
  form.value.category = cat;
  searchQuery.value = cat;
  showCategoryList.value = false;
};

// Fetch content and banners
const fetchContent = async () => {
  isLoadingContent.value = true;
  try {
    const res = await axios.get("https://backend-5gsq.onrender.com/api/news", {
      headers: {
        "Content-Type": "multipart/form-data",
        Authorization: `Bearer ${localStorage.getItem("token")}`,
      },
    });

    allContent.value = res.data.newsArticles || [];
  } catch (err) {
    console.error("Fetch error:", err);
  } finally {
    isLoadingContent.value = false;
  }
};

const fetchBanners = async () => {
  isLoadingBanners.value = true;
  try {
    const res = await axios.get("https://backend-5gsq.onrender.com/api/banner", {
      headers: {
        Authorization: `Bearer ${localStorage.getItem("token")}`,
      },
    });

    banners.value = res.data.info || [];
    console.log(banners.value);
  } catch (err) {
    console.error("Fetch error:", err.response?.data || err.message);
  } finally {
    isLoadingBanners.value = false;
  }
};

// Save Content (Add / Edit)
const saveContent = async () => {
  if (!form.value.category) {
    alert("Please select a category.");
    return;
  }
  
  isLoading.value = true;
  try {
    const formData = new FormData();
    formData.append("title", form.value.title);
    formData.append("content", form.value.content);
    formData.append("category", form.value.category);
    formData.append("author", form.value.author);

    if (form.value.image && typeof form.value.image !== "string") {
      formData.append("image", form.value.image);
    }

    const config = {
      headers: {
        "Content-Type": "multipart/form-data",
        Authorization: `Bearer ${localStorage.getItem("token")}`,
      },
    };

    if (editingIndex.value !== null) {
      const id = allContent.value[editingIndex.value]._id;
      const { data } = await axios.put(
        `https://backend-5gsq.onrender.com/api/news/editnews/${id}`,
        formData,
        config
      );
      allContent.value.splice(editingIndex.value, 1, data);
    } else {
      await axios.post("https://backend-5gsq.onrender.com/api/news", formData, config);
    }
    await fetchContent();
    resetForm();
    page.value = 1;
  } catch (err) {
    console.error("Save error:", err);
  } finally {
    isLoading.value = false;
  }
};

// Toggle Banner
const toggleBanner = async () => {
  if (!showBanners.value) {
    isLoadingBanners.value = true;
    showBanners.value = true;
    await fetchBanners();
  } else {
    showBanners.value = false;
  }
};

const handleImage = async (e) => {
  const file = e.target.files[0];
  if (file) {
    isUploadingImage.value = true;
    try {
      form.value.image = file;
      // Any processing needed for image preview or validation
    } finally {
      isUploadingImage.value = false;
    }
  }
};

// Edit / Delete content
const editCard = (item) => {
  const index = allContent.value.findIndex((i) => i._id === item._id);
  editingIndex.value = index;
  form.value = {
    title: item.title,
    image: item.image,
    content: item.content,
    category: item.category?.name || item.category || "",
    author: item.author?.username || item.author || "",
  };
  searchQuery.value = form.value.category;
  showModal.value = true;
};

const deleteCard = (item) => {
  indexToDelete.value = item._id;
  showDeleteModal.value = true;
};

const confirmDelete = async () => {
  isDeleting.value = true;
  try {
    await axios.delete(
      `https://backend-5gsq.onrender.com/api/news/deletenews/${indexToDelete.value}`,
      {
        headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
      }
    );
    allContent.value = allContent.value.filter(
      (item) => item._id !== indexToDelete.value
    );
    showDeleteModal.value = false;
    indexToDelete.value = null;
  } catch (err) {
    console.error("Delete error:", err);
  } finally {
    isDeleting.value = false;
  }
};

const cancelDelete = () => {
  showDeleteModal.value = false;
  indexToDelete.value = null;
};

const cancelEdit = () => resetForm();

const resetForm = () => {
  showModal.value = false;
  editingIndex.value = null;
  form.value = { title: "", image: "", content: "", category: "", author: "" };
  searchQuery.value = "";
};

// Pagination
const page = ref(1);
const perPage = 12;

const filteredContent = computed(() => {
  if (categoryName.value.toLowerCase() === "all") return allContent.value;
  return allContent.value.filter((item) => {
    const cat =
      typeof item.category === "object" ? item.category.name : item.category;
    return cat?.toLowerCase() === categoryName.value.toLowerCase();
  });
});

const totalPages = computed(() =>
  Math.ceil(filteredContent.value.length / perPage)
);
const paginatedContent = computed(() => {
  const start = (page.value - 1) * perPage;
  return filteredContent.value.slice(start, start + perPage);
});

const changePage = async (newPage) => {
  if (newPage >= 1 && newPage <= totalPages.value) {
    isLoadingPage.value = true;
    try {
      page.value = newPage;
      // If you need to fetch data from API instead of using client-side pagination
      // await fetchPageData(newPage);
    } finally {
      setTimeout(() => {
        isLoadingPage.value = false;
      }, 300); // Small delay to show loading effect
    }
  }
};

// Dropdown click-outside close
const handleClickOutside = (event) => {
  setTimeout(() => {
    if (
      categoryDropdownRef.value &&
      !categoryDropdownRef.value.contains(event.target)
    ) {
      showCategoryList.value = false;
    }
  }, 0);
};

// ---------------------------
//        BANNER FUNCTIONS
// ---------------------------

// Open edit banner modal
const openEditBannerModal = (banner) => {
  currentBannerToEdit.value = {
    _id: banner._id,
    link: banner.link,
    image: banner.image, // URL for preview
    newFile: null, // new file for upload
  };
  isEditBannerModalVisible.value = true;
};

// Close edit modal
const closeEditBannerModal = () => {
  isEditBannerModalVisible.value = false;
  currentBannerToEdit.value = { image: "", link: "", _id: "" };
};

// Save banner edit
const saveBanner = async () => {
  isSavingBanner.value = true;
  try {
    const formData = new FormData();

    if (currentBannerToEdit.value.newFile instanceof File) {
      formData.append("image", currentBannerToEdit.value.newFile);
    }

    formData.append("link", currentBannerToEdit.value.link);

    const response = await axios.put(
      `https://backend-5gsq.onrender.com/api/banner/${currentBannerToEdit.value._id}`,
      formData,
      {
        headers: {
          "Content-Type": "multipart/form-data",
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
      }
    );

    console.log("Banner updated successfully:", response.data);
    await fetchBanners();
    closeEditBannerModal();
  } catch (error) {
    console.error("Error updating banner:", error);
  } finally {
    isSavingBanner.value = false;
  }
};

// Open delete banner modal
const openDeleteBannerModal = (banner) => {
  bannerToDeleteId.value = banner?._id;
  isDeleteBannerModalVisible.value = true;
};

// Close delete modal
const closeDeleteBannerModal = () => {
  bannerToDeleteId.value = null;
  isDeleteBannerModalVisible.value = false;
};

// Delete banner
const confirmDeleteBanner = async () => {
  console.log(bannerToDeleteId.value);
  isDeletingBanner.value = true;
  if (bannerToDeleteId.value && bannerToDeleteId.value) {
    try {
      await axios.delete(
        `https://backend-5gsq.onrender.com/api/banner/${bannerToDeleteId.value}`,
        {
          headers: { Authorization: `Bearer ${localStorage.getItem("token")}` },
        }
      );

      console.log("Banner deleted successfully");
      await fetchBanners();
      closeDeleteBannerModal();
    } catch (error) {
      console.error("Error deleting banner:", error);
    } finally {
      isDeletingBanner.value = false;
    }
  } else {
    console.error("Banner ID is undefined or missing");
    isDeletingBanner.value = false;
  }
};

onMounted(() => {
  fetchContent();
  document.addEventListener("click", handleClickOutside);
});

onBeforeUnmount(() => {
  document.removeEventListener("click", handleClickOutside);
});
</script>

<style>
/* Optional: Custom scroll styling */
::-webkit-scrollbar {
  width: 6px;
}

::-webkit-scrollbar-thumb {
  background-color: #b48d3e;
  border-radius: 4px;
}

/* Ensure the Quill editor toolbar displays properly */
.ql-toolbar {
  border-bottom: 1px solid #ddd !important;
  border-top-left-radius: 0.25rem;
  border-top-right-radius: 0.25rem;
}

.ql-container {
  min-height: 200px;
  max-height: 400px;
  overflow-y: auto;
  border-bottom-left-radius: 0.25rem;
  border-bottom-right-radius: 0.25rem;
}

/* Fix for Quill editor container */
.quill-editor {
  display: flex;
  flex-direction: column;
}
</style>