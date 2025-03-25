<template>
	<view>
		<!-- 课程详情弹窗 -->
		<view v-if="showPopup" class="popup-mask" @click="closePopup">
			<view class="popup-content" @click.stop>
				<view class="close-btn" @click="closePopup">×</view>
				<view class="course-header">
					<text class="course-name">{{ selectedCourse?.name }}</text>
					<text :class="['course-status', nowWeek >= selectedCourse?.week_begin && nowWeek <= selectedCourse?.week_end ? 'status-current' : 'status-other']">
						{{ nowWeek >= selectedCourse?.week_begin && nowWeek <= selectedCourse?.week_end ? '本周课程' : '非本周' }}
					</text>
				</view>
				<view class="course-detail">
					<view class="detail-row">
						<text class="detail-label">授课教师</text>
						<text class="detail-value">{{ selectedCourse?.teacher }}</text>
					</view>
					<view class="detail-row">
						<text class="detail-label">上课地点</text>
						<text class="detail-value">{{ selectedCourse?.place }}</text>
					</view>
					<view class="detail-row">
						<text class="detail-label">周次安排</text>
						<text class="detail-value">第{{ selectedCourse?.week_begin }}-{{ selectedCourse?.week_end }}周</text>
					</view>
					<view class="detail-row">
						<text class="detail-label">上课时间</text>
						<text class="detail-value">{{ selectedCourse?.begin }}-{{ selectedCourse?.end }}节
							({{ time[selectedCourse?.begin - 1][0] }}-{{ time[selectedCourse?.end - 1][1] }})</text>
					</view>
				</view>
			</view>
		</view>
		<image v-if="image!=null" mode="widthFix" :src="image" class="backimg" :style="'height:'+height+'px'"></image>
		<view v-if="showWeeks" class="weeks" @click="showWeeks = false">
			<view class="week-selector-container" @click.stop>
				<view class="week-top">
					<text>选择周次</text>
					<text class="week-close-btn" @click="showWeeks = false">×</text>
				</view>
				<view class="week-content">
					<view class="week-semester">{{ semesterName }}</view>
					<view class="week-grid">
						<view 
							v-for="weekNum in totalWeeks" 
							:key="weekNum" 
							class="week-item"
							:class="{ 
								'week-item-current': weekNum === currentDisplayWeek,
								'week-item-selected': weekNum === nowWeek 
							}"
							@click="changeWeek(weekNum)"
						>
							{{ weekNum }}
						</view>
					</view>
					<view class="week-legend">
						<view class="legend-item">
							<view class="legend-color current-color"></view>
							<text>当前周</text>
						</view>
						<view class="legend-item">
							<view class="legend-color selected-color"></view>
							<text>已选择</text>
						</view>
					</view>
				</view>
			</view>
		</view>
		<view>
			<view class="top">
				<view :style="{ height: statusBarHeight +'px' }"></view>
				<view class="week" @click="changeWeek(0)">
					<view class="week-info">
						<text class="week-date">{{ formatDate }}</text>
						<view class="week-text">
							<text>第{{ nowWeek }}周 </text>
							<text class="week-current">当前为第{{ currentDisplayWeek }}周</text>
						</view>
					</view>
					<view class="week-actions">
						<text class="action-btn">+</text>
						<text class="action-btn">⬇</text>
					</view>
				</view>
				<view style="display: flex; flex-direction: row;">
					<view class="top-text month-cell" :style="'width:' + (width / 8 - 14) + 'px;'">
						<text class="month-num">{{ currentMonth }}</text>
						<text class="month-text">月</text>
					</view>
					<view class="top-text date-cell" :style="'width:' + (width / 8 + 2) + 'px;'"
						v-for="(item, index) in ['一', '二', '三', '四', '五', '六', '日']" :key="index">
						<view class="weekday">周{{ item }}</view>
						<view class="date">{{ currentWeekDates[index].date }}</view>
					</view>
				</view>
			</view>
			<view 
  style="width: 100%; display: flex;" 
  class="content-container"
  @touchstart="handleTouchStart"
  @touchend="handleTouchEnd"
