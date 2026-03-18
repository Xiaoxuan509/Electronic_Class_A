<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const now = ref(new Date())
const weekDays = ['日', '一', '二', '三', '四', '五', '六']
const days = ['一', '二', '三', '四', '五']

let timer
onMounted(() => {
  timer = setInterval(() => {
    now.value = new Date()
  }, 1000) // 每 1000 毫秒（1秒）更新一次
})

onUnmounted(() => {
  clearInterval(timer) // 離開頁面時清除計時器，避免耗電
})

const sections = [1, 2, 3, 4, 9, 5, 6, 7, 8]
const restSection = 9 // 第5節休息

const today = computed(() => now.value.getDay())

const selectedCourse = ref(null)

const colors = {
  common: '#d9f2ff', Pro: "#ffe4c4", other: "#e6f5d0", practice: "#ffd6e7"
}

// 課程
const courses = [
  { day: 1, start: 1, span: 2, name: '國文', color: colors.common },
  { day: 1, start: 3, span: 1, name: '體育', color: colors.other },
  { day: 1, start: 4, span: 1, name: '數學', color: colors.common },
  { day: 1, start: 5, span: 1, name: '數學', color: colors.common },
  { day: 1, start: 6, span: 2, name: '彈性選修', color: colors.other },
  { day: 1, start: 8, span: 1, name: '電子學', color: colors.Pro },

  { day: 2, start: 1, span: 4, name: '程式專題實習', color: colors.practice },
  { day: 2, start: 5, span: 1, name: '電子學', color: colors.Pro },
  { day: 2, start: 6, span: 1, name: '微處理機', color: colors.Pro },
  { day: 2, start: 7, span: 1, name: '公民與社會', color: colors.other },

  { day: 3, start: 1, span: 3, name: '電子學實習', color: colors.practice },
  { day: 3, start: 4, span: 1, name: '單晶片微處理機實習', color: colors.practice },
  { day: 3, start: 5, span: 2, name: '單晶片微處理機實習', color: colors.practice },
  { day: 3, start: 7, span: 1, name: '國文', color: colors.common },

  { day: 4, start: 1, span: 2, name: '數學', color: colors.common },
  { day: 4, start: 3, span: 2, name: '英文', color: colors.common },
  { day: 4, start: 5, span: 1, name: '公民與社會', color: colors.other },
  { day: 4, start: 6, span: 1, name: '體育', color: colors.other },
  { day: 4, start: 7, span: 2, name: '微處理機', color: '#cde7ff' },

  { day: 5, start: 1, span: 1, name: '團體活動時間1', color: colors.other },
  { day: 5, start: 2, span: 1, name: '微處理機', color: colors.Pro },
  { day: 5, start: 3, span: 2, name: '團體活動時間23', color: colors.other },
  { day: 5, start: 5, span: 1, name: '英文', color: colors.common },
  { day: 5, start: 6, span: 2, name: '電子學', color: colors.Pro },
]

// 公告事項
const announcements = ref([
  { text: '無', time: '', color: '#ff3b30' },
  { text: '無', time: '', color: '#ffcc00' },
  { text: '無', time: '', color: '#34c759' }
])

// 課程函數
const getCourse = (day, section) => courses.find(
  c => c.day === day && section >= c.start && section < c.start + c.span
)
const isCourseStart = (day, section) => courses.some(c => c.day === day && c.start === section)
const getCourseStyle = (day, section) => {
  const course = getCourse(day, section)
  return { background: course.color, height: `${70 * course.span + (course.span - 1) * 6}px` }
}
const openCourse = (day, section) => selectedCourse.value = getCourse(day, section)

// 日期顯示
const currentDateLabel = computed(() => {
  const month = now.value.getMonth() + 1
  const date = now.value.getDate()
  const day = weekDays[now.value.getDay()]

  // 取得時分秒，並用 padStart 補零（例如 9秒 變成 09秒）
  const hours = String(now.value.getHours()).padStart(2, '0')
  const minutes = String(now.value.getMinutes()).padStart(2, '0')
  const seconds = String(now.value.getSeconds()).padStart(2, '0')

  return `${month}月${date}日 星期${day} ${hours}:${minutes}:${seconds}`
})
</script>

