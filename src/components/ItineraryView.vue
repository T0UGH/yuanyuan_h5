<script setup>
// 修复：添加生命周期钩子导入
import { ref, computed, onMounted, onBeforeUnmount } from 'vue';

const currentImageIndex = ref(0);

const prevImage = () => {
  if (!currentSpot.value?.images) return;
  currentImageIndex.value = (currentImageIndex.value - 1 + currentSpot.value.images.length) % currentSpot.value.images.length;
};

const nextImage = () => {
  if (!currentSpot.value?.images) return;
  currentImageIndex.value = (currentImageIndex.value + 1) % currentSpot.value.images.length;
};
// 修复：添加itineraryData导入
import { 
  getCurrentTimeSpot, 
  getItineraryByTime,
  itineraryData  // 添加这一行
} from '../data/itinerary.js';

const props = defineProps({
  isGirl: Boolean
});

const emit = defineEmits(['update:isGirl']);

console.log('ItineraryView组件初始化，isGirl:', props.isGirl);

const selectedDate = ref(new Date());
const customDate = ref('');
const customTime = ref('');
const showCustomInput = ref(false);
// 新增响应式变量
const selectedDateTime = ref('');

const toggleUserType = () => {
  emit('update:isGirl', !props.isGirl);
};

// 删除旧的 customDate 和 customTime 声明
// const customDate = ref('');
// const customTime = ref('');

// 修改后的时间选项计算属性
// 重命名为 dateTimeOptions 避免重复声明
const dateTimeOptions = computed(() => {
  const options = [];
  Object.entries(itineraryData).forEach(([date, entries]) => {
    entries.forEach(entry => {
      // 过滤无效时间段（如zzz）
      if (!entry.time.includes('-')) return;
      
      const value = `${date}_${entry.time}`;
      const label = `${date.replace(/-/g, '/')} ${entry.time}`;
      
      if (!options.some(opt => opt.value === value)) {
        options.push({ value, label });
      }
    });
  });
  
  // 按日期时间排序
  return options.sort((a, b) => 
    new Date(a.value.split('_')[0]) - new Date(b.value.split('_')[0]) ||
    a.value.localeCompare(b.value)
  );
});

// 修改后的日期变更处理函数
const handleDateTimeChange = () => {
  if (!selectedDateTime.value) return;
  
  // 解析选择的日期和时间段
  const [dateStr, timeStr] = selectedDateTime.value.split('_');
  const [startTime] = timeStr?.split('-') || [];
  
  if (!dateStr || !startTime) {
    console.error('无效的日期时间格式');
    return;
  }

  // 创建日期对象（时区已处理）
  const [hours, minutes] = startTime.split(':');
  const date = new Date(`${dateStr}T${hours}:${minutes}:00+08:00`);
  
  if (date.toString() === 'Invalid Date') {
    console.error('无效的日期时间');
    return;
  }
  
  selectedDate.value = date;
};

const currentSpot = computed(() => {
  console.log('计算currentSpot，当前时间:', selectedDate.value.toLocaleString(), '用户类型:', props.isGirl);
  const spot = getCurrentTimeSpot(selectedDate.value, props.isGirl);
  console.log('获取到的currentSpot:', spot);
  return spot;
});

const allSpots = computed(() => {
  console.log('计算allSpots，选择的日期:', selectedDate.value.toLocaleString(), '用户类型:', props.isGirl);
  const spots = getItineraryByTime(selectedDate.value, props.isGirl);
  console.log('获取到的allSpots:', spots);
  return spots;
});

const toggleCustomInput = () => {
  showCustomInput.value = !showCustomInput.value;
};

const handleCustomDateChange = () => {
  // 改为从下拉框直接获取时间
  if (!customDate.value) return;
  
  // 解析选择的时间段（示例："07:00-08:30"）
  const [startTime] = customTime.value.split('-');
  const [hours, minutes] = startTime.split(':');
  
  // 创建日期对象（时区已处理）
  const dateTimeStr = `${customDate.value}T${hours}:${minutes}:00+08:00`;
  console.log('日期时间变更，新选择的日期:', customDate.value, '时间:', customTime.value);
  
  const date = new Date(dateTimeStr);
  if (date.toString() === 'Invalid Date') {
    console.error('无效的日期格式');
    return;
  }
  
  selectedDate.value = date;
};

// 获取所有可用时间段（从行程数据中提取）

// 修改后的时间选项计算属性（保持命名一致）
const timeOptions = computed(() => {
  const options = [];
  Object.entries(itineraryData).forEach(([date, entries]) => {
    entries.forEach(entry => {
      // 确保 entry.time 是有效时间段（如 "07:00-08:30"）
      if (!entry.time || !entry.time.includes('-')) return;
      
      const value = `${date}_${entry.time}`;
      const label = `${date.replace(/-/g, '/')} ${entry.time}`;
      
      if (!options.some(opt => opt.value === value)) {
        options.push({ value, label });
      }
    });
  });
  
  // 按日期排序
  return options.sort((a, b) => 
    new Date(a.value.split('_')[0]) - new Date(b.value.split('_')[0]) ||
    a.value.localeCompare(b.value)
  );
});