>
				<view>
					<view class="left"
						:style="'width:' + (width / 8 - 14) + 'px; height: 160rpx; color:' + timeColor + ';'"
						v-for="(item, index) in time.length" :key="index">
						<view style="font-size: 30rpx">{{ index+1 }}</view>
						<view style="font-size: 20rpx">{{ time[index][0] }}</view>
						<view style="font-size: 20rpx">{{ time[index][1] }}</view>
					</view>
				</view>
				<view v-for="(item, index) in time.length" :key="index">
					<view v-if="index!=0" :style="'margin-top:' + (index == 0 ? 0 : index * 160) + 'rpx;'" class="hx">
					</view>
					<view v-if="index<7"
						:style="'margin-left:' + (index == 0 ? 0 : (index * (width - (width / 8 - 14))) / 7) + 'px;' +'height:'+(time.length*160)+'rpx;' "
						class="sx"></view>
				</view>
				<view v-for="(item, index) in courseList" :key="item.id">
					<view v-if="(nowWeek>=item.week_begin&&nowWeek<=item.week_end) || showNotWeek" @click="getCourse(item)" class="kcb-item text-center" :style="'margin-left:' +
			                (((item.week - 1) * (width - (width / 8 - 14))) / 7 + 2) + 'px; margin-top:' +
			                ((item.begin - 1) * 160 + 5) + 'rpx; width:' +
			                ((width - (width / 8 - 14)) / 7 - 4) + 'px; height: ' + ((item.end-item.begin +1)*151 + (item.end-item.begin)*10) + 'rpx; '
							 + 'background:' + ((nowWeek>=item.week_begin&&nowWeek<=item.week_end)?colorArrays[index % 10]:'#a0a0a0')">
						<view class="smalltext">
							<view style="margin-top: 2px;">{{ item.name }}</view>
							<view>{{ item.place }}</view>
							<view>{{ item.teacher }}</view>
							<view>{{ item.begin }}-{{ item.end }}节</view>
							<view>
								{{ item.week_begin!=item.week_end?(item.week_begin+'-'+item.week_end):item.week_begin }}周
							</view>
							<view v-if="!(nowWeek>=item.week_begin&&nowWeek<=item.week_end)">非本周</view>
						</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script setup>
import { ref, defineProps, defineEmits, onMounted, computed } from 'vue';

// 计算日期相关函数
const getWeekDates = (week) => {
  // 设置第一周的起始日期（2025年2月17日）
  const firstWeekStart = new Date(2025, 1, 17); // 注意：月份是从0开始的，所以1代表2月
  
  // 计算目标周的起始日期
  const targetWeekStart = new Date(firstWeekStart);
  targetWeekStart.setDate(firstWeekStart.getDate() + (week - 1) * 7);
  
  // 生成一周的日期
  const weekDates = [];
  for (let i = 0; i < 7; i++) {
    const date = new Date(targetWeekStart);
    date.setDate(targetWeekStart.getDate() + i);
    weekDates.push({
      month: date.getMonth() + 1,
      date: date.getDate()
    });
  }
  return weekDates;
};

// 定义 props
const props = defineProps({
  showNotWeek: {
    type: Boolean,
    default: true
  },
  week: {
    type: Number,
    default: 1
  },
  totalWeeks: {
    type: Number,
    default: 20
  },
  semesterName: {
    type: String,
    default: '2024-2025学年第二学期'
  },
  image: {
    type: String,
    default: null
  },
  time: {
    type: Array,
    default: () => [
      ['8:00', '8:45'],
      ['8:50', '9:35'],
      ['9:55', '10:40'],
      ['10:45', '11:30'],
      ['14:00', '14:45'],
      ['14:50', '15:35'],
      ['15:55', '16:40'],
      ['16:45', '17:30'],
      ['19:00', '19:45'],
      ['19:50', '20:35']
    ]
  },
  colorArrays: {
    type: Array,
    default: () => [
      '#70eb55', '#ffc450', '#ff9bd7', '#86affe', '#58eea4', 
      '#ffa17d', '#1cbbb4', '#6964ad', '#d42245', '#218285'
    ]
  },
  timeColor: {
    type: String,
    default: '#000000'
  },
  courseList: {
    type: Array,
    default: () => []
  }
});

// 定义 emits
const emit = defineEmits(['click']);

// 响应式数据
const touchStartX = ref(0);
const width = ref(0);
const height = ref(0);
const statusBarHeight = ref(0);
const showWeeks = ref(false);
const nowWeek = ref(props.week);
const showPopup = ref(false);
const selectedCourse = ref(null);
const totalWeeks = ref(20); // 默认总周数
const semesterName = ref('2024-2025学年第二学期'); // 学期名称

// 计算当前周的日期
const currentWeekDates = computed(() => {
  return getWeekDates(nowWeek.value);
});

// 当前月份
const currentMonth = computed(() => {
  return currentWeekDates.value[0].month;
});

// 格式化日期显示
const formatDate = computed(() => {
  const date = getWeekDates(nowWeek.value)[0];
  return `${2025}/${date.month}/${date.date}`;
});

// 当前显示周次（示例：当前为第5周）
const currentDisplayWeek = ref(5);

