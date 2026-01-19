<template>
	<view class="container">
		<!-- 顶部导航栏 -->
		<uni-nav-bar title="人脸识别开柜" left-icon="back" @clickLeft="goBack" backgroundColor="#007AFF" color="#fff" fixed="true"
			statusBar="true">
		</uni-nav-bar>

		<!-- 人脸识别区域 -->
		<view class="face-scan-section">
			<view class="scan-card">
				<view class="camera-preview">
					<image class="camera-placeholder" src="/static/face-scan.png"></image>
					<view class="scan-overlay">
						<view class="scan-frame"></view>
						<view class="scan-guide">
							<text class="guide-text">请将面部对准框内</text>
						</view>
					</view>
				</view>

				<view class="scan-status">
					<text class="status-text" v-if="!isScanning">准备扫描</text>
					<text class="status-text scanning" v-if="isScanning">识别中...</text>
					<text class="status-text success" v-if="scanSuccess">识别成功！</text>
					<text class="status-text error" v-if="scanError">识别失败，请重试</text>
				</view>

				<view class="scan-controls">
					<button class="scan-btn" :class="{ scanning: isScanning }" @click="startFaceScan" :disabled="isScanning">
						<text class="btn-text">{{ isScanning ? '识别中...' : '开始人脸识别' }}</text>
					</button>
				</view>
			</view>
		</view>

		<!-- 识别结果 -->
		<view class="result-section" v-if="scanSuccess">
			<view class="result-card">
				<view class="user-info">
					<image class="user-avatar" :src="userInfo.avatar || '/static/logo.png'"></image>
					<view class="user-details">
						<text class="user-name">{{ userInfo.name }}</text>
						<text class="user-phone">{{ userInfo.phone }}</text>
					</view>
				</view>

				<view class="package-list" v-if="userPackages.length > 0">
					<text class="list-title">您的快递</text>
					<view class="package-item" v-for="(pkg, index) in userPackages" :key="index">
						<view class="package-details">
							<text class="package-location">{{ pkg.cabinetName }} {{ pkg.cellNumber }}号仓</text>
							<text class="package-time">存入时间: {{ pkg.storageTime }}</text>
						</view>
						<button class="open-btn" size="mini" @click="openCabinet(pkg)">开柜</button>
					</view>
				</view>

				<view class="empty-packages" v-else>
					<text class="empty-text">您当前没有待取的快递</text>
				</view>
			</view>
		</view>

		<!-- 快速操作 -->
		<view class="quick-actions">
			<view class="action-title">其他开柜方式</view>
			<view class="action-grid">
				<view class="action-item" @click="navigateTo('/pages/user/pickup')">
					<view class="action-icon">
						<uni-icons type="key" size="30" color="#007AFF"></uni-icons>
					</view>
					<text class="action-text">取件码开柜</text>
				</view>
				<view class="action-item" @click="navigateTo('/pages/index/index')">
					<view class="action-icon">
						<uni-icons type="scan" size="30" color="#34C759"></uni-icons>
					</view>
					<text class="action-text">扫码开柜</text>
				</view>
				<view class="action-item" @click="navigateTo('/pages/user/register')">
					<view class="action-icon">
						<uni-icons type="personadd" size="30" color="#FF9500"></uni-icons>
					</view>
					<text class="action-text">注册绑定</text>
				</view>
			</view>
		</view>

		<!-- 使用说明 -->
		<view class="instructions">
			<view class="instructions-title">使用说明</view>
			<view class="instructions-list">
				<view class="instruction-item">
					<view class="step-number">1</view>
					<text class="step-text">确保光线充足，正对摄像头</text>
				</view>
				<view class="instruction-item">
					<view class="step-number">2</view>
					<text class="step-text">请勿遮挡面部，保持自然表情</text>
				</view>
				<view class="instruction-item">
					<view class="step-number">3</view>
					<text class="step-text">识别成功后自动显示您的快递</text>
				</view>
				<view class="instruction-item">
					<view class="step-number">4</view>
					<text class="step-text">点击"开柜"按钮即可打开对应仓门</text>
				</view>
			</view>
		</view>

		<!-- 安全提示 -->
		<view class="safety-tips">
			<uni-notice-bar :text="safetyTip" background-color="#fffbe6" color="#666" />
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				isScanning: false,
				scanSuccess: false,
				scanError: false,
				userInfo: {
					name: '',
					phone: '',
					avatar: ''
				},
				userPackages: [],
				safetyTip: '安全提示：人脸识别仅用于开柜验证，系统不会存储您的面部图像。如有问题请联系客服400-123-4567。'
			}
		},
		onLoad() {
			// 页面加载时检查是否已登录
			this.checkLogin()
		},
		methods: {
			goBack() {
				uni.navigateBack()
			},
			checkLogin() {
				// 检查是否已登录，如果未登录可以提示
				const isLoggedIn = uni.getStorageSync('isLoggedIn')
				if (!isLoggedIn) {
					// 未登录也可以使用人脸识别，因为人脸识别本身就是一种身份验证
				}
			},
			startFaceScan() {
				if (this.isScanning) return

				this.isScanning = true
				this.scanSuccess = false
				this.scanError = false

				// 模拟人脸识别过程
				setTimeout(() => {
					this.isScanning = false

					// 模拟识别成功
					const success = Math.random() > 0.3 // 70%成功率

					if (success) {
						this.scanSuccess = true
						this.scanError = false

						// 模拟获取用户信息
						this.userInfo = {
							name: '张三',
							phone: '138****1234',
							avatar: '/static/logo.png'
						}

						// 模拟获取用户的快递
						this.userPackages = [{
								cabinetName: 'A区快递柜',
								cellNumber: '12',
								storageTime: '2024-01-18 10:30',
								pickupCode: '123456'
							},
							{
								cabinetName: 'B区快递柜',
								cellNumber: '05',
								storageTime: '2024-01-18 09:15',
								pickupCode: '654321'
							}
						]

						uni.showToast({
							title: '识别成功',
							icon: 'success'
						})
					} else {
						this.scanSuccess = false
						this.scanError = true
						uni.showToast({
							title: '识别失败，请重试',
							icon: 'none'
						})
					}
				}, 2000)
			},
			openCabinet(pkg) {
				uni.showModal({
					title: '确认开柜',
					content: `确定要打开 ${pkg.cabinetName} ${pkg.cellNumber}号仓门吗？`,
					success: (res) => {
						if (res.confirm) {
							uni.showLoading({
								title: '开柜中...'
							})

							setTimeout(() => {
								uni.hideLoading()
								uni.showModal({
									title: '开柜成功',
									content: `${pkg.cabinetName} ${pkg.cellNumber}号仓门已打开，请取走快递`,
									showCancel: false
								})

								// 从列表中移除已开柜的快递
								this.userPackages = this.userPackages.filter(item =>
									!(item.cabinetName === pkg.cabinetName && item.cellNumber === pkg.cellNumber)
								)
							}, 1500)
						}
					}
				})
			},
			navigateTo(url) {
				uni.navigateTo({
					url: url
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

	.face-scan-section {
		padding: 30rpx;
		margin-top: 80rpx;
	}

	.scan-card {
		background-color: white;
		border-radius: 20rpx;
		padding: 40rpx 30rpx;
		text-align: center;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.camera-preview {
		position: relative;
		width: 100%;
		height: 400rpx;
		background-color: #000;
		border-radius: 20rpx;
		overflow: hidden;
		margin-bottom: 40rpx;
	}

	.camera-placeholder {
		width: 100%;
		height: 100%;
		opacity: 0.8;
	}

	.scan-overlay {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.scan-frame {
		width: 300rpx;
		height: 300rpx;
		border: 4rpx solid #007AFF;
		border-radius: 20rpx;
		background-color: transparent;
	}

	.scan-guide {
		position: absolute;
		bottom: 40rpx;
		left: 0;
		width: 100%;
		text-align: center;
	}

	.guide-text {
		color: white;
		font-size: 28rpx;
		background-color: rgba(0, 0, 0, 0.5);
		padding: 10rpx 30rpx;
		border-radius: 20rpx;
	}

	.scan-status {
		margin-bottom: 40rpx;
	}

	.status-text {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
	}

	.status-text.scanning {
		color: #FF9500;
	}

	.status-text.success {
		color: #34C759;
	}

	.status-text.error {
		color: #FF3B30;
	}

	.scan-controls {
		margin-top: 30rpx;
	}

	.scan-btn {
		width: 100%;
		height: 90rpx;
		background: linear-gradient(135deg, #007AFF, #5856D6);
		border-radius: 12rpx;
		border: none;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.scan-btn.scanning {
		background: linear-gradient(135deg, #FF9500, #FF5E3A);
	}

	.btn-text {
		font-size: 32rpx;
		font-weight: bold;
		color: white;
	}

	.result-section {
		padding: 0 30rpx;
		margin-top: 30rpx;
	}

	.result-card {
		background-color: white;
		border-radius: 20rpx;
		padding: 30rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.user-info {
		display: flex;
		align-items: center;
		margin-bottom: 30rpx;
		padding-bottom: 30rpx;
		border-bottom: 1rpx solid #f0f0f0;
	}

	.user-avatar {
		width: 80rpx;
		height: 80rpx;
		border-radius: 50%;
		margin-right: 20rpx;
	}

	.user-details {
		display: flex;
		flex-direction: column;
	}

	.user-name {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 10rpx;
	}

	.user-phone {
		font-size: 26rpx;
		color: #666;
	}

	.package-list {
		text-align: left;
	}

	.list-title {
		display: block;
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 20rpx;
	}

	.package-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 25rpx 0;
		border-bottom: 1rpx solid #f0f0f0;
	}

	.package-item:last-child {
		border-bottom: none;
	}

	.package-details {
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
	}

	.open-btn {
		padding: 8rpx 24rpx;
		font-size: 24rpx;
		border-radius: 20rpx;
		background-color: #007AFF;
		color: white;
		border: none;
	}

	.empty-packages {
		text-align: center;
		padding: 50rpx 0;
	}

	.empty-text {
		font-size: 26rpx;
		color: #999;
	}

	.quick-actions {
		padding: 30rpx;
	}

	.action-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 30rpx;
	}

	.action-grid {
		display: flex;
		justify-content: space-between;
	}

	.action-item {
		display: flex;
		flex-direction: column;
		align-items: center;
		width: 200rpx;
	}

	.action-icon {
		width: 80rpx;
		height: 80rpx;
		background-color: #f8f8f8;
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-bottom: 15rpx;
	}

	.action-text {
		font-size: 24rpx;
		color: #666;
	}

	.instructions {
		padding: 30rpx;
		background-color: white;
		margin: 0 30rpx;
		border-radius: 20rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.instructions-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 30rpx;
	}

	.instructions-list {
		display: flex;
		flex-direction: column;
		gap: 25rpx;
	}

	.instruction-item {
		display: flex;
		align-items: flex-start;
	}

	.step-number {
		width: 40rpx;
		height: 40rpx;
		background: linear-gradient(135deg, #007AFF, #5856D6);
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		color: white;
		font-size: 24rpx;
		font-weight: bold;
		margin-right: 20rpx;
		flex-shrink: 0;
	}

	.step-text {
		font-size: 26rpx;
		color: #666;
		line-height: 1.5;
		padding-top: 5rpx;
	}

	.safety-tips {
		margin: 30rpx;
	}
</style>
