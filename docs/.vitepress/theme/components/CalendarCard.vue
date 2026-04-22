<template>
  <TkPageCard>
    <div class="card-widget" id="card-widget-calendar">
      <div class="item-headline">
        <i class="icon-calendar"></i>
      </div>
      <div class="item-content">
        <div id="calendar-area-left">
          <div id="calendar-week">
            第{{ weekNumber }}周&nbsp;{{ weekDays[today.getDay()] }}
          </div>
          <div id="calendar-date">{{ today.getDate() }}</div>
          <div id="calendar-solar">
            {{ today.getFullYear() }}年{{ today.getMonth() + 1 }}月第{{
              dayOfYear
            }}天
          </div>
          <div id="calendar-lunar">
            {{ lunarYear }}&nbsp;{{ lunarMonth }}&nbsp;{{ lunarDay }}
          </div>
        </div>
        <div id="calendar-area-right">
          <div id="calendar-main">
            <!-- 星期 -->
            <div class="calendar-r0">
              <div class="calendar-d0"><a>日</a></div>
              <div class="calendar-d1"><a>一</a></div>
              <div class="calendar-d2"><a>二</a></div>
              <div class="calendar-d3"><a>三</a></div>
              <div class="calendar-d4"><a>四</a></div>
              <div class="calendar-d5"><a>五</a></div>
              <div class="calendar-d6"><a>六</a></div>
            </div>

            <!-- 日期 -->
            <div
              v-for="(week, weekIndex) in calendarWeeks"
              :key="weekIndex"
              :class="`calendar-r${weekIndex + 1}`"
            >
              <div
                v-for="(day, dayIndex) in week"
                :key="dayIndex"
                :class="`calendar-d${dayIndex}`"
              >
                <a
                  v-if="day.date"
                  :class="{ now: day.isToday, 'other-month': day.isOtherMonth }"
                >
                  {{ day.date }}
                </a>
                <a v-else></a>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </TkPageCard>
</template>

<script setup>
import { TkPageCard } from "vitepress-theme-teek";
import { ref, onMounted, onUnmounted, computed } from "vue";

const weekDays = ["周日", "周一", "周二", "周三", "周四", "周五", "周六"];

/** 当前时间 */
const today = ref(new Date());

/** ===== 日历 ===== */
const calendarWeeks = computed(() => {
  const year = today.value.getFullYear();
  const month = today.value.getMonth();

  const firstDay = new Date(year, month, 1);
  const lastDay = new Date(year, month + 1, 0);

  const startDay = new Date(firstDay);
  startDay.setDate(firstDay.getDate() - firstDay.getDay());

  const endDay = new Date(lastDay);
  if (endDay.getDay() < 6) {
    endDay.setDate(lastDay.getDate() + (6 - endDay.getDay()));
  }

  const weeks = [];
  let currentDay = new Date(startDay);

  while (currentDay <= endDay) {
    const week = [];
    for (let i = 0; i < 7; i++) {
      const date = currentDay.getDate();
      const isToday =
        currentDay.toDateString() === today.value.toDateString();
      const isOtherMonth = currentDay.getMonth() !== month;

      week.push({ date, isToday, isOtherMonth });
      currentDay.setDate(currentDay.getDate() + 1);
    }
    weeks.push(week);
  }

  return weeks;
});

const dayOfYear = computed(() => {
  const start = new Date(today.value.getFullYear(), 0, 0);
  return Math.floor((today.value - start) / 86400000);
});

const weekNumber = computed(() => {
  const firstDay = new Date(today.value.getFullYear(), 0, 1);
  const pastDays = (today.value - firstDay) / 86400000;
  return Math.ceil((pastDays + firstDay.getDay() + 1) / 7);
});

/** ===== 农历（保持你原逻辑） ===== */
const lunarYear = ref("");
const lunarMonth = ref("");
const lunarDay = ref("");

const updateLunar = () => {
  const d = today.value;
  lunarYear.value = d.getFullYear() + "年";
  lunarMonth.value = d.getMonth() + 1 + "月";
  lunarDay.value = d.getDate() + "日";
};

