<script setup lang="ts">
import { onMounted, ref, watch } from 'vue';

import type { HerbSummary } from '@/services/herbs-service';

import HerbChart from '@/components/HerbChart.vue';
import HerbTable from '@/components/HerbTable.vue';
import DashboardLayout from '@/components/layout/DashboardLayout.vue';
import SummaryCards from '@/components/SummaryCards.vue';
import herbsService from '@/services/herbs-service';

// ฟังก์ชันคำนวณปีงบประมาณปัจจุบัน (พ.ศ.)
function getCurrentThaiFiscalYear(): number {
  const today = new Date();
  const month = today.getMonth(); // 0-11
  const yearAD = today.getFullYear();

  // ถ้าเดือน >= ต.ค. (9) ให้ถือเป็นปีงบประมาณหน้า (AD)
  const fiscalYearAD = month >= 9 ? yearAD + 1 : yearAD;

  // แปลงเป็น พ.ศ. (+543)
  return fiscalYearAD + 543;
}

// State
const currentThaiYear = getCurrentThaiFiscalYear();
const selectedYear = ref<number>(currentThaiYear);
const summaryData = ref<HerbSummary | null>(null);
const loading = ref<boolean>(true);
const error = ref<string | null>(null);

// สร้างรายการปี พ.ศ. ย้อนหลัง 5 ปี (เช่น 2568, 2567, 2566...)
const availableYears = Array.from({ length: 5 }, (_, i) => currentThaiYear - i);

async function fetchData() {
  loading.value = true;
  error.value = null;
  try {
    // ส่งปี พ.ศ. ไปที่ Backend
    const data = await herbsService.getHerbSummary(selectedYear.value);
    summaryData.value = data;
  }
  catch (err) {
    error.value = err instanceof Error ? err.message : 'An unexpected error occurred';
    console.error(err);
  }
  finally {
    loading.value = false;
  }
}

onMounted(() => {
  fetchData();
});

watch(selectedYear, () => {
  fetchData();
});
</script>

<template>
  <DashboardLayout>
    <template #controls>
      <div class="flex items-center gap-3">
        <label for="year-select" class="text-sm font-medium text-text-secondary hidden sm:block">
          ปีงบประมาณ (พ.ศ.):
        </label>
        <div class="relative group">
          <select
            id="year-select"
            v-model="selectedYear"
            class="appearance-none bg-surface border border-border text-text-primary text-sm rounded-lg focus:ring-2 focus:ring-primary/20 focus:border-primary block w-full py-2 pl-4 pr-10 cursor-pointer font-medium shadow-sm hover:border-primary/50 transition-colors"
          >
            <option v-for="year in availableYears" :key="year" :value="year">
              {{ year }}
            </option>
          </select>
          <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-3 text-text-muted group-hover:text-primary transition-colors">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
            </svg>
          </div>
        </div>
      </div>
    </template>

    <div v-if="loading" class="flex flex-col items-center justify-center h-96">
      <div class="relative">
        <div class="w-12 h-12 rounded-full border-4 border-border-muted" />
        <div class="absolute inset-0 w-12 h-12 rounded-full border-4 border-transparent border-t-primary animate-spin" />
      </div>
      <p class="mt-4 text-text-muted font-medium text-sm animate-pulse">
        กำลังโหลดข้อมูล...
      </p>
    </div>

    <div
      v-else-if="error"
      class="bg-error-bg border border-error/20 text-error rounded-2xl p-8 text-center shadow-sm max-w-md mx-auto mt-10 animate-fade-in"
    >
      <div class="w-12 h-12 mx-auto mb-4 bg-error/10 rounded-full flex items-center justify-center">
        <svg class="w-6 h-6 text-error" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
        </svg>
      </div>
      <h3 class="font-bold text-lg mb-2 text-text-primary">
        เกิดข้อผิดพลาด
      </h3>
      <p class="text-text-secondary mb-6 text-sm">
        {{ error }}
      </p>
      <button
        class="inline-flex items-center gap-2 px-4 py-2 bg-white border border-border hover:bg-bg-alt text-text-primary rounded-lg text-sm font-medium transition-colors shadow-sm"
        @click="fetchData"
      >
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
        </svg>
        ลองใหม่อีกครั้ง
      </button>
    </div>

    <template v-else-if="summaryData">
      <SummaryCards :summary="summaryData" />

      <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <div class="h-full">
          <HerbChart :herbs="summaryData.herbs" />
        </div>
        <div class="h-full">
          <HerbTable :herbs="summaryData.herbs" />
        </div>
      </div>
    </template>
  </DashboardLayout>
</template>