// 组件挂载时获取设备信息
onMounted(() => {
  const systemInfo = uni.getSystemInfoSync();
  width.value = systemInfo.windowWidth;
  height.value = systemInfo.windowHeight;
  statusBarHeight.value = systemInfo.statusBarHeight;
  
  // 初始化配置
  if (props.totalWeeks) totalWeeks.value = props.totalWeeks;
  if (props.semesterName) semesterName.value = props.semesterName;
});

// 课程点击事件
const getCourse = (item) => {
  emit('click', item);
  selectedCourse.value = item;
  showPopup.value = true;
};

const closePopup = () => {
  showPopup.value = false;
  selectedCourse.value = null;
};

const handleTouchStart = (e) => {
  touchStartX.value = e.touches[0].clientX;
};

const handleTouchEnd = (e) => {
  const touchEndX = e.changedTouches[0].clientX;
  const deltaX = touchEndX - touchStartX.value;
  
  if (Math.abs(deltaX) > 50) { // 滑动阈值50px
    if (deltaX > 0) {
      changeWeek(Math.max(1, nowWeek.value - 1));
    } else {
      changeWeek(nowWeek.value + 1);
    }
  }
};

// 切换周次
const changeWeek = (week) => {
  if (week) {
    showWeeks.value = false;
    nowWeek.value = week;
  } else {
    showWeeks.value = true;
  }
};
</script>

