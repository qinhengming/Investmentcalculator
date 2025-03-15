<script setup>
import { ref, computed, watch } from "vue";

const props = defineProps({
  calculatorData: {
    type: Object,
    required: true,
  },
});

// 当前显示的年月
const currentYear = ref(new Date().getFullYear());
const currentMonth = ref(new Date().getMonth());

// 计算当前月份的天数
const daysInMonth = computed(() => {
  return new Date(currentYear.value, currentMonth.value + 1, 0).getDate();
});

// 计算当前月份第一天是星期几 (0-6)
const firstDayOfMonth = computed(() => {
  return new Date(currentYear.value, currentMonth.value, 1).getDay();
});

// 计算日历网格需要的单元格数量
const calendarCells = computed(() => {
  const cells = [];
  const totalCells = firstDayOfMonth.value + daysInMonth.value;
  const rows = Math.ceil(totalCells / 7);

  // 填充空白单元格
  for (let i = 0; i < firstDayOfMonth.value; i++) {
    cells.push({ day: null, value: null });
  }

  // 填充日期单元格
  for (let day = 1; day <= daysInMonth.value; day++) {
    cells.push({
      day,
      value: calculateDailyValue(day),
    });
  }

  // 补充剩余空白单元格，使其成为完整的网格
  const remainingCells = rows * 7 - cells.length;
  for (let i = 0; i < remainingCells; i++) {
    cells.push({ day: null, value: null });
  }

  return cells;
});

// 计算每日收益
// 计算每日收益
function calculateDailyValue(day) {
  const { principal, rate, compoundPeriod } = props.calculatorData;

  if (!principal || !rate) return 0;

  // 根据计息周期调整收益率
  let adjustedRate;
  let dailyRate;

  switch (compoundPeriod) {
    case "daily":
      dailyRate = rate / 100;
      break;
    case "monthly":
      adjustedRate = rate / 30 / 100; // 将年收益率转换为月收益率
      dailyRate = adjustedRate; // 简化处理，每月按30天计算
      break;
    case "yearly":
    default:
      adjustedRate = rate / 365 / 100;
      dailyRate = adjustedRate;
      break;
  }

  // 计算到当前日期的累计天数
  const dayOfYear = getDayOfYear(currentYear.value, currentMonth.value, day);

  // 对于今天之前的日期，返回本金
  if (dayOfYear < 0) {
    return principal;
  }
  // 根据复利公式计算
  let value;
  if (compoundPeriod === "daily") {
    // 每日复利
    value = principal * Math.pow(1 + dailyRate, dayOfYear);
  } else if (compoundPeriod === "monthly") {
    // 每月复利
    const monthsPassed = currentMonth.value;
    const daysInCurrentMonth = day;
    value =
      principal *
      Math.pow(1 + adjustedRate, monthsPassed) *
      (1 + dailyRate * daysInCurrentMonth);
  } else {
    // 每年复利 (简化处理，按日计算线性增长)
    value = principal * (1 + dailyRate * dayOfYear);
  }

  return value;
}

// 计算一年中的第几天
function getDayOfYear(year, month, day) {
  const date = new Date(year, month, day);
  const today = new Date();
  today.setHours(0, 0, 0, 0); // 将时间设置为当天的 0 点
  const diff = date - today;
  return Math.floor(diff / (1000 * 60 * 60 * 24));
}

// 计算收益颜色 (根据收益率生成从浅绿到深绿的渐变)
function getValueColor(value) {
  if (!value) return "transparent";

  const principal = props.calculatorData.principal;
  if (!principal) return "transparent";

  // 计算收益率
  const growthRate = (value - principal) / principal;

  // 将收益率映射到颜色深度 (0-100)
  const colorIntensity = Math.min(Math.max(growthRate * 1000, 0), 100);

  // 生成颜色 (从浅绿到深绿)
  return `rgba(66, 184, 131, ${0.1 + (colorIntensity / 100) * 0.9})`;
}

// 格式化货币
function formatCurrency(value) {
  if (!value) return "";
  const lookup = [
    { value: 1e12, symbol: "T" },
    { value: 1e9, symbol: "B" },
    { value: 1e6, symbol: "M" },
    { value: 1e3, symbol: "K" },
  ];

  const item = lookup.find((item) => value >= item.value);
  if (item) {
    return (value / item.value).toFixed(2) + item.symbol;
  }

  return value.toFixed(2);
}

// 月份名称
const monthNames = [
  "一月",
  "二月",
  "三月",
  "四月",
  "五月",
  "六月",
  "七月",
  "八月",
  "九月",
  "十月",
  "十一月",
  "十二月",
];

// 切换到上一个月
function previousMonth() {
  if (currentMonth.value === 0) {
    currentMonth.value = 11;
    currentYear.value--;
  } else {
    currentMonth.value--;
  }
}

