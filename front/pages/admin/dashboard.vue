<template>
	<view class="container">
		<!-- 顶部导航栏 -->
		<uni-nav-bar title="管理平台" backgroundColor="#007AFF" color="#fff" fixed="true" statusBar="true">
		</uni-nav-bar>

		<!-- 统计概览 -->
		<view class="stats-overview">
			<view class="stats-grid">
				<view class="stat-card">
					<view class="stat-icon" style="background-color: #34C759;">
						<uni-icons type="cube" size="24" color="#fff"></uni-icons>
					</view>
					<text class="stat-value">{{ overviewData.cabinets }}</text>
					<text class="stat-label">快递柜总数</text>
				</view>
				<view class="stat-card">
					<view class="stat-icon" style="background-color: #007AFF;">
						<uni-icons type="person" size="24" color="#fff"></uni-icons>
					</view>
					<text class="stat-value">{{ overviewData.users }}</text>
					<text class="stat-label">注册用户</text>
				</view>
				<view class="stat-card">
					<view class="stat-icon" style="background-color: #FF9500;">
						<uni-icons type="staff" size="24" color="#fff"></uni-icons>
					</view>
					<text class="stat-value">{{ overviewData.couriers }}</text>
					<text class="stat-label">快递员</text>
				</view>
				<view class="stat-card">
					<view class="stat-icon" style="background-color: #FF3B30;">
						<uni-icons type="flag" size="24" color="#fff"></uni-icons>
					</view>
					<text class="stat-value">{{ overviewData.activeOrders }}</text>
					<text class="stat-label">今日订单</text>
				</view>
			</view>
		</view>

		<!-- 功能模块 -->
		<view class="function-modules">
			<view class="module-title">功能管理</view>
			<view class="module-grid">
				<view class="module-item" @click="navigateTo('cabinet')">
					<view class="module-icon" style="background-color: #34C759;">
						<uni-icons type="home" size="30" color="#fff"></uni-icons>
					</view>
					<text class="module-text">快递柜管理</text>
				</view>
				<view class="module-item" @click="navigateTo('user')">
					<view class="module-icon" style="background-color: #007AFF;">
						<uni-icons type="person" size="30" color="#fff"></uni-icons>
					</view>
					<text class="module-text">用户管理</text>
				</view>
				<view class="module-item" @click="navigateTo('courier')">
					<view class="module-icon" style="background-color: #FF9500;">
						<uni-icons type="staff" size="30" color="#fff"></uni-icons>
					</view>
					<text class="module-text">快递员管理</text>
				</view>
				<view class="module-item" @click="navigateTo('order')">
					<view class="module-icon" style="background-color: #5856D6;">
						<uni-icons type="list" size="30" color="#fff"></uni-icons>
					</view>
					<text class="module-text">订单管理</text>
				</view>
				<view class="module-item" @click="navigateTo('storage')">
					<view class="module-icon" style="background-color: #FF2D55;">
						<uni-icons type="box" size="30" color="#fff"></uni-icons>
					</view>
					<text class="module-text">寄存管理</text>
				</view>
				<view class="module-item" @click="navigateTo('analysis')">
					<view class="module-icon" style="background-color: #5AC8FA;">
						<uni-icons type="chart" size="30" color="#fff"></uni-icons>
					</view>
					<text class="module-text">统计分析</text>
				</view>
				<view class="module-item" @click="navigateTo('monitor')">
					<view class="module-icon" style="background-color: #4CD964;">
						<uni-icons type="eye" size="30" color="#fff"></uni-icons>
					</view>
					<text class="module-text">实时监控</text>
				</view>
				<view class="module-item" @click="navigateTo('settings')">
					<view class="module-icon" style="background-color: #8E8E93;">
						<uni-icons type="gear" size="30" color="#fff"></uni-icons>
					</view>
					<text class="module-text">系统设置</text>
				</view>
			</view>
		</view>

		<!-- 快递柜状态 -->
		<view class="cabinet-status">
			<view class="section-header">
				<text class="section-title">快递柜状态</text>
				<text class="section-more" @click="viewAllCabinets">查看更多</text>
			</view>
			<view class="cabinet-list">
				<view class="cabinet-item" v-for="cabinet in cabinetList" :key="cabinet.id">
					<view class="cabinet-info">
						<text class="cabinet-name">{{ cabinet.name }}</text>
						<text class="cabinet-location">{{ cabinet.location }}</text>
					</view>
					<view class="cabinet-stats">
						<text class="stat">使用率: {{ cabinet.usageRate }}</text>
						<text class="stat">电量: {{ cabinet.power }}</text>
					</view>
					<view class="cabinet-status-indicator" :class="cabinet.status">
						<text>{{ cabinet.statusText }}</text>
					</view>
				</view>
			</view>
		</view>

		<!-- 今日订单统计 -->
		<view class="order-stats">
			<view class="section-header">
				<text class="section-title">今日订单统计</text>
				<text class="section-more" @click="viewOrderReport">查看报表</text>
			</view>
			<view class="order-chart">
				<view class="chart-placeholder">
					<text class="chart-text">订单统计图表</text>
					<text class="chart-subtext">快递: {{ orderStats.delivery }} | 寄存: {{ orderStats.storage }} | 寄件: {{ orderStats.send }}</text>
				</view>
			</view>
			<view class="order-summary">
				<view class="summary-item">
					<text class="summary-label">总收入</text>
					<text class="summary-value">¥{{ orderStats.totalIncome }}</text>
				</view>
				<view class="summary-item">
					<text class="summary-label">总订单</text>
					<text class="summary-value">{{ orderStats.totalOrders }}</text>
				</view>
				<view class="summary-item">
					<text class="summary-label">完成率</text>
					<text class="summary-value">{{ orderStats.completionRate }}</text>
				</view>
			</view>
		</view>

		<!-- 系统公告 -->
		<view class="system-notice">
			<view class="notice-header">
				<uni-icons type="sound" size="24" color="#FF9500"></uni-icons>
				<text class="notice-title">系统公告</text>
			</view>
			<view class="notice-content">
				<text class="notice-text">系统维护通知：本周六凌晨2:00-4:00进行系统升级，期间服务可能短暂中断。</text>
			</view>
		</view>

		<!-- 快速操作 -->
		<view class="quick-actions">
			<view class="action-title">快速操作</view>
			<view class="action-buttons">
				<button class="action-btn" @click="remoteOpen">远程开仓</button>
				<button class="action-btn" @click="addCabinet">添加快递柜</button>
				<button class="action-btn" @click="generateReport">生成报表</button>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				overviewData: {
					cabinets: 156,
					users: 8924,
					couriers: 203,
					activeOrders: 342
				},
				cabinetList: [{
						id: 1,
						name: 'A区快递柜-01',
						location: '1号楼大厅',
						usageRate: '85%',
						power: '正常',
						status: 'normal',
						statusText: '正常'
					},
					{
						id: 2,
						name: 'B区快递柜-02',
						location: '2号楼门口',
						usageRate: '92%',
						power: '正常',
						status: 'warning',
						statusText: '高负载'
					},
					{
						id: 3,
						name: 'C区快递柜-03',
						location: '图书馆侧门',
						usageRate: '45%',
						power: '正常',
						status: 'normal',
						statusText: '正常'
					},
					{
						id: 4,
						name: 'D区快递柜-04',
						location: '食堂门口',
						usageRate: '78%',
						power: '故障',
						status: 'error',
						statusText: '故障'
					}
				],
				orderStats: {
					delivery: 156,
					storage: 89,
					send: 97,
					totalIncome: '4,568.00',
					totalOrders: 342,
					completionRate: '98%'
				}
			}
		},
		onLoad() {
			this.checkAdminAuth()
		},
		methods: {
			checkAdminAuth() {
				const userInfo = uni.getStorageSync('userInfo')
				if (!userInfo || userInfo.role !== 'admin') {
					uni.showModal({
						title: '权限不足',
						content: '您没有管理员权限，无法访问此页面',
						showCancel: false,
						success: () => {
							uni.switchTab({
								url: '/pages/index/index'
							})
						}
					})
				}
			},
			navigateTo(module) {
				uni.showToast({
					title: `跳转到${this.getModuleName(module)}`,
					icon: 'none'
				})
				// 实际项目中这里应该是 uni.navigateTo 跳转到具体页面
			},
			getModuleName(module) {
				const names = {
					'cabinet': '快递柜管理',
					'user': '用户管理',
					'courier': '快递员管理',
					'order': '订单管理',
					'storage': '寄存管理',
					'analysis': '统计分析',
					'monitor': '实时监控',
					'settings': '系统设置'
				}
				return names[module] || '未知模块'
			},
			viewAllCabinets() {
				uni.showToast({
					title: '查看所有快递柜',
					icon: 'none'
				})
			},
			viewOrderReport() {
				uni.showToast({
					title: '查看订单报表',
					icon: 'none'
				})
			},
			remoteOpen() {
				uni.showModal({
					title: '远程开仓',
					content: '请选择要打开的仓门',
					editable: true,
					placeholderText: '请输入仓门编号',
					success: (res) => {
						if (res.confirm && res.content) {
							uni.showLoading({
								title: '开仓中...'
							})
							setTimeout(() => {
								uni.hideLoading()
								uni.showToast({
									title: `仓门 ${res.content} 已打开`,
									icon: 'success'
								})
							}, 1500)
						}
					}
				})
			},
			addCabinet() {
				uni.showModal({
					title: '添加快递柜',
					content: '请输入新快递柜信息',
					editable: true,
					placeholderText: '快递柜名称/位置',
					success: (res) => {
						if (res.confirm && res.content) {
							uni.showToast({
								title: '已添加快递柜',
								icon: 'success'
							})
						}
					}
				})
			},
			generateReport() {
				uni.showLoading({
					title: '生成报表中...'
				})
				setTimeout(() => {
					uni.hideLoading()
					uni.showModal({
						title: '报表生成成功',
						content: '报表已生成，是否下载？',
						success: (res) => {
							if (res.confirm) {
								uni.showToast({
									title: '开始下载',
									icon: 'success'
								})
							}
						}
					})
				}, 2000)
			}
		}
	}