<template>
  <div class="ios-dashboard">
    <!-- Header -->
    <header class="ios-header">
      <div class="header-left">
        <span class="date-label">{{ currentDateLabel }}</span>
        <h1>子二甲</h1>
      </div>
      <!--<div class="user-avatar">學</div>-->
    </header>

    <!-- 公告事項 -->
    <section class="announcement-section">
      <h2 class="section-title">重要事項</h2>
      <div class="ios-card-stack">
        <div v-for="(note, i) in announcements" :key="i" class="note-card">
          <div class="note-icon" :style="{ background: note.color }"></div>
          <div class="note-content">
            <div class="note-text">{{ note.text }}</div>
            <div class="note-time">{{ note.time }}</div>
          </div>
        </div>
      </div>
    </section>

    <!-- 課表 -->
    <section class="schedule-section">
      <h2 class="section-title">課表</h2>
      <div class="schedule-grid">
        <div class="corner-space"></div>
        <div v-for="(day, i) in days" :key="day" class="day-header" :class="{ isToday: i + 1 === today }">{{ day }}
        </div>

        <template v-for="section in sections" :key="'sec-'+section">
          <div class="section-num" :class="{ rest: section === restSection }">
            {{ section === restSection ? '休息' : section }}
          </div>
          <div v-for="dayIndex in 5" :key="dayIndex + '-' + section" class="grid-cell">
            <div v-if="isCourseStart(dayIndex, section)" class="course-block" :style="getCourseStyle(dayIndex, section)"
              @click="openCourse(dayIndex, section)">
              {{ getCourse(dayIndex, section).name }}
            </div>
          </div>
        </template>
      </div>
    </section>

    <!-- Modal -->
    <div v-if="selectedCourse" class="modal" @click="selectedCourse = null">
      <div class="modal-content" @click.stop>
        <h3>{{ selectedCourse.name }}</h3>
        <p>星期：{{ days[selectedCourse.day - 1] }}</p>
        <p>節次：{{ selectedCourse.start }} ~ {{ selectedCourse.start + selectedCourse.span - 1 }}</p>
        <button @click="selectedCourse = null">關閉</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* 基礎重置 */
:global(html), :global(body) {
  background-color: #000 !important;
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
}

* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}

.ios-dashboard {
  background: #000;
  color: #fff;
  min-height: 100vh;
  padding: 20px 16px;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
}

/* Header 與 日期 */
.ios-header {
  margin-bottom: 24px;
}

.date-label {
  color: #8e8e93;
  font-size: 13px;
  font-weight: 600;
}

.ios-header h1 {
  font-size: 34px;
  margin-top: 4px;
  font-weight: 800;
}

/* 重要事項卡片 */
.section-title {
  font-size: 22px;
  margin-bottom: 12px;
  font-weight: 700;
}

.ios-card-stack {
  background: #1c1c1e;
  border-radius: 14px;
  padding: 0 16px;
  margin-bottom: 24px;
}

.note-card {
  display: flex;
  align-items: center;
  padding: 12px 0;
  border-bottom: 0.5px solid #38383a;
}

.note-card:last-child {
  border-bottom: none;
}

.note-icon {
  width: 4px;
  height: 24px;
  border-radius: 2px;
  margin-right: 12px;
}

.note-text {
  font-size: 16px;
}

.note-time {
  font-size: 12px;
  color: #8e8e93;
}

/* 課表 Grid 核心 - 解決黏在一起的問題 */
.schedule-grid {
  display: grid;
  grid-template-columns: 35px repeat(5, 1fr);
  gap: 6px;
  /* 這是格子之間的縫隙 */
  background: #000;
}

.day-header {
  text-align: center;
  font-size: 14px;
  padding-bottom: 8px;
  color: #8e8e93;
}

.day-header.isToday {
  color: #fff;
  font-weight: bold;
}

.section-num {
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 13px;
  color: #8e8e93;
  height: 70px;
}

.section-num.rest {
  height: 40px;
  /* 休息列變窄 */
  font-size: 11px;
}

/* 格子底色 */
.grid-cell {
  background: #1c1c1e;
  border-radius: 8px;
  height: 70px;
  position: relative;
}

/* 讓休息列的格子同步變窄 */
.grid-cell:nth-child(n+26):nth-child(-n+31) {
  height: 40px;
}

/* 課程方塊 - 修正寬高與間距 */
.course-block {
  position: absolute;
  /* 關鍵修正：寬度改為 calc 減去微小間距，並留出 1px 的 margin */
  width: calc(100% - 2px);
  left: 1px;
  top: 1px;

  border-radius: 7px;
  padding: 4px;
  font-size: 14px;
  color: #000;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  z-index: 5;
  cursor: pointer;

  /* 讓跨節方塊不要死板板貼齊 */
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
}

/* 針對行動裝置微調字體 */
@media screen and (max-width: 768px) {
  .course-block {
    font-size: 12px;
  }
}

/* Modal 彈窗 */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
}

.modal-content {
  background: #1c1c1e;
  color: #fff;
  padding: 24px;
  border-radius: 18px;
  width: 280px;
  text-align: center;
}

.modal-content h3 {
  margin-bottom: 12px;
  font-size: 20px;
}

.modal-content p {
  margin: 8px 0;
  color: #8e8e93;
}

.modal-content button {
  margin-top: 16px;
  padding: 8px 24px;
  background: #34c759;
  border: none;
  border-radius: 8px;
  color: #fff;
  font-weight: 600;
}
</style>