<style>
	.popup-mask {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background: rgba(0, 0, 0, 0.6);
		backdrop-filter: blur(8px);
		display: flex;
		justify-content: center;
		align-items: center;
		z-index: 999;
		animation: fadeIn 0.2s ease-out;
	}

	.popup-content {
		position: relative;
		width: 650rpx;
		padding: 50rpx 40rpx;
		background: #ffffff;
		border-radius: 24rpx;
		box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.1);
		transform: scale(0.9);
		opacity: 0;
		animation: scaleIn 0.3s ease-out forwards;
	}

	.close-btn {
		position: absolute;
		top: 16rpx;
		right: 16rpx;
		width: 60rpx;
		height: 60rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 44rpx;
		color: #666;
		border-radius: 50%;
		transition: all 0.2s;
	}

	.close-btn:active {
		background-color: #f5f5f5;
		transform: scale(0.95);
	}

	.course-detail {
		margin-top: 20rpx;
	}

	.course-title {
		font-size: 36rpx;
		font-weight: 600;
		color: #333;
		margin-bottom: 30rpx;
		text-align: center;
	}

	.detail-item {
		font-size: 28rpx;
		line-height: 56rpx;
		color: #333;
		padding: 10rpx 20rpx;
		background: #f8f8f8;
		border-radius: 12rpx;
		margin-bottom: 16rpx;
		display: flex;
		align-items: center;
	}

	.detail-item:last-child {
		margin-bottom: 0;
	}

	.detail-label {
		color: #666;
		margin-right: 16rpx;
		font-size: 26rpx;
	}

	@keyframes fadeIn {
		from {
			opacity: 0;
		}
		to {
			opacity: 1;
		}
	}

	@keyframes scaleIn {
		from {
			transform: scale(0.9);
			opacity: 0;
		}
		to {
			transform: scale(1);
			opacity: 1;
		}
	}

	.course-header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin-bottom: 30rpx;
	}

	.course-name {
		font-size: 36rpx;
		font-weight: bold;
		color: #333;
	}

	.course-status {
		padding: 4rpx 16rpx;
		border-radius: 8rpx;
		font-size: 24rpx;
	}

	.status-current {
		background: #e8f5e9;
		color: #4caf50;
	}

	.status-other {
		background: #f5f5f5;
		color: #999;
	}

	.course-detail {
		margin-top: 20rpx;
	}

	.detail-row {
		display: flex;
		margin-bottom: 24rpx;
		align-items: center;
	}

	.detail-label {
		width: 140rpx;
		color: #666;
		font-size: 28rpx;
	}

	.detail-value {
		flex: 1;
		color: #333;
		font-size: 28rpx;
	}

	.kcb-item {
		position: absolute;
		justify-content: center;
		display: flex;
		border-radius: 5px;
		background-size: 400% 400%;
		animation: Change 2s ease infinite;
	}

	.hx {
		position: absolute;
		display: flex;
		border-bottom: 1px dashed #a3a3a3;
		left: 0;
		right: 0;
	}

	.sx {
		position: absolute;
		display: flex;
		/* border-left: 1px dashed #a3a3a3; */
	}

	.smalltext {
		display: block;
		color: #fff;
		font-size: 24rpx;
		font-weight: bolder;
		overflow: hidden;
		text-overflow: ellipsis;
	}

	.left {
		justify-content: center;
		display: flex;
		flex-direction: column;
		align-items: center;
		color: black;
	}

	.text-center {
		display: flex;
		align-items: center;
		justify-content: center;
		text-align: center;
	}

	.top {
		display: flex;
		flex-direction: column;
		color: black;
		align-items: center;
		position: relative;
		background: #fff;
		z-index: 1;
		border-bottom: 1rpx solid #a3a3a3;
		font-size: 30rpx;
		width: 100%;
	}
	
	.content-container {
		margin-top: 10px;
	}

	.top-text {
		justify-content: center;
		display: flex;
		align-items: center;
		flex-direction: column;
		height: 50px;
		padding: 4rpx 0;
	}

	.month-cell {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.month-num {
		font-size: 32rpx;
		font-weight: 500;
		line-height: 1;
		margin-bottom: 4rpx;
	}

	.month-text {
		font-size: 24rpx;
		color: #666;
	}

	.date-cell {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: space-between;
	}

	.weekday {
		font-size: 28rpx;
		margin-bottom: 4rpx;
	}

	.date {
		font-size: 24rpx;
		color: #666;
	}

	.backimg {
		width: 100%;
		z-index: -1;
		position: fixed;
	}

	.week {
		display: flex;
		width: 100%;
		flex-direction: row;
		align-items: center;
		justify-content: space-between;
		padding: 20rpx 30rpx;
		height: auto;
		box-sizing: border-box;
	}

	.week-info {
		display: flex;
		flex-direction: column;
		align-items: flex-start;
		gap: 4rpx;
	}

	.week-date {
		font-size: 36rpx;
		font-weight: bold;
		color: #333;
	}

	.week-text {
		font-size: 28rpx;
		color: #333;
		display: flex;
		align-items: center;
		gap: 8rpx;
	}

	.week-current {
		color: #666;
	}

	.week-actions {
		display: flex;
		align-items: center;
		gap: 20rpx;
	}

	.action-btn {
		font-size: 36rpx;
		color: #333;
		padding: 4rpx 12rpx;
		cursor: pointer;
	}

	.weeks {
		width: 100vw;
		height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;
		background: rgba(0, 0, 0, 0.5);
		position: fixed;
		z-index: 999;
		animation: fadeIn 0.2s ease-out;
	}

	.week-selector-container {
		width: 650rpx;
		background: #ffffff;
		border-radius: 16rpx;
		box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.15);
		overflow: hidden;
		transform: scale(0.95);
		animation: scaleIn 0.3s ease-out forwards;
	}

	.week-top {
		width: 100%;
		display: flex;
		align-items: center;
		justify-content: space-between;
		background: #39b54a;
		color: #ffffff;
		padding: 25rpx 30rpx;
		box-sizing: border-box;
		font-size: 32rpx;
		font-weight: 500;
	}

	.week-close-btn {
		font-size: 40rpx;
		width: 60rpx;
		height: 60rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		border-radius: 50%;
	}

	.week-content {
		padding: 30rpx;
	}

	.week-semester {
		text-align: center;
		font-size: 28rpx;
		color: #666;
		margin-bottom: 20rpx;
	}

	.week-grid {
		display: grid;
		grid-template-columns: repeat(5, 1fr);
		gap: 15rpx;
		margin-bottom: 30rpx;
	}

	.week-item {
		display: flex;
		align-items: center;
		justify-content: center;
		height: 80rpx;
		background: #f5f5f5;
		border-radius: 8rpx;
		font-size: 28rpx;
		color: #333;
		position: relative;
		transition: all 0.2s ease;
	}

	.week-item::before {
		content: "第";
		font-size: 20rpx;
		position: absolute;
		top: 12rpx;
		left: 12rpx;
		color: #999;
	}

	.week-item::after {
		content: "周";
		font-size: 20rpx;
		position: absolute;
		bottom: 12rpx;
		right: 12rpx;
		color: #999;
	}

	.week-item:active {
		transform: scale(0.95);
	}

	.week-item-current {
		border: 2rpx solid #3a86ff;
		color: #3a86ff;
	}

	.week-item-selected {
		background: #39b54a;
		color: white;
	}

	.week-item-selected::before,
	.week-item-selected::after {
		color: rgba(255, 255, 255, 0.8);
	}

	.week-legend {
		display: flex;
		justify-content: center;
		gap: 30rpx;
	}

	.legend-item {
		display: flex;
		align-items: center;
		gap: 8rpx;
		font-size: 24rpx;
		color: #666;
	}

	.legend-color {
		width: 24rpx;
		height: 24rpx;
		border-radius: 4rpx;
	}

	.current-color {
		border: 2rpx solid #3a86ff;
		background: transparent;
	}

	.selected-color {
		background: #39b54a;
	}
</style>