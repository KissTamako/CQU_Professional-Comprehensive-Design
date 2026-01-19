<template>
	<view class="container">
		<!-- 顶部状态栏 -->
		<view class="status-bar">
			<view class="user-info">
				<image class="avatar" src="/static/logo.png"></image>
				<view class="user-details">
					<text class="username">快递员：李师傅</text>
					<text class="user-id">工号：COU2024001</text>
				</view>
			</view>
			<view class="stats">
				<view class="stat-item">
					<text class="stat-value">12</text>
					<text class="stat-label">今日任务</text>
				</view>
				<view class="stat-item">
					<text class="stat-value">8</text>
					<text class="stat-label">已完成</text>
				</view>
				<view class="stat-item">
					<text class="stat-value">4</text>
					<text class="stat-label">待处理</text>
				</view>
			</view>
		</view>

		<!-- 功能模块 -->
		<view class="function-grid">
			<view class="grid-title">快捷功能</view>
			<view class="grid-container">
				<view class="grid-item" @click="scanQRCode">
					<view class="grid-icon">
						<uni-icons type="scan" size="30" color="#007AFF"></uni-icons>
					</view>
					<text class="grid-text">扫码取件</text>
				</view>
				<view class="grid-item" @click="navigateToTaskList">
					<view class="grid-icon">
						<uni-icons type="list" size="30" color="#34C759"></uni-icons>
					</view>
					<text class="grid-text">任务列表</text>
				</view>
				<view class="grid-item" @click="navigateToDelivery">
					<view class="grid-icon">
						<uni-icons type="paperplane" size="30" color="#FF9500"></uni-icons>
					</view>
					<text class="grid-text">派送管理</text>
				</view>
				<view class="grid-item" @click="navigateToHistory">
					<view class="grid-icon">
						<uni-icons type="calendar" size="30" color="#5856D6"></uni-icons>
					</view>
					<text class="grid-text">历史记录</text>
				</view>
			</view>
		</view>

		<!-- 今日任务列表 -->
		<view class="task-section">
			<view class="section-header">
				<text class="section-title">今日任务</text>
				<text class="section-more" @click="navigateToTaskList">查看更多</text>
			</view>
			<view class="task-list">
				<view class="task-item" v-for="(task, index) in taskList" :key="index">
					<view class="task-info">
						<text class="task-location">{{ task.location }}</text>
						<text class="task-time">{{ task.time }}</text>
						<text class="task-status" :class="task.statusClass">{{ task.status }}</text>
					</view>
					<view class="task-actions">
						<button class="action-btn" size="mini" @click="viewTaskDetail(task)">详情</button>
						<button class="action-btn primary" size="mini" @click="startTask(task)" v-if="task.status === '待处理'">开始</button>
						<button class="action-btn success" size="mini" @click="completeTask(task)" v-if="task.status === '进行中'">完成</button>
					</view>
				</view>
			</view>
		</view>

		<!-- 底部导航 -->
		<view class="bottom-nav">
			<view class="nav-item" @click="switchTab('home')">
				<uni-icons type="home" size="24" :color="currentTab === 'home' ? '#007AFF' : '#8E8E93'"></uni-icons>
				<text class="nav-text" :class="{ active: currentTab === 'home' }">首页</text>
			</view>
			<view class="nav-item" @click="switchTab('tasks')">
				<uni-icons type="bell" size="24" :color="currentTab === 'tasks' ? '#007AFF' : '#8E8E93'"></uni-icons>
				<text class="nav-text" :class="{ active: currentTab === 'tasks' }">通知</text>
			</view>
			<view class="nav-item active" @click="switchTab('courier')">
				<uni-icons type="person" size="24" :color="currentTab === 'courier' ? '#007AFF' : '#8E8E93'"></uni-icons>
				<text class="nav-text" :class="{ active: currentTab === 'courier' }">工作台</text>
			</view>
			<view class="nav-item" @click="switchTab('profile')">
				<uni-icons type="gear" size="24" :color="currentTab === 'profile' ? '#007AFF' : '#8E8E93'"></uni-icons>
				<text class="nav-text" :class="{ active: currentTab === 'profile' }">设置</text>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				currentTab: 'courier',
				taskList: [{
						id: 1,
						location: 'A区快递柜-12号',
						time: '09:30',
						status: '已完成',
						statusClass: 'status-completed'
					},
					{
						id: 2,
						location: 'B区快递柜-05号',
						time: '10:15',
						status: '进行中',
						statusClass: 'status-inprogress'
					},
					{
						id: 3,
						location: 'C区快递柜-08号',
						time: '11:00',
						status: '待处理',
						statusClass: 'status-pending'
					},
					{
						id: 4,
						location: 'D区快递柜-03号',
						time: '14:30',
						status: '待处理',
						statusClass: 'status-pending'
					}
				]
			}
		},
		onShow() {
			// 检查登录状态
			const isLoggedIn = uni.getStorageSync('isLoggedIn')
			if (!isLoggedIn) {
				uni.redirectTo({
					url: '/pages/login/login'
				})
			}
		},
		methods: {
			scanQRCode() {
				uni.scanCode({
					success: (res) => {
						uni.showToast({
							title: '扫描成功',
							icon: 'success'
						})
						console.log('扫描结果:', res.result)
						// 这里可以处理扫描结果，比如跳转到取件页面
					},
					fail: (err) => {
						uni.showToast({
							title: '扫描失败',
							icon: 'none'
						})
					}
				})
			},
			navigateToTaskList() {
				uni.showToast({
					title: '跳转到任务列表',
					icon: 'none'
				})
				// 实际项目中这里应该是 uni.navigateTo 跳转
			},
			navigateToDelivery() {
				uni.showToast({
					title: '跳转到派送管理',
					icon: 'none'
				})
			},
			navigateToHistory() {
				uni.showToast({
					title: '跳转到历史记录',
					icon: 'none'
				})
			},
			viewTaskDetail(task) {
				uni.showModal({
					title: '任务详情',
					content: `地点：${task.location}\n时间：${task.time}\n状态：${task.status}`,
					showCancel: false
				})
			},
			startTask(task) {
				task.status = '进行中'
				task.statusClass = 'status-inprogress'
				uni.showToast({
					title: '任务已开始',
					icon: 'success'
				})
			},
			completeTask(task) {
				task.status = '已完成'
				task.statusClass = 'status-completed'
				uni.showToast({
					title: '任务已完成',
					icon: 'success'
				})
			},
			switchTab(tab) {
				this.currentTab = tab
				if (tab === 'home') {
					uni.switchTab({
						url: '/pages/index/index'
					})
				}
				// 其他tab的跳转逻辑可以在这里添加
			}
		}
	}
