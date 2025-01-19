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
            <div
                v-for="(day, index) in daysInMonth"
                :key="index"
                :class="[
                'flex justify-center items-center border rounded cursor-pointer hover:bg-blue-200 hover:shadow-lg transition-colors',
                isInRange(day) && day.isCurrentMonth ? 'bg-blue-500 text-white' : 'bg-white text-gray-700',
                isTodayDay(day) && day.isCurrentMonth ? 'bg-blue-100' : '',
                isStartDate(day) && day.isCurrentMonth ? 'bg-blue-500 text-white' : '',
                isDifferentMonth(day) ? 'text-gray-400 cursor-not-allowed hover:bg-white' : ''
              ]"
                @click="!isDifferentMonth(day) && selectDay(day)"
            >
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
            <div
                v-for="(day, index) in nextMonthDays"
                :key="index"
                :class="[
                'flex justify-center items-center border rounded cursor-pointer hover:bg-blue-200 hover:shadow-lg transition-colors',
                isInRange(day) && day.isCurrentMonth ? 'bg-blue-500 text-white' : 'bg-white text-gray-700',
                isTodayDay(day) && day.isCurrentMonth ? 'bg-blue-100' : '',
                isStartDate(day) && day.isCurrentMonth ? 'bg-blue-500 text-white' : '',
                isDifferentMonth(day) ? 'text-gray-400 cursor-not-allowed hover:bg-white' : ''
              ]"
                @click="!isDifferentMonth(day) && selectDay(day)"
            >
              {{ day.date }}
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="mt-4">
      <h3 class="text-lg font-semibold text-gray-700">Oblíbené termíny</h3>
      <ul class="mt-2">
        <li v-for="term in favoriteTerms" :key="term.label">
          <label class="flex items-center gap-2 cursor-pointer">
            <input
                type="radio"
                name="favorite-term"
                class="form-radio text-blue-500"
                @change="selectFavorite(term.range)"
            />
            <span class="text-gray-600">{{ term.label }}</span>
          </label>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import {
  startOfMonth,
  endOfMonth,
  startOfWeek,
  addDays,
  eachDayOfInterval,
  format,
  isToday,
  isSameDay,
  isWithinInterval,
} from 'date-fns';
import { cs } from 'date-fns/locale';

const weekStartsOn = 1;

const maxRange = ref(30);
const minDate = ref(new Date());
const locale = ref(cs); // Используем объект локализации
const preselectedRange = ref<{ start: Date | null; end: Date | null }>({ start: null, end: null });
const favoriteTerms = ref([
  { label: 'Last Minute', range: { start: new Date(), end: addDays(new Date(), 7) } },
  { label: 'Listopad / Prosinec 2024', range: { start: new Date(2024, 10, 1), end: new Date(2024, 11, 31) } },
  { label: 'Leden / Únor 2025', range: { start: new Date(2025, 0, 1), end: new Date(2025, 1, 28) } },
  { label: 'Březen / Duben 2025', range: { start: new Date(2025, 2, 1), end: new Date(2025, 3, 30) } },
  { label: 'Květen / Červen 2025', range: { start: new Date(2025, 4, 1), end: new Date(2025, 5, 30) } },
  { label: 'Letní prázdniny 2025', range: { start: new Date(2025, 6, 1), end: new Date(2025, 7, 31) } },
]);

const currentDate = ref(new Date());
const currentMonth = ref(currentDate.value.getMonth());
const currentYear = ref(currentDate.value.getFullYear());

const selectedRange = ref<{ start: Date | null; end: Date | null }>({
  start: preselectedRange.value.start,
  end: preselectedRange.value.end,
});

const daysOfWeek = computed(() => {
  const weekStart = startOfWeek(new Date(), { weekStartsOn });
  return Array.from({ length: 7 }, (_, i) => format(addDays(weekStart, i), 'EEEEEE', { locale: locale.value }));
});

const daysInMonth = computed(() => {
  const start = startOfWeek(startOfMonth(new Date(currentYear.value, currentMonth.value)), { weekStartsOn });
  const end = endOfMonth(new Date(currentYear.value, currentMonth.value));
  return eachDayOfInterval({ start, end }).map((date) => ({
    date: format(date, 'd'),
    fullDate: date,
    isCurrentMonth: date.getMonth() === currentMonth.value,
  }));
});

const nextMonth = computed(() => (currentMonth.value + 1) % 12);
const nextYear = computed(() => (currentMonth.value === 11 ? currentYear.value + 1 : currentYear.value));
const nextMonthDays = computed(() => {
  const start = startOfWeek(startOfMonth(new Date(nextYear.value, nextMonth.value)), { weekStartsOn });
  const end = endOfMonth(new Date(nextYear.value, nextMonth.value));
  return eachDayOfInterval({ start, end }).map((date) => ({
    date: format(date, 'd'),
    fullDate: date,
    isCurrentMonth: date.getMonth() === nextMonth.value,
  }));
});

const currentMonthYear = computed(() =>
    format(new Date(currentYear.value, currentMonth.value), 'MMMM yyyy', { locale: locale.value })
);
const nextMonthYear = computed(() =>
    format(new Date(nextYear.value, nextMonth.value), 'MMMM yyyy', { locale: locale.value })
);

const isTodayDay = (day: { date: string; fullDate: Date }) => isToday(day.fullDate);
const isStartDate = (day: { date: string; fullDate: Date }) =>
    selectedRange.value.start ? isSameDay(day.fullDate, selectedRange.value.start) : false;
const isInRange = (day: { date: string; fullDate: Date }) => {
  if (!selectedRange.value.start || !selectedRange.value.end) return false;
  return isWithinInterval(day.fullDate, {
    start: selectedRange.value.start,
    end: selectedRange.value.end,
  });
};
const isDifferentMonth = (day: { isCurrentMonth: boolean }) => !day.isCurrentMonth;

const selectDay = (day: { date: string; fullDate: Date }) => {
  if (!selectedRange.value.start) {
    selectedRange.value.start = day.fullDate;
  } else if (!selectedRange.value.end) {
    const diff = (day.fullDate.getTime() - selectedRange.value.start.getTime()) / (1000 * 60 * 60 * 24);
    if (diff <= maxRange.value) {
      selectedRange.value.end = day.fullDate;
    }
  } else {
    selectedRange.value = { start: day.fullDate, end: null };
  }
};

const selectFavorite = (range: { start: Date; end: Date }) => {
  selectedRange.value = { ...range };
  const startMonth = range.start.getMonth();
  const startYear = range.start.getFullYear();

  currentMonth.value = startMonth;
  currentYear.value = startYear;
};

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

.bg-blue-500 {
  background-color: #007bff !important;
}

.text-white {
  color: #fff !important;
}

.text-gray-400 {
  color: #a0aec0;
}

.cursor-not-allowed {
  cursor: not-allowed;
}
</style>
