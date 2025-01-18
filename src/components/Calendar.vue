<template>
  <div class="max-w-screen-xl flex flex-col items-center">
    <div class="flex flex-row justify-center gap-8">
      <div class="bg-gray-100 p-4 rounded shadow">
        <div class="flex justify-between w-full max-w-md mb-4">
          <button @click="prevMonth" class="btn">&lt;</button>
          <h2 class="text-lg font-bold text-gray-700">{{ currentMonthYear }}</h2>
        </div>

        <div class="calendar-body flex flex-col items-center">
          <div class="grid grid-cols-7 gap-2">
            <div v-for="day in daysOfWeek" :key="day" class="uppercase font-bold text-center text-gray-600">
              {{ day }}
            </div>

            <div v-for="(day, index) in daysInMonth" :key="index"
                 :class="['flex justify-center items-center border rounded cursor-pointer hover:bg-blue-200 hover:shadow-lg transition-colors', isSelected(day) ? 'bg-blue-500 text-white' : 'bg-white text-gray-700', isToday(day) ? 'bg-blue-100' : '']"
                 @click="selectDay(day)">
              {{ day.date }}
            </div>
          </div>
        </div>
      </div>

      <div class="bg-gray-100 p-4 rounded shadow">
        <div class="flex justify-between w-full max-w-md mb-4">
          <h2 class="text-lg font-bold text-gray-700">{{ nextMonthYear }}</h2>
          <button @click="nextMonthHandler" class="btn">&gt;</button>
        </div>

        <div class="calendar-body flex flex-col items-center">
          <div class="grid grid-cols-7 gap-2">
            <div v-for="day in daysOfWeek" :key="day" class="uppercase font-bold text-center text-gray-600">
              {{ day }}
            </div>

            <div v-for="(day, index) in nextMonthDays" :key="index"
                 :class="['flex justify-center items-center border rounded cursor-pointer hover:bg-blue-200 hover:shadow-lg transition-colors', isSelected(day) ? 'bg-blue-500 text-white' : 'bg-white text-gray-700', isToday(day) ? 'bg-blue-100' : '']"
                 @click="selectDay(day)">
              {{ day.date }}
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="mt-4">
      <h3 class="text-lg font-semibold text-gray-700">Favorite Terms</h3>
      <ul class="mt-2">
        <li v-for="term in favoriteTerms" :key="term.label">
          <button
              class="cursor-pointer hover:underline text-gray-600" @click="selectFavorite(term.range)"
          >
            {{ term.label }}
          </button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import {ref, computed} from 'vue';

const getDaysInMonth = (year: number, month: number) => {
  const days: { date: number, fullDate: Date }[] = [];
  const date = new Date(year, month, 1);
  while (date.getMonth() === month) {
    days.push({date: date.getDate(), fullDate: new Date(date)});
    date.setDate(date.getDate() + 1);
  }
  return days;
};

const currentDate = ref(new Date());
const selectedRange = ref<{ start: Date | null; end: Date | null }>({start: null, end: null});

const daysOfWeek = ['Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa', 'Su'];
const favoriteTerms = ref([
  {label: 'Last Minute', range: {start: new Date(), end: new Date(new Date().setDate(new Date().getDate() + 7))}},
  {label: 'Summer Vacation', range: {start: new Date(2025, 5, 1), end: new Date(2025, 7, 31)}}
]);

const currentMonth = ref(currentDate.value.getMonth());
const currentYear = ref(currentDate.value.getFullYear());

const daysInMonth = computed(() => getDaysInMonth(currentYear.value, currentMonth.value));
const currentMonthYear = computed(() => {
  return new Date(currentYear.value, currentMonth.value).toLocaleString('default', {month: 'long', year: 'numeric'});
});

const nextMonth = computed(() => (currentMonth.value + 1) % 12);
const nextYear = computed(() => (currentMonth.value === 11 ? currentYear.value + 1 : currentYear.value));
const nextMonthDays = computed(() => getDaysInMonth(nextYear.value, nextMonth.value));
const nextMonthYear = computed(() => {
  return new Date(nextYear.value, nextMonth.value).toLocaleString('default', {month: 'long', year: 'numeric'});
});

const prevMonth = () => {
  if (currentMonth.value === 0) {
    currentMonth.value = 11;
    currentYear.value -= 1;
  } else {
    currentMonth.value -= 1;
  }
};

const nextMonthHandler = () => {
  if (currentMonth.value === 11) {
    currentMonth.value = 0;
    currentYear.value += 1;
  } else {
    currentMonth.value += 1;
  }
};

const selectDay = (day: { date: number; fullDate: Date }) => {
  if (!selectedRange.value.start) {
    selectedRange.value.start = day.fullDate;
  } else if (!selectedRange.value.end) {
    selectedRange.value.end = day.fullDate;
  } else {
    selectedRange.value = {start: day.fullDate, end: null};
  }
};

const isSelected = (day: { date: number; fullDate: Date }) => {
  if (!selectedRange.value.start) return false;
  if (!selectedRange.value.end) return day.fullDate.getTime() === selectedRange.value.start.getTime();
  return (
      day.fullDate.getTime() >= selectedRange.value.start.getTime() &&
      day.fullDate.getTime() <= selectedRange.value.end.getTime()
  );
};

const isToday = (day: { date: number; fullDate: Date }) => {
  const today = new Date();
  return (
      day.fullDate.getDate() === today.getDate() &&
      day.fullDate.getMonth() === today.getMonth() &&
      day.fullDate.getFullYear() === today.getFullYear()
  );
};

const selectFavorite = (range: { start: Date; end: Date }) => {
  selectedRange.value = {...range};
};
</script>

<style scoped>
.btn {
  background-color: #007bff;
  color: white;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 0.25rem;
  cursor: pointer;
}

.btn:hover {
  background-color: #0056b3;
}

.calendar-body {
  background-color: #f9f9f9;
  padding: 1rem;
  border-radius: 0.5rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
</style>