// 在这里添加调试输出
console.log('生成的时间选项:', timeOptions.value);

const resetToCurrentTime = () => {
  console.log('重置为当前时间');
  selectedDate.value = new Date();
  customDate.value = '';
  showCustomInput.value = false;
};

const autoPlay = ref(null); // 添加自动播放的ref

// 添加自动播放逻辑
onMounted(() => {
  autoPlay.value = setInterval(nextImage, 3000);
});

onBeforeUnmount(() => {
  clearInterval(autoPlay.value);
});
</script>

<template>
  <div class="itinerary-container">
    <div class="cherry-blossom-bg"></div>
    <div class="content">
        <div class="user-control">
        <div class="user-info">
          <p class="user-type">我是谁: {{ isGirl ? '圆圆' : '孤寡' }}</p>
          <button class="switch-user" @click="toggleUserType">切换身份</button>
        </div>
      </div>

      <div class="time-control">
        <button @click="resetToCurrentTime" :class="{ active: !showCustomInput }">
          使用当前时间
        </button>
        <button @click="toggleCustomInput" :class="{ active: showCustomInput }">
          选择其他日期
        </button>
      </div>

      <div v-if="showCustomInput" class="date-input">
        <select 
          v-model="selectedDateTime"
          @change="handleDateTimeChange"
          class="time-select"
        >
          <option value="">选择日期和时间段</option>
          <option 
            v-for="(option, index) in timeOptions" 
            :key="index" 
            :value="option.value"
          >
            {{ option.label }}
          </option>
        </select>
      </div>

      <div class="current-spot" v-if="currentSpot">
        <h3>当前行程</h3>
        <div class="spot-card">
          <div class="image-carousel" v-if="currentSpot.images && currentSpot.images.length > 0">
            <img :src="currentSpot.images[currentImageIndex]" :alt="currentSpot.spot" class="spot-image">
            <div class="carousel-indicator" v-if="currentSpot.images.length > 1">
              <div 
                v-for="(img, index) in currentSpot.images" 
                :key="index" 
                class="indicator-dot"
                :class="{ active: index === currentImageIndex }"
              ></div>
            </div>
          </div>
          <div class="spot-header">
            <h4>{{ currentSpot.spot }}</h4>
            <span class="time">{{ currentSpot.time }}</span>
          </div>
          <p class="description">{{ currentSpot.description }}</p>
          <div class="tips">
            <h5>游玩建议</h5>
            <ul>
              <li v-for="(tip, index) in currentSpot.tips" :key="index">
                {{ tip }}
              </li>
            </ul>
          </div>
        </div>
      </div>

      <div class="all-spots">
        <h3>当日全部行程</h3>
        <div class="spot-card" v-for="spot in allSpots" :key="spot.time">
          <img v-if="currentSpot.image" :src="currentSpot.image" :alt="currentSpot.spot" class="spot-image">
          <div class="spot-header">
            <h4>{{ spot.spot }}</h4>
            <span class="time">{{ spot.time }}</span>
          </div>
          <p class="description">{{ spot.description }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.itinerary-container {
  position: relative;
  min-height: 100vh;
  padding: 2rem 1rem;
  background-color: #fff5f7;
}

.cherry-blossom-bg {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, #fff5f7 0%, #ffd6e7 100%);
  opacity: 0.8;
  z-index: 0;
}

.content {
  position: relative;
  z-index: 1;
  max-width: 800px;
  margin: 0 auto;
}

.time-control {
  display: flex;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

button {
  flex: 1;
  padding: 1rem;
  border: 2px solid #ff69b4;
  background-color: white;
  color: #ff69b4;
  border-radius: 1rem;
  cursor: pointer;
  transition: all 0.3s;
  font-size: 1rem;
  font-weight: 600;
  box-shadow: 0 2px 8px rgba(255, 105, 180, 0.2);
}

button.active, button:hover {
  background-color: #ff69b4;
  color: white;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(255, 105, 180, 0.3);
}

.date-input {
  margin-bottom: 1.5rem;
  display: flex;
  gap: 1rem;
}

input[type="date"],
input[type="time"] {
  width: 100%;
  padding: 1rem;
  border: 2px solid #ff69b4;
  border-radius: 1rem;
  background-color: white;
  color: #ff69b4;
  font-size: 1rem;
  font-weight: 500;
  appearance: none;
  background-image: url("data:image/svg+xml;charset=UTF-8,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='%23ff69b4'%3e%3cpath d='M7 10l5 5 5-5z'/%3e%3c/svg%3e");
  background-repeat: no-repeat;
  background-position: right 1rem center;
  background-size: 1.2em;
  box-shadow: 0 2px 8px rgba(255, 105, 180, 0.2);
  transition: all 0.3s;
}

/* 新增下拉框样式 */
.time-select {
  width: 100%;
  padding: 1rem;
  border: 2px solid #ff69b4;
  border-radius: 1rem;
  background-color: white;
  color: #ff69b4;
  font-size: 1rem;
  font-weight: 500;
  appearance: none;
  background-image: url("data:image/svg+xml;charset=UTF-8,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='%23ff69b4'%3e%3cpath d='M7 10l5 5 5-5z'/%3e%3c/svg%3e");
  background-repeat: no-repeat;
  background-position: right 1rem center;
  background-size: 1.2em;
  box-shadow: 0 2px 8px rgba(255, 105, 180, 0.2);
  transition: all 0.3s;
}

.time-select:focus {
  outline: none;
  border-color: #ff1493;
  box-shadow: 0 0 0 3px rgba(255, 20, 147, 0.1);
}

/* 移动旧的下拉样式到正确位置 */
.date-input {
  margin-bottom: 1.5rem;
}

/* 删除重复的.time-select:hover样式 */
.time-select:hover {
  border-color: #ff1493;
  box-shadow: 0 4px 12px rgba(255, 105, 180, 0.3);
}

.time-select option {
  padding: 0.8rem;
  background: white;
  color: #ff69b4;
}

.time-select option:hover {
  background: #fff5f7 !important;
}

.current-spot, .all-spots {
  margin-bottom: 2rem;
}

h3 {
  color: #ff1493;
  margin-bottom: 1.5rem;
  font-size: 1.5rem;
  font-weight: 600;
  text-align: center;
  text-shadow: 0 2px 4px rgba(255, 105, 180, 0.2);
}

.user-control {
  margin-bottom: 1.5rem;
  text-align: center;
}

.user-control {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.switch-user {
  width: auto;
  padding: 0.5rem 1.5rem;
  margin-top: 0.5rem;
}

.user-type {
  color: #ff1493;
  font-size: 1.1rem;
  font-weight: 600;
  margin: 0;
}

.spot-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
  border-radius: 0.5rem;
  margin-bottom: 1rem;
}

.spot-card {
  background: white;
  border-radius: 0.5rem;
  padding: 1.5rem;
  margin-bottom: 1rem;
  box-shadow: 0 2px 8px rgba(255, 105, 180, 0.1);
  border-left: 8px solid #ff1493;
  transition: transform 0.3s, box-shadow 0.3s;
}

.spot-card:hover {
  transform: translateX(5px);
  box-shadow: 0 4px 12px rgba(255, 105, 180, 0.2);
}

.spot-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.spot-card h4 {
  color: #ff1493;
  margin: 0;
  font-size: 1.3rem;
  font-weight: 600;
}

.time {
  color: #ff69b4;
  font-size: 1rem;
  font-weight: 500;
  background: #fff5f7;
  padding: 0.5rem 1rem;
  border-radius: 2rem;
}

.description {
  color: #666;
  margin-bottom: 1.5rem;
  line-height: 1.6;
  font-size: 1rem;
}

.tips {
  background: #fff5f7;
  padding: 1.5rem;
  border-radius: 1rem;
  border: 1px dashed #ff69b4;
}

.tips h5 {
  color: #ff1493;
  margin: 0 0 1rem 0;
  font-size: 1.1rem;
  font-weight: 600;
}

.tips ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.tips li {
  color: #666;
  margin-bottom: 0.8rem;
  padding-left: 1.5rem;
  position: relative;
  line-height: 1.5;
}

.tips li::before {
  content: '🌸';
  position: absolute;
  left: 0;
  font-size: 0.9rem;
}

@media (max-width: 600px) {
  .itinerary-container {
    padding: 1rem;
  }

  .user-control {
  margin-bottom: 1.5rem;
  text-align: center;
}

.user-control {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.switch-user {
  width: auto;
  padding: 0.5rem 1.5rem;
  margin-top: 0.5rem;
}

.user-type {
  color: #ff1493;
  font-size: 1.1rem;
  font-weight: 600;
  margin: 0;
}

.image-carousel {
  position: relative;
  width: 100%;
  margin-bottom: 1rem;
}

.carousel-controls {
  position: absolute;
  top: 50%;
  left: 0;
  right: 0;
  transform: translateY(-50%);
  display: flex;
  justify-content: space-between;
  padding: 0 1rem;
}

.carousel-btn {
  background: rgba(255, 105, 180, 0.8);
  color: white;
  border: none;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.5rem;
  transition: all 0.3s;
}

.carousel-btn:hover {
  background: rgba(255, 20, 147, 0.9);
  transform: scale(1.1);
}

.spot-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
  border-radius: 0.5rem;
  margin-bottom: 1rem;
}

.spot-card {
    padding: 1.2rem;
  }

  .spot-header {
    flex-direction: column;
    align-items: flex-start;
  }

  .time {
    margin-top: 0.5rem;
  }
}
.user-info {
  display: flex;
  align-items: center;
  gap: 1.5rem;
}

.user-type {
  margin: 0; /* 移除原有margin */
}

.switch-user {
  margin-top: 0; /* 移除原有上边距 */
}
</style>

export function getCurrentTimeSpot(date, isGirl = false) {
  const spots = getItineraryByTime(date, isGirl);
  if (!spots) return defaultSpot;
  
  // 修复：保持传入日期的时间部分
  const now = new Date(date); 
  const currentTime = now.getHours() * 60 + now.getMinutes();

  // ... existing find logic ...
}