<template>
	<view class="container">
		<!-- 顶部导航栏 -->
		<uni-nav-bar title="取快递" left-icon="back" @clickLeft="goBack" backgroundColor="#007AFF" color="#fff" fixed="true"
			statusBar="true">
		</uni-nav-bar>

		<!-- 扫描和输入区域 -->
		<view class="scan-section">
			<view class="scan-card">
				<view class="scan-header">
					<uni-icons type="scan" size="30" color="#007AFF"></uni-icons>
					<text class="scan-title">扫描柜门二维码</text>
				</view>
				<button class="scan-btn" @click="scanQRCode">
					<text class="scan-btn-text">开始扫描</text>
				</button>
			</view>

			<view class="or-divider">
				<view class="divider-line"></view>
				<text class="or-text">或</text>
				<view class="divider-line"></view>
			</view>

			<view class="input-card">
				<view class="input-header">
					<uni-icons type="key" size="30" color="#FF9500"></uni-icons>
					<text class="input-title">输入取件码开柜</text>
				</view>
				<view class="code-input-group">
					<input class="code-input" v-model="pickupCode" placeholder="请输入6位取件码" maxlength="6" type="number" />
					<button class="open-btn" :disabled="!isValidCode" @click="openByCode">开柜</button>
				</view>
			</view>
		</view>

		<!-- 我的快递列表 -->
		<view class="packages-section" v-if="packages.length > 0">
			<view class="section-header">
				<text class="section-title">我的快递 ({{ packages.length }})</text>
				<text class="refresh-btn" @click="refreshPackages">
					<uni-icons type="refresh" size="20" color="#007AFF"></uni-icons>
					刷新
				</text>
			</view>
			<view class="packages-list">
				<view class="package-card" v-for="(pkg, index) in packages" :key="index">
					<view class="package-header">
						<text class="package-id">快递单号: {{ pkg.trackingNumber }}</text>
						<text class="package-status" :class="'status-' + pkg.status">{{ pkg.statusText }}</text>
					</view>
					<view class="package-info">
						<view class="info-row">
							<uni-icons type="location" size="20" color="#666"></uni-icons>
							<text class="info-text">柜体位置: {{ pkg.cabinetName }} {{ pkg.cellNumber }}号仓</text>
						</view>
						<view class="info-row">
							<uni-icons type="calendar" size="20" color="#666"></uni-icons>
							<text class="info-text">存入时间: {{ pkg.storageTime }}</text>
						</view>
						<view class="info-row">
							<uni-icons type="person" size="20" color="#666"></uni-icons>
							<text class="info-text">收件人: {{ pkg.recipientName }} {{ pkg.recipientPhone }}</text>
						</view>
						<view class="info-row" v-if="pkg.pickupCode">
							<uni-icons type="key" size="20" color="#666"></uni-icons>
							<text class="info-text">取件码: {{ pkg.pickupCode }}</text>
						</view>
					</view>
					<view class="package-actions">
						<button class="action-btn" size="mini" @click="viewPackageDetail(pkg)">详情</button>
						<button class="action-btn primary" size="mini" @click="openCabinet(pkg)"
							v-if="pkg.status === 'ready'">开柜取件</button>
						<button class="action-btn disabled" size="mini" disabled v-if="pkg.status === 'picked'">已取件</button>
						<button class="action-btn expired" size="mini" disabled v-if="pkg.status === 'expired'">已过期</button>
					</view>
				</view>
			</view>
		</view>

		<!-- 空状态 -->
		<view class="empty-state" v-else>
			<image class="empty-image" src="/static/empty-box.png"></image>
			<text class="empty-title">暂无快递</text>
			<text class="empty-desc">您当前没有待取的快递</text>
		</view>

		<!-- 操作指引 -->
		<view class="guide-section">
			<view class="guide-title">取件指引</view>
			<view class="guide-steps">
				<view class="step">
					<view class="step-number">1</view>
					<text class="step-text">扫描快递柜上的二维码，或输入取件码</text>
				</view>
				<view class="step">
					<view class="step-number">2</view>
					<text class="step-text">系统会自动识别您的快递所在仓门</text>
				</view>
				<view class="step">
					<view class="step-number">3</view>
					<text class="step-text">点击"开柜取件"按钮，仓门自动打开</text>
				</view>
				<view class="step">
					<view class="step-number">4</view>
					<text class="step-text">取走快递后，请关闭仓门</text>
				</view>
			</view>
		</view>

		<!-- 底部安全提示 -->
		<view class="safety-tips">
			<uni-notice-bar :text="safetyTip" background-color="#fffbe6" color="#666" />
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				pickupCode: '',
				packages: [{
						trackingNumber: 'YT1234567890',
						cabinetName: 'A区快递柜',
						cellNumber: '12',
						storageTime: '2024-01-18 10:30:22',
						recipientName: '张三',
						recipientPhone: '138****1234',
						pickupCode: '123456',
						status: 'ready',
						statusText: '待取件'
					},
					{
						trackingNumber: 'SF9876543210',
						cabinetName: 'B区快递柜',
						cellNumber: '05',
						storageTime: '2024-01-18 09:15:10',
						recipientName: '张三',
						recipientPhone: '138****1234',
						pickupCode: '654321',
						status: 'ready',
						statusText: '待取件'
					},
					{
						trackingNumber: 'JD5556667778',
						cabinetName: 'C区快递柜',
						cellNumber: '08',
						storageTime: '2024-01-17 16:45:30',
						recipientName: '张三',
						recipientPhone: '138****1234',
						pickupCode: '888888',
						status: 'expired',
						statusText: '已过期'
					}
				],
				safetyTip: '安全提示：请核对快递信息无误后再取件，取件后请及时关闭仓门。如有问题请联系客服400-123-4567。'
			}
		},
		computed: {
			isValidCode() {
				return this.pickupCode.length === 6
			}
		},
		onLoad() {
			this.checkLogin()
		},
		methods: {
			checkLogin() {
				const isLoggedIn = uni.getStorageSync('isLoggedIn')
				if (!isLoggedIn) {
					uni.showModal({
						title: '提示',
						content: '请先登录',
						confirmText: '去登录',
						cancelText: '取消',
						success: (res) => {
							if (res.confirm) {
								uni.navigateTo({
									url: '/pages/login/login'
								})
							} else {
								uni.switchTab({
									url: '/pages/index/index'
								})
							}
						}
					})
				}
			},
			goBack() {
				uni.navigateBack()
			},
			scanQRCode() {
				uni.scanCode({
					success: (res) => {
						console.log('扫描结果:', res.result)
						// 解析二维码内容，获取柜体信息
						uni.showLoading({
							title: '查询中...'
						})

						setTimeout(() => {
							uni.hideLoading()
							// 模拟根据二维码查询到的快递
							uni.showModal({
								title: '扫描成功',
								content: '已找到您的快递，请在下方列表查看',
								showCancel: false
							})
						}, 1000)
					},
					fail: (err) => {
						console.error('扫描失败:', err)
						uni.showToast({
							title: '扫描失败',
							icon: 'none'
						})
					}
				})
			},
			openByCode() {
				if (!this.isValidCode) {
					uni.showToast({
						title: '请输入6位取件码',
						icon: 'none'
					})
					return
				}

				uni.showLoading({
					title: '验证取件码...'
				})

				// 模拟验证取件码
				setTimeout(() => {
					uni.hideLoading()
					const matchedPackage = this.packages.find(pkg => pkg.pickupCode === this.pickupCode)

					if (matchedPackage) {
						if (matchedPackage.status === 'ready') {
							this.openCabinet(matchedPackage)
						} else if (matchedPackage.status === 'picked') {
							uni.showModal({
								title: '提示',
								content: '该快递已被取走',
								showCancel: false
							})
						} else if (matchedPackage.status === 'expired') {
							uni.showModal({
								title: '提示',
								content: '该快递已过期，请联系客服',
								showCancel: false
							})
						}
					} else {
						uni.showModal({
							title: '取件码错误',
							content: '未找到匹配的快递，请检查取件码是否正确',
							showCancel: false
						})
					}
				}, 1500)
			},
			refreshPackages() {
				uni.showLoading({
					title: '刷新中...'
				})

				setTimeout(() => {
					uni.hideLoading()
					uni.showToast({
						title: '刷新成功',
						icon: 'success'
					})
				}, 1000)
			},
			viewPackageDetail(pkg) {
				const statusMap = {
					'ready': '待取件',
					'picked': '已取件',
					'expired': '已过期'
				}

				uni.showModal({
					title: '快递详情',
					content: `
快递单号: ${pkg.trackingNumber}
柜体位置: ${pkg.cabinetName} ${pkg.cellNumber}号仓
存入时间: ${pkg.storageTime}
收件人: ${pkg.recipientName} ${pkg.recipientPhone}
取件码: ${pkg.pickupCode}
状态: ${statusMap[pkg.status] || pkg.status}
					`,
					showCancel: false
				})
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

							// 模拟开柜请求
							setTimeout(() => {
								uni.hideLoading()

								// 更新快递状态
								const index = this.packages.findIndex(item => item.trackingNumber === pkg.trackingNumber)
								if (index !== -1) {
									this.packages[index].status = 'picked'
									this.packages[index].statusText = '已取件'
								}

								uni.showModal({
									title: '开柜成功',
									content: `${pkg.cabinetName} ${pkg.cellNumber}号仓门已打开，请取走快递`,
									showCancel: false,
									success: () => {
										// 可以添加其他逻辑，如返回首页
									}
								})
							}, 2000)
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

	.scan-section {
		padding: 30rpx;
		margin-top: 80rpx;
	}

	.scan-card,
	.input-card {
		background-color: white;
		border-radius: 20rpx;
		padding: 30rpx;
		margin-bottom: 30rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.scan-header,
	.input-header {
		display: flex;
		align-items: center;
		margin-bottom: 30rpx;
	}

	.scan-title,
	.input-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-left: 15rpx;
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

	.scan-btn-text {
		font-size: 32rpx;
		font-weight: bold;
		color: white;
	}

	.or-divider {
		display: flex;
		align-items: center;
		justify-content: center;
		margin: 30rpx 0;
	}

	.divider-line {
		flex: 1;
		height: 1rpx;
		background-color: #e0e0e0;
	}

	.or-text {
		font-size: 24rpx;
		color: #999;
		padding: 0 20rpx;
	}

	.code-input-group {
		display: flex;
		align-items: center;
	}

	.code-input {
		flex: 1;
		height: 90rpx;
		border: 2rpx solid #e0e0e0;
		border-radius: 12rpx;
		padding: 0 30rpx;
		font-size: 32rpx;
		text-align: center;
		letter-spacing: 10rpx;
		margin-right: 20rpx;
	}

	.open-btn {
		width: 200rpx;
		height: 90rpx;
		background: linear-gradient(135deg, #FF9500, #FF5E3A);
		color: white;
		border: none;
		border-radius: 12rpx;
		font-size: 32rpx;
		font-weight: bold;
	}

	.open-btn:disabled {
		opacity: 0.5;
	}

	.packages-section {
		padding: 0 30rpx;
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

	.refresh-btn {
		font-size: 24rpx;
		color: #007AFF;
		display: flex;
		align-items: center;
	}

	.packages-list {
		display: flex;
		flex-direction: column;
		gap: 30rpx;
	}

	.package-card {
		background-color: white;
		border-radius: 20rpx;
		padding: 30rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.package-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 20rpx;
		padding-bottom: 20rpx;
		border-bottom: 1rpx solid #f0f0f0;
	}

	.package-id {
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
	}

	.package-status {
		font-size: 22rpx;
		padding: 6rpx 16rpx;
		border-radius: 20rpx;
	}

	.status-ready {
		background-color: #e7f7ef;
		color: #34C759;
	}

	.status-picked {
		background-color: #e7f0ff;
		color: #007AFF;
	}

	.status-expired {
		background-color: #ffeaea;
		color: #FF3B30;
	}

	.package-info {
		margin-bottom: 30rpx;
	}

	.info-row {
		display: flex;
		align-items: center;
		margin-bottom: 15rpx;
	}

	.info-text {
		font-size: 26rpx;
		color: #666;
		margin-left: 15rpx;
	}

	.package-actions {
		display: flex;
		justify-content: flex-end;
		gap: 20rpx;
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

	.action-btn.disabled {
		background-color: #f0f0f0;
		color: #999;
		border: none;
	}

	.action-btn.expired {
		background-color: #ffeaea;
		color: #FF3B30;
		border: none;
	}

	.empty-state {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 100rpx 30rpx;
	}

	.empty-image {
		width: 200rpx;
		height: 200rpx;
		margin-bottom: 30rpx;
		opacity: 0.6;
	}

	.empty-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #999;
		margin-bottom: 15rpx;
	}

	.empty-desc {
		font-size: 26rpx;
		color: #999;
	}

	.guide-section {
		padding: 30rpx;
		background-color: white;
		margin: 30rpx;
		border-radius: 20rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.guide-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 30rpx;
	}

	.guide-steps {
		display: flex;
		flex-direction: column;
		gap: 25rpx;
	}

	.step {
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