</script>

<style lang="scss">
	.container {
		padding-bottom: 40rpx;
		background-color: #f5f5f5;
		min-height: 100vh;
	}

	.stats-overview {
		padding: 30rpx;
		margin-top: 80rpx;
	}

	.stats-grid {
		display: grid;
		grid-template-columns: repeat(2, 1fr);
		gap: 20rpx;
	}

	.stat-card {
		background-color: white;
		border-radius: 20rpx;
		padding: 30rpx;
		text-align: center;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.stat-icon {
		width: 60rpx;
		height: 60rpx;
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		margin: 0 auto 20rpx;
	}

	.stat-value {
		display: block;
		font-size: 36rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 10rpx;
	}

	.stat-label {
		display: block;
		font-size: 24rpx;
		color: #666;
	}

	.function-modules {
		padding: 0 30rpx;
	}

	.module-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 30rpx;
	}

	.module-grid {
		display: grid;
		grid-template-columns: repeat(4, 1fr);
		gap: 20rpx;
	}

	.module-item {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.module-icon {
		width: 80rpx;
		height: 80rpx;
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-bottom: 15rpx;
	}

	.module-text {
		font-size: 22rpx;
		color: #666;
		text-align: center;
	}

	.cabinet-status {
		padding: 30rpx;
		background-color: white;
		margin: 30rpx;
		border-radius: 20rpx;
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

	.cabinet-list {
		display: flex;
		flex-direction: column;
		gap: 20rpx;
	}

	.cabinet-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 25rpx;
		background-color: #f8f8f8;
		border-radius: 12rpx;
	}

	.cabinet-info {
		display: flex;
		flex-direction: column;
		flex: 1;
	}

	.cabinet-name {
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 10rpx;
	}

	.cabinet-location {
		font-size: 24rpx;
		color: #666;
	}

	.cabinet-stats {
		display: flex;
		flex-direction: column;
		align-items: flex-end;
		margin-right: 20rpx;
	}

	.stat {
		font-size: 22rpx;
		color: #666;
		margin-bottom: 5rpx;
	}

	.cabinet-status-indicator {
		padding: 8rpx 16rpx;
		border-radius: 20rpx;
		font-size: 22rpx;
	}

	.cabinet-status-indicator.normal {
		background-color: #e7f7ef;
		color: #34C759;
	}

	.cabinet-status-indicator.warning {
		background-color: #fff3e0;
		color: #FF9500;
	}

	.cabinet-status-indicator.error {
		background-color: #ffeaea;
		color: #FF3B30;
	}

	.order-stats {
		padding: 30rpx;
		background-color: white;
		margin: 0 30rpx 30rpx;
		border-radius: 20rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.order-chart {
		height: 200rpx;
		background-color: #f8f8f8;
		border-radius: 12rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-bottom: 30rpx;
	}

	.chart-placeholder {
		text-align: center;
	}

	.chart-text {
		display: block;
		font-size: 28rpx;
		color: #666;
		margin-bottom: 10rpx;
	}

	.chart-subtext {
		display: block;
		font-size: 22rpx;
		color: #999;
	}

	.order-summary {
		display: flex;
		justify-content: space-between;
	}

	.summary-item {
		display: flex;
		flex-direction: column;
		align-items: center;
		flex: 1;
	}

	.summary-label {
		font-size: 24rpx;
		color: #666;
		margin-bottom: 10rpx;
	}

	.summary-value {
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
	}

	.system-notice {
		padding: 30rpx;
		background-color: #fffbe6;
		margin: 0 30rpx 30rpx;
		border-radius: 20rpx;
	}

	.notice-header {
		display: flex;
		align-items: center;
		margin-bottom: 15rpx;
	}

	.notice-title {
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
		margin-left: 10rpx;
	}

	.notice-content {
		font-size: 26rpx;
		color: #666;
		line-height: 1.5;
	}

	.quick-actions {
		padding: 30rpx;
		background-color: white;
		margin: 0 30rpx;
		border-radius: 20rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.action-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 30rpx;
	}

	.action-buttons {
		display: flex;
		justify-content: space-between;
	}

	.action-btn {
		flex: 1;
		height: 80rpx;
		margin: 0 10rpx;
		border-radius: 12rpx;
		border: 2rpx solid #007AFF;
		background-color: white;
		color: #007AFF;
		font-size: 28rpx;
	}

	.action-btn:first-child {
		margin-left: 0;
	}

	.action-btn:last-child {
		margin-right: 0;
	}
</style>
