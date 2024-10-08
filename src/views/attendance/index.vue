<script setup>
import MainLayout from "@/components/MainLayout.vue";
import { DeleteOutlined, EditOutlined, PlusOutlined } from "@ant-design/icons-vue";
import attendance from "@/stores/attendance-api.js";
import { onMounted, ref } from "vue";

const isLoading = ref(false);
const attendanceData = ref([]);
const allAttendance = ref({});
const page = ref(1);
const paginate = ref(10);

const deleteAttendance = async (index) => {
  const attendanceId = attendanceData.value[index].id;
  try {
    await attendance.deleteAttendance(attendanceId);
    attendanceData.value.splice(index, 1);
    allAttendance.value.total -= 1;
  } catch (error) {
    console.error("Failed to delete attendance:", error);
  }
};

const getAllAttendance = async () => {
  isLoading.value = true;
  try {
    const response = await attendance.fetchAttendanceList(page.value, paginate.value);
    allAttendance.value = response.data;
    attendanceData.value = response.data.data;
  } catch (error) {
    console.error("Failed to fetch attendance:", error);
  } finally {
    isLoading.value = false;
  }
};

const attendanceSearch = async (input) => {
  if (input) {
    try {
      const response = await attendance.searchAttendanceList(input);
      allAttendance.value = response.data;
      attendanceData.value = response.data.data;
    } catch (error) {
      console.error("Failed to search attendance:", error);
    }
  } else {
    getAllAttendance();
  }
};

const handlePagination = (pageNo) => {
  page.value = pageNo;
  getAllAttendance();
};

onMounted(() => {
  getAllAttendance();
});
</script>

<template>
  <MainLayout>
    <div class="flex justify-between mb-4">
      <input
        type="text"
        placeholder="Search Attendance..."
        class="px-4 py-2 border rounded"
        @input="attendanceSearch($event?.target?.value)"
      />
      <router-link :to="{ name: 'attendance-create' }">
        <button class="flex items-center px-4 py-2 bg-[#000180] text-white rounded hover:bg-indigo-600">
          <PlusOutlined class="mr-2" />
          New Attendance
        </button>
      </router-link>
    </div>
    <h6 class="font-medium">Attendance ({{ allAttendance?.total || 0 }})</h6>
    <table class="table border-collapse border border-slate-400 w-full bg-white my-4">
      <thead class="table-header">
        <tr>
          <th>Actions</th>
          <th class="text-left font-bold">SL.</th>
          <th class="text-center">Date</th>
          <th class="text-center">Employee</th>
          <th class="text-center">Designation</th>
          <th class="text-center">Branch</th>
          <th class="text-center">Check In</th>
          <th class="text-center">Check Out</th>
          <th class="text-center">Hours</th>
          <th class="text-center">Late Time</th>
          <th class="text-center">Over Time</th>
          <th class="text-center">Status</th>
        </tr>
      </thead>
      <tbody class="table-body">
        <tr v-if="isLoading">
          <td colspan="4" class="text-red-600">Loading . . .</td>
        </tr>
        <tr v-if="!isLoading && !attendanceData?.length">
          <td colspan="4" class="text-red-600">No Attendance Found . . .</td>
        </tr>
        <tr
          v-for="(attendance, index) in attendanceData"
          :key="index"
          class="hover:bg-gray-100 transition-colors duration-200"
        >
          <td class="text-center w-16">
            <button
              type="button"
              class="edit_btn"
              @click="$router.push({ name: 'attendance-edit', params: { id: attendance?.id } })"
            >
              <EditOutlined class="align-middle" />
            </button>
            <a-popconfirm
              title="Are you sure you want to delete?"
              @confirm="deleteAttendance(index)"
            >
              <button type="button" class="del_btn ml-2">
                <DeleteOutlined class="align-middle" />
              </button>
            </a-popconfirm>
          </td>
          <td class="font-bold">{{ allAttendance?.from + index }}</td>
          <td class="text-center">{{ attendance.date || '-' }}</td>
          <td class="text-center">{{ attendance.name || '-' }}</td>
          <td class="text-center">{{ attendance.designation || '-' }}</td>
          <td class="text-center">{{ attendance.branch || '-' }}</td>
          <td class="text-center">{{ attendance.check_in_time || '-' }}</td>
          <td class="text-center">{{ attendance.check_out_time || '-' }}</td>
          <td class="text-center">{{ attendance.hours || '-' }}</td>
          <td class="text-center">{{ attendance.late_time || '-' }}</td>
          <td class="text-center">{{ attendance.over_time || '-' }}</td>
          <td class="text-center">
            <button
              :class="[
                attendance.status === 1 ? 'bg-green-500' : 
                attendance.status === 0 ? 'bg-red-500' : 'bg-gray-500',
                'text-white font-bold py-1 px-3 rounded text-sm'
              ]"
            >
              {{ attendance.status === 1 ? 'Present' : 
                 attendance.status === 0 ? 'Absent' : '-' }}
            </button>
          </td>
        </tr>
      </tbody>
    </table>
    <a-pagination
      v-model:current="page"
      v-model:page-size="paginate"
      :total="allAttendance?.total"
      :show-total="(total) => `Total ${total} attendance`"
      @change="handlePagination"
    />
  </MainLayout>
</template>