</script>

<style lang="scss">
	.container {
		padding-bottom: 100rpx;
		background-color: #f5f5f5;
	}

	.status-bar {
		background: linear-gradient(135deg, #007AFF, #5856D6);
		padding: 40rpx 30rpx;
		color: white;
		border-radius: 0 0 30rpx 30rpx;
	}

	.user-info {
		display: flex;
		align-items: center;
		margin-bottom: 30rpx;
	}

	.avatar {
		width: 100rpx;
		height: 100rpx;
		border-radius: 50%;
		border: 3rpx solid white;
		margin-right: 20rpx;
	}

	.user-details {
		display: flex;
		flex-direction: column;
	}

	.username {
		font-size: 36rpx;
		font-weight: bold;
		margin-bottom: 10rpx;
	}

	.user-id {
		font-size: 24rpx;
		opacity: 0.9;
	}

	.stats {
		display: flex;
		justify-content: space-between;
		margin-top: 20rpx;
	}

	.stat-item {
		display: flex;
		flex-direction: column;
		align-items: center;
		flex: 1;
	}

	.stat-value {
		font-size: 48rpx;
		font-weight: bold;
		margin-bottom: 10rpx;
	}

	.stat-label {
		font-size: 24rpx;
		opacity: 0.9;
	}

	.function-grid {
		background-color: white;
		margin: 30rpx;
		border-radius: 20rpx;
		padding: 30rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.grid-title {
		font-size: 32rpx;
		font-weight: bold;
		margin-bottom: 30rpx;
		color: #333;
	}

	.grid-container {
		display: flex;
		flex-wrap: wrap;
		justify-content: space-between;
	}

	.grid-item {
		display: flex;
		flex-direction: column;
		align-items: center;
		width: 150rpx;
		margin-bottom: 30rpx;
	}

	.grid-icon {
		width: 80rpx;
		height: 80rpx;
		background-color: #f8f8f8;
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-bottom: 15rpx;
	}

	.grid-text {
		font-size: 24rpx;
		color: #666;
	}

	.task-section {
		background-color: white;
		margin: 30rpx;
		border-radius: 20rpx;
		padding: 30rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.section-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 30rpx;
	}

	.section-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
	}

	.section-more {
		font-size: 24rpx;
		color: #007AFF;
	}

	.task-list {
		display: flex;
		flex-direction: column;
	}

	.task-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 30rpx 0;
		border-bottom: 1rpx solid #f0f0f0;
	}

	.task-item:last-child {
		border-bottom: none;
	}

	.task-info {
		display: flex;
		flex-direction: column;
		flex: 1;
	}

	.task-location {
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 10rpx;
	}

	.task-time {
		font-size: 24rpx;
		color: #666;
		margin-bottom: 10rpx;
	}

	.task-status {
		font-size: 22rpx;
		padding: 4rpx 12rpx;
		border-radius: 20rpx;
		align-self: flex-start;
	}

	.status-completed {
		background-color: #e7f7ef;
		color: #34C759;
	}

	.status-inprogress {
		background-color: #e7f0ff;
		color: #007AFF;
	}

	.status-pending {
		background-color: #fff3e0;
		color: #FF9500;
	}

	.task-actions {
		display: flex;
		gap: 15rpx;
	}

	.action-btn {
		padding: 8rpx 24rpx;
		font-size: 24rpx;
		border-radius: 20rpx;
		border: 1rpx solid #ddd;
		background-color: white;
		color: #666;
	}

	.action-btn.primary {
		background-color: #007AFF;
		color: white;
		border: none;
	}

	.action-btn.success {
		background-color: #34C759;
		color: white;
		border: none;
	}

	.bottom-nav {
		position: fixed;
		bottom: 0;
		left: 0;
		right: 0;
		background-color: white;
		display: flex;
		justify-content: space-around;
		padding: 20rpx 0;
		box-shadow: 0 -2rpx 10rpx rgba(0, 0, 0, 0.1);
		z-index: 1000;
	}

	.nav-item {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.nav-text {
		font-size: 20rpx;
		color: #8E8E93;
		margin-top: 10rpx;
	}

	.nav-text.active {
		color: #007AFF;
		font-weight: bold;
	}
</style>