/** ===== 时间系统（核心修复） ===== */

let minuteTimer = null;
let dayTimer = null;

/** 同步时间 */
const syncTime = () => {
  today.value = new Date();
  updateLunar();
};

/** 启动“分钟级”更新（兜底用） */
const startMinuteTimer = () => {
  const delay = 60000 - (Date.now() % 60000);

  setTimeout(() => {
    syncTime();
    minuteTimer = setInterval(syncTime, 60000);
  }, delay);
};

/** 🚨 关键：跨天定时器 */
const startDayTimer = () => {
  const now = new Date();

  const tomorrow = new Date(
    now.getFullYear(),
    now.getMonth(),
    now.getDate() + 1,
    0, 0, 0, 0
  );

  const delay = tomorrow - now;

  dayTimer = setTimeout(() => {
    syncTime();

    // 递归，继续监听下一天
    startDayTimer();
  }, delay);
};

/** 生命周期 */
onMounted(() => {
  syncTime();

  startMinuteTimer(); // 兜底
  startDayTimer();    // 核心

  document.addEventListener("visibilitychange", syncTime);
  window.addEventListener("focus", syncTime);
  window.addEventListener("pageshow", syncTime);
});

onUnmounted(() => {
  clearInterval(minuteTimer);
  clearTimeout(dayTimer);

  document.removeEventListener("visibilitychange", syncTime);
  window.removeEventListener("focus", syncTime);
  window.removeEventListener("pageshow", syncTime);
});
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "Microsoft YaHei", sans-serif;
}

body {
  background-color: #f5f7fa;
  padding: 20px;
}

:root {
  --other-month-bg: #fa0000;
}

.dark {
  --other-month-bg: #0080ff;
}

.tk-page-card {
  margin-top: 10px;
}

.card-widget {
  max-height: calc(100vh - 100px);
  position: relative;
}

#card-widget-calendar .item-headline {
  padding-bottom: 0;
  margin-left: 8px;
  font-size: 1em;
  font-weight: 700;
  display: flex;
  align-items: center;
  gap: 5px;
  margin-bottom: 10px;
}

#card-widget-calendar .item-headline i {
  font-size: 18px;
}

#card-widget-calendar .item-content {
  display: flex;
}

#calendar-area-left,
#calendar-area-right {
  height: 100%;
  padding: 4px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

#calendar-area-left {
  width: 45%;
}

#calendar-week {
  height: 1.2rem;
  font-size: 14px;
  letter-spacing: 1px;
  font-weight: 700;
  display: flex;
  align-items: center;
}

#calendar-date {
  height: 3rem;
  line-height: 1.3;
  font-size: 36px;
  letter-spacing: 3px;
  color: var(--vp-c-brand-1);
  font-weight: 700;
  position: absolute;
  top: calc(50% - 2.1rem);
}

#calendar-solar {
  bottom: 2.1rem;
}

#calendar-lunar,
#calendar-solar {
  height: 1rem;
  font-size: 11px;
  position: absolute;
}

#calendar-lunar {
  bottom: 1rem;
}

#calendar-area-right {
  width: 55%;
}

#calendar-main {
  width: 100%;
}

.calendar-r0,
.calendar-r1,
.calendar-r2,
.calendar-r3,
.calendar-r4,
.calendar-r5,
.calendar-r6 {
  height: 1.2rem;
  display: flex;
}

.calendar-d0,
.calendar-d1,
.calendar-d2,
.calendar-d3,
.calendar-d4,
.calendar-d5,
.calendar-d6 {
  width: calc(100% / 7);
  display: flex;
  justify-content: center;
  align-items: center;
}

#calendar-main a {
  height: 1.2rem;
  width: 1.2rem;
  border-radius: 50%;
  font-size: 12px;
  display: flex;
  justify-content: center;
  align-items: center;
  text-decoration: none;
  color: var(--calendar-main-a-color);
}

#calendar-main a.now {
  background: var(--vp-c-brand-1);
  color: #fff;
}

#calendar-main a.other-month {
  color: var(--other-month-color);
}
</style>