// 切换到下一个月
function nextMonth() {
  if (currentMonth.value === 11) {
    currentMonth.value = 0;
    currentYear.value++;
  } else {
    currentMonth.value++;
  }
}

// 切换到当前月份
function goToCurrentMonth() {
  const now = new Date();
  currentYear.value = now.getFullYear();
  currentMonth.value = now.getMonth();
}
</script>

<template>
  <div class="calendar-view">
    <div class="calendar-header">
      <button @click="previousMonth" class="nav-button">&lt;</button>
      <h3>{{ monthNames[currentMonth] }} {{ currentYear }}</h3>
      <button @click="nextMonth" class="nav-button">&gt;</button>
      <button @click="goToCurrentMonth" class="today-button">今天</button>
    </div>

    <div class="weekdays">
      <div class="weekday">日</div>
      <div class="weekday">一</div>
      <div class="weekday">二</div>
      <div class="weekday">三</div>
      <div class="weekday">四</div>
      <div class="weekday">五</div>
      <div class="weekday">六</div>
    </div>

    <div class="calendar-grid">
      <div
        v-for="(cell, index) in calendarCells"
        :key="index"
        class="calendar-cell"
        :class="{ 'empty-cell': !cell.day }"
        :style="{ backgroundColor: getValueColor(cell.value) }"
      >
        <template v-if="cell.day">
          <div class="day-number">{{ cell.day }}</div>
          <div class="day-value">¥{{ formatCurrency(cell.value) }}</div>
        </template>
      </div>
    </div>

    <div class="color-legend">
      <div class="legend-item">
        <div
          class="color-box"
          style="background-color: rgba(66, 184, 131, 0.1)"
        ></div>
        <span>低收益</span>
      </div>
      <div class="legend-item">
        <div
          class="color-box"
          style="background-color: rgba(66, 184, 131, 0.5)"
        ></div>
        <span>中等收益</span>
      </div>
      <div class="legend-item">
        <div
          class="color-box"
          style="background-color: rgba(66, 184, 131, 1)"
        ></div>
        <span>高收益</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.calendar-view {
  background-color: white;
  border-radius: 8px;
  padding: clamp(10px, 3vw, 20px);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.calendar-header {
  display: flex;
  align-items: center;
  margin-bottom: 15px;
  gap: 8px;
}

.calendar-header h3 {
  margin: 0 15px;
  flex-grow: 1;
  text-align: center;
  font-size: clamp(14px, 4vw, 18px);
}

.nav-button,
.today-button {
  background-color: #f5f5f5;
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: clamp(4px, 2vw, 10px);
  cursor: pointer;
  font-size: clamp(12px, 3vw, 14px);
  min-width: 32px;
}

.nav-button:hover,
.today-button:hover {
  background-color: #e0e0e0;
}

.today-button {
  margin-left: 10px;
  background-color: #42b883;
  color: white;
  border: none;
}

.today-button:hover {
  background-color: #3aa876;
}

.weekdays {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  margin-bottom: 10px;
}

.weekday {
  text-align: center;
  font-weight: bold;
  color: #555;
  font-size: clamp(12px, 3vw, 14px);
  padding: clamp(2px, 1vw, 5px);
}

.calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: clamp(2px, 1vw, 5px);
}

.calendar-cell {
  aspect-ratio: 1;
  border: 1px solid #eee;
  border-radius: 4px;
  padding: clamp(2px, 1vw, 5px);
  display: flex;
  flex-direction: column;
  transition: transform 0.2s;
}

.calendar-cell:hover {
  transform: scale(1.05);
  z-index: 1;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}

.empty-cell {
  background-color: transparent;
  border: none;
}

.day-number {
  font-size: clamp(12px, 3vw, 14px);
  margin-bottom: clamp(2px, 1vw, 5px);
  font-weight: bold;
}

.day-value {
  font-size: clamp(10px, 2.5vw, 12px);
  overflow-wrap: break-word;
  word-break: break-all;
  word-wrap: break-word;
  white-space: normal;
  line-height: 1.2;
  max-height: 70%;
  overflow-y: auto;
}

.color-legend {
  display: flex;
  justify-content: center;
  margin-top: 20px;
  gap: 10px;
}

.legend-item {
  display: flex;
  align-items: center;
  font-size: clamp(12px, 3vw, 14px);
}

.color-box {
  width: 20px;
  height: 20px;
  border-radius: 4px;
  margin-right: 5px;
}
@media (max-width: 480px) {
  .calendar-header {
    margin-bottom: 10px;
  }
  
  .today-button {
    padding: 4px 8px;
  }
  
  .color-legend {
    justify-content: space-around;
  }
}
</style>
