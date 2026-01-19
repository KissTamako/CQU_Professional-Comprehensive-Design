<template>
	<view class="container">
		<!-- 顶部导航栏 -->
		<uni-nav-bar title="快递柜综合应用系统" backgroundColor="#007AFF" color="#fff" fixed="true" statusBar="true">
		</uni-nav-bar>

		<!-- 用户状态栏 -->
		<view class="user-status" v-if="isLoggedIn">
			<view class="user-info">
				<image class="avatar" :src="userInfo.avatar || '/static/logo.png'"></image>
				<view class="user-details">
					<text class="username">{{ userInfo.name || '用户' }}</text>
					<text class="user-role" v-if="userInfo.role === 'courier'">快递员</text>
					<text class="user-role" v-else-if="userInfo.role === 'admin'">管理员</text>
					<text class="user-role" v-else>普通用户</text>
				</view>
			</view>
			<view class="notifications">
				<uni-icons type="bell" size="24" color="#666" @click="navigateToNotifications"></uni-icons>
				<view class="badge" v-if="unreadCount > 0">{{ unreadCount }}</view>
			</view>
		</view>

		<!-- 主功能入口 -->
		<view class="main-functions">
			<view class="section-title">主要功能</view>
			<view class="function-grid">
				<view class="function-item" @click="scanQRCode">
					<view class="function-icon" style="background-color: #34C759;">
						<uni-icons type="scan" size="30" color="#fff"></uni-icons>
					</view>
					<text class="function-label">扫码开柜</text>
				</view>
				<view class="function-item" @click="navigateTo('/pages/user/pickup')">
					<view class="function-icon" style="background-color: #007AFF;">
						<uni-icons type="cube" size="30" color="#fff"></uni-icons>
					</view>
					<text class="function-label">取快递</text>
				</view>
				<view class="function-item" @click="navigateTo('/pages/user/storage')">
					<view class="function-icon" style="background-color: #FF9500;">
						<uni-icons type="box" size="30" color="#fff"></uni-icons>
					</view>
					<text class="function-label">寄存物品</text>
				</view>
				<view class="function-item" @click="navigateTo('/pages/user/send')">
					<view class="function-icon" style="background-color: #5856D6;">
						<uni-icons type="paperplane" size="30" color="#fff"></uni-icons>
					</view>
					<text class="function-label">发快递</text>
				</view>
				<view class="function-item" @click="navigateTo('/pages/user/face')">
					<view class="function-icon" style="background-color: #FF2D55;">
						<uni-icons type="camera" size="30" color="#fff"></uni-icons>
					</view>
					<text class="function-label">人脸开柜</text>
				</view>
				<view class="function-item" @click="navigateTo('/pages/courier/courier')" v-if="userInfo.role === 'courier'">
					<view class="function-icon" style="background-color: #FF9500;">
						<uni-icons type="person" size="30" color="#fff"></uni-icons>
					</view>
					<text class="function-label">工作台</text>
				</view>
				<view class="function-item" @click="navigateTo('/pages/admin/dashboard')" v-if="userInfo.role === 'admin'">
					<view class="function-icon" style="background-color: #FF3B30;">
						<uni-icons type="settings" size="30" color="#fff"></uni-icons>
					</view>
					<text class="function-label">管理平台</text>
				</view>
				<view class="function-item" @click="navigateTo('/pages/user/register')" v-if="!isLoggedIn">
					<view class="function-icon" style="background-color: #34C759;">
						<uni-icons type="personadd" size="30" color="#fff"></uni-icons>
					</view>
					<text class="function-label">注册绑定</text>
				</view>
			</view>
		</view>

		<!-- 快速取件码输入 -->
		<view class="quick-pickup">
			<view class="section-title">快速取件</view>
			<view class="input-group">
				<input class="pickup-input" v-model="pickupCode" placeholder="请输入取件码" maxlength="6" />
				<button class="pickup-btn" :disabled="!pickupCode" @click="quickPickup">开柜取件</button>
			</view>
		</view>

		<!-- 我的快递 -->
		<view class="my-packages" v-if="isLoggedIn && packages.length > 0">
			<view class="section-header">
				<text class="section-title">我的快递</text>
				<text class="section-more" @click="navigateTo('/pages/user/pickup')">查看更多</text>
			</view>
			<view class="package-list">
				<view class="package-item" v-for="(pkg, index) in packages.slice(0, 3)" :key="index">
					<view class="package-info">
						<text class="package-location">柜号: {{ pkg.cabinetNo }}-{{ pkg.cellNo }}</text>
						<text class="package-time">放入时间: {{ pkg.time }}</text>
						<text class="package-status" :class="'status-' + pkg.status">{{ pkg.statusText }}</text>
					</view>
					<button class="action-btn" size="mini" @click="openCabinet(pkg)" v-if="pkg.status === 'ready'">开柜</button>
				</view>
			</view>
		</view>

		<!-- 系统公告 -->
		<view class="announcement">
			<uni-notice-bar :text="announcement" background-color="#fff" color="#666" />
		</view>

		<!-- 底部提示 -->
		<view class="bottom-tips">
			<text class="tip-text">遇到问题? 请联系客服: 400-123-4567</text>
			<text class="tip-text">服务时间: 08:00-22:00</text>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				isLoggedIn: false,
				userInfo: {
					name: '',
					avatar: '',
					role: 'user'
				},
				unreadCount: 2,
				pickupCode: '',
				packages: [{
						cabinetNo: 'A01',
						cellNo: '12',
						time: '2024-01-18 10:30',
						status: 'ready',
						statusText: '待取件'
					},
					{
						cabinetNo: 'B03',
						cellNo: '05',
						time: '2024-01-18 09:15',
						status: 'ready',
						statusText: '待取件'
					},
					{
						cabinetNo: 'C02',
						cellNo: '08',
						time: '2024-01-17 16:45',
						status: 'expired',
						statusText: '已过期'
					}
				],
				announcement: '系统维护通知：本周六凌晨2:00-4:00进行系统升级，期间服务可能短暂中断。'
			}
		},
		onShow() {
			this.checkLoginStatus()
		},
		methods: {
			checkLoginStatus() {
				const isLoggedIn = uni.getStorageSync('isLoggedIn')
				this.isLoggedIn = isLoggedIn
				if (isLoggedIn) {
					const user = uni.getStorageSync('userInfo') || {}
					this.userInfo = {
						name: user.name || '用户',
						avatar: user.avatar || '',
						role: user.role || 'user'
					}
				} else {
					this.userInfo = {
						name: '',
						avatar: '',
						role: 'user'
					}
				}
			},
			scanQRCode() {
				uni.scanCode({
					success: (res) => {
						console.log('扫描结果:', res.result)
						// 根据扫描结果处理不同的逻辑
						if (res.result.includes('pickup')) {
							this.navigateTo('/pages/user/pickup')
						} else if (res.result.includes('storage')) {
							this.navigateTo('/pages/user/storage')
						} else if (res.result.includes('send')) {
							this.navigateTo('/pages/user/send')
						} else {
							uni.showModal({
								title: '扫描结果',
								content: res.result,
								showCancel: false
							})
						}
					},
					fail: (err) => {
						uni.showToast({
							title: '扫描失败',
							icon: 'none'
						})
					}
				})
			},
			navigateTo(url) {
				if (!this.isLoggedIn && url !== '/pages/user/register') {
					uni.showModal({
						title: '提示',
						content: '请先登录或注册',
						confirmText: '去登录',
						cancelText: '取消',
						success: (res) => {
							if (res.confirm) {
								uni.navigateTo({
									url: '/pages/login/login'
								})
							}
						}
					})
					return
				}
				uni.navigateTo({
					url: url
				})
			},
			navigateToNotifications() {
				uni.showToast({
					title: '跳转到通知页面',
					icon: 'none'
				})
			},
			quickPickup() {
				if (!this.pickupCode) {
					uni.showToast({
						title: '请输入取件码',
						icon: 'none'
					})
					return
				}

				// 模拟开柜请求
				uni.showLoading({
					title: '开柜中...'
				})

				setTimeout(() => {
					uni.hideLoading()
					uni.showModal({
						title: '开柜成功',
						content: `柜门 ${this.pickupCode.substring(0, 2)}-${this.pickupCode.substring(2, 4)} 已打开`,
						showCancel: false,
						success: () => {
							this.pickupCode = ''
						}
					})
				}, 1500)
			},
			openCabinet(pkg) {
				uni.showModal({
					title: '确认开柜',
					content: `确定要打开 ${pkg.cabinetNo}-${pkg.cellNo} 号柜门吗？`,
					success: (res) => {
						if (res.confirm) {
							uni.showToast({
								title: '柜门已打开',
								icon: 'success'
							})
						}
					}
				})
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

	.user-status {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 30rpx;
		background: linear-gradient(135deg, #007AFF, #5856D6);
		margin-top: 80rpx;
		border-radius: 0 0 30rpx 30rpx;
		color: white;
	}

	.user-info {
		display: flex;
		align-items: center;
	}

	.avatar {
		width: 80rpx;
		height: 80rpx;
		border-radius: 50%;
		border: 3rpx solid white;
		margin-right: 20rpx;
	}

	.user-details {
		display: flex;
		flex-direction: column;
	}

	.username {
		font-size: 32rpx;
		font-weight: bold;
		margin-bottom: 10rpx;
	}

	.user-role {
		font-size: 24rpx;
		opacity: 0.9;
	}

	.notifications {
		position: relative;
		padding: 10rpx;
	}

	.badge {
		position: absolute;
		top: -5rpx;
		right: -5rpx;
		background-color: #FF3B30;
		color: white;
		font-size: 20rpx;
		width: 32rpx;
		height: 32rpx;
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.main-functions {
		padding: 30rpx;
		background-color: white;
		margin: 30rpx;
		border-radius: 20rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.section-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 30rpx;
	}

	.function-grid {
		display: flex;
		flex-wrap: wrap;
		justify-content: space-between;
	}

	.function-item {
		display: flex;
		flex-direction: column;
		align-items: center;
		width: 150rpx;
		margin-bottom: 30rpx;
	}

	.function-icon {
		width: 80rpx;
		height: 80rpx;
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-bottom: 15rpx;
	}

	.function-label {
		font-size: 24rpx;
		color: #666;
	}

	.quick-pickup {
		padding: 30rpx;
		background-color: white;
		margin: 0 30rpx 30rpx;
		border-radius: 20rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.input-group {
		display: flex;
		align-items: center;
	}

	.pickup-input {
		flex: 1;
		height: 80rpx;
		border: 2rpx solid #e0e0e0;
		border-radius: 12rpx;
		padding: 0 30rpx;
		font-size: 28rpx;
		margin-right: 20rpx;
	}

	.pickup-btn {
		width: 200rpx;
		height: 80rpx;
		background: linear-gradient(135deg, #007AFF, #5856D6);
		color: white;
		border: none;
		border-radius: 12rpx;
		font-size: 28rpx;
	}

	.my-packages {
		padding: 30rpx;
		background-color: white;
		margin: 0 30rpx 30rpx;
		border-radius: 20rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.section-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 30rpx;
	}

	.section-more {
		font-size: 24rpx;
		color: #007AFF;
	}

	.package-list {
		display: flex;
		flex-direction: column;
	}

	.package-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 30rpx 0;
		border-bottom: 1rpx solid #f0f0f0;
	}

	.package-item:last-child {
		border-bottom: none;
	}

	.package-info {
		display: flex;
		flex-direction: column;
		flex: 1;
	}

	.package-location {
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 10rpx;
	}

	.package-time {
		font-size: 24rpx;
		color: #666;
		margin-bottom: 10rpx;
	}

	.package-status {
		font-size: 22rpx;
		padding: 4rpx 12rpx;
		border-radius: 20rpx;
		align-self: flex-start;
	}

	.status-ready {
		background-color: #e7f7ef;
		color: #34C759;
	}

	.status-expired {
		background-color: #ffeaea;
		color: #FF3B30;
	}

	.action-btn {
		padding: 8rpx 24rpx;
		font-size: 24rpx;
		border-radius: 20rpx;
		background-color: #007AFF;
		color: white;
		border: none;
	}

	.announcement {
		margin: 0 30rpx 30rpx;
	}

	.bottom-tips {
		text-align: center;
		font-size: 22rpx;
		color: #999;
		display: flex;
		flex-direction: column;
		gap: 10rpx;
	}
</style>
