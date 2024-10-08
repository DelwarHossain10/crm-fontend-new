<template>
  <MainLayout>
    <div class="bg-white p-3 rounded-md">
      <div class="flex justify-between items-center">
        <div class="mb-3">
          <h6 class="font-medium">Create New Division</h6>
        </div>
        <div class="mb-3">
          <button
            type="button"
            class="px-4 py-2 bg-[#000180] text-white rounded hover:bg-indigo-600"
            @click="$router.go(-1)"
          >
            Back
          </button>
        </div>
      </div>
      <hr />
      <form @submit.prevent="submitForm">
        <div class="lg:grid grid-cols-3 gap-4 items-center">
          <!-- Division Name -->
          <label for="name"
            >Division Name <span class="text-red-600">*</span></label
          >
          <div class="col-span-2">
            <input
              id="name"
              type="text"
              placeholder="Enter here . . ."
              v-model="form.name"
              :class="{ 'border-red-500': formErrors.name }"
              class="input-text w-full"
            />
            <p v-if="formErrors.name" class="text-red-500">{{ formErrors.name }}</p>
          </div>

          <!-- Status -->
          <label for="status">Status</label>
          <div class="col-span-2">
            <select
              v-model="form.status"
              id="status"
              class="common-select w-full rounded-lg"
            >
              <option value="1">Active</option>
              <option value="0">Inactive</option>
            </select>
          </div>

          <!-- Submit Button -->
          <div class="col-span-3 flex justify-end mt-3">
            <button
              type="submit"
              :disabled="loading"
              class="px-4 py-2 min-w-32 bg-[#000180] text-white rounded-lg hover:bg-indigo-600"
            >
              {{ loading ? 'Submitting...' : 'Submit' }}
            </button>
          </div>
        </div>
      </form>
    </div>
  </MainLayout>
</template>

<script setup>
import MainLayout from "@/components/MainLayout.vue";
import division from "@/stores/setting/division-api.js";
import { showNotification } from "@/utilities/notification";
import { ref } from "vue";
import { useRouter } from "vue-router";

const loading = ref(false);

const form = ref({
  name: "",
  status: "1", // Default value for status set to "Active"
});

const formErrors = ref({});

// Form Validation
function validateForm() {
  const errors = {};
  if (!form.value.name) errors.name = "Division Name is required";
  formErrors.value = errors;
  return Object.keys(errors).length === 0;
}

const router = useRouter();

// Submit Form
const submitForm = async () => {
  if (validateForm()) {
    loading.value = true;
    try {
      const response = await division.insertDivision(form.value);

      if (response?.status === 201) {
        showNotification(
          "success",
          response?.data?.message || "Division successfully created."
        );
        form.value = { name: "", status: "1" }; // Reset form after success
        router.push({ name: "division" });
      }
    } catch (error) {
      if (error.response) {
        showNotification(
          "error",
          error.response.data.message || "Failed to create division."
        );
      } else if (error.request) {
        showNotification("error", "Network error, please try again later.");
      }
    } finally {
      loading.value = false;
    }
  }
};
</script>
