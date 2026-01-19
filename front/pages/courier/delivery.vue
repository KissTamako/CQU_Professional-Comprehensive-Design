<template>
	<view class="container">
		<!-- 顶部导航栏 -->
		<uni-nav-bar title="快递入柜" left-icon="back" @clickLeft="goBack" backgroundColor="#007AFF" color="#fff" fixed="true"
			statusBar="true">
		</uni-nav-bar>

		<!-- 扫描二维码区域 -->
		<view class="scan-section">
			<view class="scan-card">
				<view class="scan-title">第一步：扫描柜门二维码</view>
				<view class="scan-area" @click="scanQRCode">
					<view class="scan-placeholder">
						<uni-icons type="scan" size="60" color="#007AFF"></uni-icons>
						<text class="scan-text">点击扫描二维码</text>
					</view>
				</view>
				<view class="scan-result" v-if="scannedData">
					<text class="result-label">扫描结果：</text>
					<text class="result-value">{{ scannedData }}</text>
				</view>
			</view>
		</view>

		<!-- 选择仓体 -->
		<view class="cabinet-section" v-if="scannedData">
			<view class="cabinet-card">
				<view class="cabinet-title">第二步：选择仓体尺寸</view>
				<view class="cabinet-grid">
					<view class="cabinet-item" v-for="cabinet in cabinets" :key="cabinet.id" 
						:class="{ selected: selectedCabinet === cabinet.id }" @click="selectCabinet(cabinet.id)">
						<view class="cabinet-icon">
							<uni-icons :type="cabinet.icon" size="30" :color="selectedCabinet === cabinet.id ? '#fff' : '#007AFF'"></uni-icons>
						</view>
						<text class="cabinet-name">{{ cabinet.name }}</text>
						<text class="cabinet-desc">{{ cabinet.desc }}</text>
						<text class="cabinet-status" v-if="cabinet.available">可用 {{ cabinet.available }}个</text>
						<text class="cabinet-status unavailable" v-else>已满</text>
					</view>
				</view>
			</view>
		</view>

		<!-- 快递信息 -->
		<view class="package-section" v-if="selectedCabinet">
			<view class="package-card">
				<view class="package-title">第三步：输入快递信息</view>
				
				<view class="form-group">
					<view class="form-label">快递单号</view>
					<view class="input-with-scan">
						<input class="form-input" v-model="packageInfo.trackingNumber" placeholder="请输入快递单号" />
						<button class="scan-btn" @click="scanBarcode">扫码</button>
					</view>
				</view>

				<view class="form-group">
					<view class="form-label">快递公司</view>
					<uni-data-select v-model="packageInfo.courierCompany" :localdata="courierCompanies" placeholder="请选择快递公司"></uni-data-select>
				</view>

				<view class="form-group">
					<view class="form-label">快递类型</view>
					<view class="type-options">
						<view class="type-item" :class="{ active: packageInfo.packageType === 'normal' }" 
							@click="packageInfo.packageType = 'normal'">
							<text>普通快递</text>
						</view>
						<view class="type-item" :class="{ active: packageInfo.packageType === 'fragile' }" 
							@click="packageInfo.packageType = 'fragile'">
							<text>易碎品</text>
						</view>
						<view class="type-item" :class="{ active: packageInfo.packageType === 'important' }" 
							@click="packageInfo.packageType = 'important'">
							<text>重要文件</text>
						</view>
					</view>
				</view>

				<view class="form-group">
					<view class="form-label">备注信息</view>
					<textarea class="form-textarea" v-model="packageInfo.remarks" placeholder="请输入备注信息（可选）" maxlength="100"></textarea>
				</view>

				<view class="query-section" v-if="packageInfo.trackingNumber && !recipientInfo">
					<button class="query-btn" @click="queryRecipientInfo" :disabled="queryLoading">
						{{ queryLoading ? '查询中...' : '查询收件人信息' }}
					</button>
				</view>

				<!-- 收件人信息 -->
				<view class="recipient-section" v-if="recipientInfo">
					<view class="recipient-title">收件人信息</view>
					<view class="recipient-info">
						<view class="info-row">
							<text class="info-label">姓名：</text>
							<text class="info-value">{{ recipientInfo.name }}</text>
						</view>
						<view class="info-row">
							<text class="info-label">电话：</text>
							<text class="info-value">{{ recipientInfo.phone }}</text>
						</view>
						<view class="info-row">
							<text class="info-label">地址：</text>
							<text class="info-value">{{ recipientInfo.address }}</text>
						</view>
					</view>

					<view class="send-sms-section">
						<button class="sms-btn" @click="sendPickupCode" :disabled="smsSent">
							{{ smsSent ? '短信已发送' : '发送取件码短信' }}
						</button>
						<view class="pickup-code" v-if="pickupCode">
							<text class="code-label">取件码：</text>
							<text class="code-value">{{ pickupCode }}</text>
						</view>
					</view>
				</view>
			</view>
		</view>

		<!-- 确认放入 -->
		<view class="confirm-section" v-if="recipientInfo && smsSent">
			<view class="confirm-card">
				<view class="confirm-title">第四步：确认放入</view>
				<view class="confirm-info">
					<view class="info-item">
						<text class="item-label">柜门位置：</text>
						<text class="item-value">{{ selectedCabinetInfo.name }}</text>
					</view>
					<view class="info-item">
						<text class="item-label">快递单号：</text>
						<text class="item-value">{{ packageInfo.trackingNumber }}</text>
					</view>
					<view class="info-item">
						<text class="item-label">收件人：</text>
						<text class="item-value">{{ recipientInfo.name }}</text>
					</view>
					<view class="info-item">
						<text class="item-label">取件码：</text>
						<text class="item-value">{{ pickupCode }}</text>
					</view>
				</view>
				<button class="confirm-btn" @click="confirmDelivery">确认放入快递</button>
			</view>
		</view>

		<!-- 绑定用户 -->
		<view class="bind-section">
			<view class="bind-card">
				<view class="bind-title">快递员绑定用户</view>
				<view class="bind-form">
					<view class="form-group">
						<view class="form-label">用户手机号</view>
						<input class="form-input" v-model="bindInfo.phone" placeholder="请输入用户手机号" type="number" maxlength="11" />
					</view>
					<view class="form-group">
						<view class="form-label">验证码</view>
						<view class="input-with-verify">
							<input class="form-input" v-model="bindInfo.verifyCode" placeholder="请输入验证码" type="number" maxlength="6" />
							<button class="verify-btn" :disabled="bindCountdown > 0" @click="sendBindVerifyCode">
								{{ bindCountdown > 0 ? `${bindCountdown}s后重发` : '获取验证码' }}
							</button>
						</view>
					</view>
					<button class="bind-btn" @click="bindUser" :disabled="!canBind">绑定用户</button>
				</view>
				<view class="bind-tips">
					<text>绑定用户后，该用户的快递将自动关联到您的账号</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				scannedData: '',
				selectedCabinet: null,
				cabinets: [
					{ id: 1, name: '小号仓', desc: '尺寸: 20×20×20cm', icon: 'cube', available: 5 },
					{ id: 2, name: '中号仓', desc: '尺寸: 30×30×30cm', icon: 'cube', available: 3 },
					{ id: 3, name: '大号仓', desc: '尺寸: 40×40×40cm', icon: 'cube', available: 2 },
					{ id: 4, name: '超大仓', desc: '尺寸: 50×50×50cm', icon: 'cube', available: 1 }
				],
				packageInfo: {
					trackingNumber: '',
					courierCompany: '',
					packageType: 'normal',
					remarks: ''
				},
				courierCompanies: [
					{ value: 'sf', text: '顺丰速运' },
					{ value: 'sto', text: '申通快递' },
					{ value: 'yto', text: '圆通速递' },
					{ value: 'zto', text: '中通快递' },
					{ value: 'yd', text: '韵达快递' },
					{ value: 'jd', text: '京东物流' }
				],
				queryLoading: false,
				recipientInfo: null,
				smsSent: false,
				pickupCode: '',
				bindInfo: {
					phone: '',
					verifyCode: ''
				},
				bindCountdown: 0
			}
		},
		computed: {
			selectedCabinetInfo() {
				return this.cabinets.find(cab => cab.id === this.selectedCabinet) || {}
			},
			canBind() {
				return this.bindInfo.phone.trim() && this.bindInfo.verifyCode.trim()
			}
		},
		methods: {
			goBack() {
				uni.navigateBack()
			},
			scanQRCode() {
				uni.scanCode({
					success: (res) => {
						this.scannedData = res.result
						uni.showToast({
							title: '扫描成功',
							icon: 'success'
						})
					},
					fail: (err) => {
						uni.showToast({
							title: '扫描失败',
							icon: 'none'
						})
						// 模拟数据
						this.scannedData = '快递柜A-12号门'
					}
				})
			},
			scanBarcode() {
				uni.scanCode({
					onlyFromCamera: true,
					scanType: ['barCode'],
					success: (res) => {
						this.packageInfo.trackingNumber = res.result
						uni.showToast({
							title: '扫码成功',
							icon: 'success'
						})
					},
					fail: (err) => {
						uni.showToast({
							title: '扫码失败',
							icon: 'none'
						})
						// 模拟数据
						this.packageInfo.trackingNumber = 'SF1234567890123'
					}
				})
			},
			selectCabinet(id) {
				this.selectedCabinet = id
			},
			queryRecipientInfo() {
				if (!this.packageInfo.trackingNumber) {
					uni.showToast({
						title: '请输入快递单号',
						icon: 'none'
					})
					return
				}

				this.queryLoading = true

				// 模拟查询收件人信息
				setTimeout(() => {
					this.queryLoading = false
					this.recipientInfo = {
						name: '张三',
						phone: '138****1234',
						address: '重庆市沙坪坝区重庆大学A区1号楼'
					}
					uni.showToast({
						title: '查询成功',
						icon: 'success'
					})
				}, 1500)
			},
			sendPickupCode() {
				// 生成6位取件码
				this.pickupCode = Math.random().toString().slice(2, 8)
				
				uni.showLoading({
					title: '发送短信中...'
				})

				// 模拟发送短信
				setTimeout(() => {
					uni.hideLoading()
					this.smsSent = true
					uni.showModal({
						title: '短信发送成功',
						content: `取件码 ${this.pickupCode} 已发送至 ${this.recipientInfo.phone}`,
						showCancel: false
					})
				}, 1000)
			},
			confirmDelivery() {
				uni.showModal({
					title: '确认放入',
					content: '请确认快递已放入选择的仓体中',
					success: (res) => {
						if (res.confirm) {
							uni.showLoading({
								title: '开仓中...'
							})

							setTimeout(() => {
								uni.hideLoading()
								uni.showModal({
									title: '操作成功',
									content: `快递已成功放入${this.selectedCabinetInfo.name}，仓门已关闭`,
									showCancel: false,
									success: () => {
										// 重置数据
										this.resetForm()
									}
								})
							}, 1500)
						}
					}
				})
			},
			sendBindVerifyCode() {
				if (!this.bindInfo.phone || this.bindInfo.phone.length !== 11) {
					uni.showToast({
						title: '请输入正确的手机号码',
						icon: 'none'
					})
					return
				}

				// 开始倒计时
				this.bindCountdown = 60

				const timer = setInterval(() => {
					this.bindCountdown--
					if (this.bindCountdown <= 0) {
						clearInterval(timer)
					}
				}, 1000)

				uni.showToast({
					title: '验证码已发送',
					icon: 'success'
				})
			},
			bindUser() {
				uni.showLoading({
					title: '绑定中...'
				})

				setTimeout(() => {
					uni.hideLoading()
					uni.showModal({
						title: '绑定成功',
						content: `用户 ${this.bindInfo.phone} 已成功绑定到您的账号`,
						showCancel: false
					})

					// 清空绑定信息
					this.bindInfo = {
						phone: '',
						verifyCode: ''
					}
					this.bindCountdown = 0
				}, 1000)
			},
			resetForm() {
				this.scannedData = ''
				this.selectedCabinet = null
				this.packageInfo = {
					trackingNumber: '',
					courierCompany: '',
					packageType: 'normal',
					remarks: ''
				}
				this.recipientInfo = null
				this.smsSent = false
				this.pickupCode = ''
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

	.scan-card {
		background-color: white;
		border-radius: 20rpx;
		padding: 30rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.scan-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 30rpx;
	}

	.scan-area {
		height: 300rpx;
		background-color: #f8f8f8;
		border-radius: 12rpx;
		border: 2rpx dashed #007AFF;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-bottom: 30rpx;
	}

	.scan-placeholder {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.scan-text {
		font-size: 28rpx;
		color: #007AFF;
		margin-top: 20rpx;
	}

	.scan-result {
		background-color: #f0f7ff;
		border-radius: 12rpx;
		padding: 20rpx;
	}

	.result-label {
		font-size: 26rpx;
		color: #666;
	}

	.result-value {
		font-size: 26rpx;
		font-weight: bold;
		color: #333;
		margin-left: 10rpx;
	}

	.cabinet-section {
		padding: 0 30rpx;
	}

	.cabinet-card {
		background-color: white;
		border-radius: 20rpx;
		padding: 30rpx;
		margin-top: 30rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.cabinet-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 30rpx;
	}

	.cabinet-grid {
		display: grid;
		grid-template-columns: repeat(2, 1fr);
		gap: 20rpx;
	}

	.cabinet-item {
		border: 2rpx solid #e0e0e0;
		border-radius: 12rpx;
		padding: 30rpx;
		text-align: center;
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.cabinet-item.selected {
		border-color: #007AFF;
		background-color: #e7f0ff;
	}

	.cabinet-icon {
		width: 60rpx;
		height: 60rpx;
		background-color: #f8f8f8;
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-bottom: 15rpx;
	}

	.cabinet-item.selected .cabinet-icon {
		background-color: #007AFF;
	}

	.cabinet-name {
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 10rpx;
	}

	.cabinet-desc {
		font-size: 22rpx;
		color: #666;
		margin-bottom: 10rpx;
	}

	.cabinet-status {
		font-size: 20rpx;
		color: #34C759;
	}

	.cabinet-status.unavailable {
		color: #FF3B30;
	}

	.package-section {
		padding: 0 30rpx;
		margin-top: 30rpx;
	}

	.package-card {
		background-color: white;
		border-radius: 20rpx;
		padding: 30rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.package-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 30rpx;
	}

	.form-group {
		margin-bottom: 30rpx;
	}

	.form-label {
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 20rpx;
		display: block;
	}

	.form-input {
		height: 80rpx;
		border: 2rpx solid #e0e0e0;
		border-radius: 12rpx;
		padding: 0 30rpx;
		font-size: 28rpx;
		color: #333;
		width: 100%;
		box-sizing: border-box;
	}

	.input-with-scan {
		display: flex;
		align-items: center;
		gap: 20rpx;
	}

	.scan-btn {
		width: 120rpx;
		height: 80rpx;
		background-color: #007AFF;
		border-radius: 12rpx;
		border: none;
		color: white;
		font-size: 24rpx;
		white-space: nowrap;
	}

	.type-options {
		display: flex;
		gap: 20rpx;
	}

	.type-item {
		flex: 1;
		height: 80rpx;
		border: 2rpx solid #e0e0e0;
		border-radius: 12rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 26rpx;
		color: #666;
	}

	.type-item.active {
		border-color: #007AFF;
		background-color: #e7f0ff;
		color: #007AFF;
		font-weight: bold;
	}

	.form-textarea {
		width: 100%;
		height: 150rpx;
		border: 2rpx solid #e0e0e0;
		border-radius: 12rpx;
		padding: 20rpx;
		font-size: 28rpx;
		color: #333;
		box-sizing: border-box;
	}

	.query-section {
		margin-top: 40rpx;
	}

	.query-btn {
		width: 100%;
		height: 90rpx;
		background: linear-gradient(135deg, #FF9500, #FF5E3A);
		border-radius: 12rpx;
		border: none;
		color: white;
		font-size: 32rpx;
		font-weight: bold;
	}

	.query-btn:disabled {
		opacity: 0.5;
	}

	.recipient-section {
		margin-top: 40rpx;
		padding-top: 30rpx;
		border-top: 1rpx solid #f0f0f0;
	}

	.recipient-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 30rpx;
	}

	.recipient-info {
		background-color: #f8f8f8;
		border-radius: 12rpx;
		padding: 30rpx;
		margin-bottom: 30rpx;
	}

	.info-row {
		display: flex;
		margin-bottom: 20rpx;
	}

	.info-row:last-child {
		margin-bottom: 0;
	}

	.info-label {
		font-size: 26rpx;
		color: #666;
		width: 120rpx;
	}

	.info-value {
		font-size: 26rpx;
		font-weight: bold;
		color: #333;
		flex: 1;
	}

	.send-sms-section {
		text-align: center;
	}

	.sms-btn {
		width: 100%;
		height: 90rpx;
		background: linear-gradient(135deg, #34C759, #2DBF5C);
		border-radius: 12rpx;
		border: none;
		color: white;
		font-size: 32rpx;
		font-weight: bold;
		margin-bottom: 30rpx;
	}

	.sms-btn:disabled {
		opacity: 0.5;
	}

	.pickup-code {
		background-color: #fff3e0;
		border-radius: 12rpx;
		padding: 20rpx;
		text-align: center;
	}

	.code-label {
		font-size: 26rpx;
		color: #FF9500;
	}

	.code-value {
		font-size: 36rpx;
		font-weight: bold;
		color: #FF9500;
		margin-left: 10rpx;
		letter-spacing: 5rpx;
	}

	.confirm-section {
		padding: 0 30rpx;
		margin-top: 30rpx;
	}

	.confirm-card {
		background-color: white;
		border-radius: 20rpx;
		padding: 30rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.confirm-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 30rpx;
	}

	.confirm-info {
		background-color: #f8f8f8;
		border-radius: 12rpx;
		padding: 30rpx;
		margin-bottom: 30rpx;
	}

	.info-item {
		display: flex;
		margin-bottom: 20rpx;
	}

	.info-item:last-child {
		margin-bottom: 0;
	}

	.item-label {
		font-size: 26rpx;
		color: #666;
		width: 140rpx;
	}

	.item-value {
		font-size: 26rpx;
		font-weight: bold;
		color: #333;
		flex: 1;
	}

	.confirm-btn {
		width: 100%;
		height: 90rpx;
		background: linear-gradient(135deg, #007AFF, #5856D6);
		border-radius: 12rpx;
		border: none;
		color: white;
		font-size: 32rpx;
		font-weight: bold;
	}

	.bind-section {
		padding: 30rpx;
		margin-top: 30rpx;
	}

	.bind-card {
		background-color: white;
		border-radius: 20rpx;
		padding: 30rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.bind-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 30rpx;
		text-align: center;
	}

	.bind-form {
		margin-bottom: 30rpx;
	}

	.input-with-verify {
		display: flex;
		align-items: center;
		gap: 20rpx;
	}

	.verify-btn {
		width: 200rpx;
		height: 80rpx;
		background: linear-gradient(135deg, #FF9500, #FF5E3A);
		border-radius: 12rpx;
		border: none;
		color: white;
		font-size: 24rpx;
		white-space: nowrap;
	}

	.verify-btn:disabled {
		opacity: 0.5;
	}

	.bind-btn {
		width: 100%;
		height: 90rpx;
		background: linear-gradient(135deg, #5856D6, #AF52DE);
		border-radius: 12rpx;
		border: none;
		color: white;
		font-size: 32rpx;
		font-weight: bold;
		margin-top: 20rpx;
	}

	.bind-btn:disabled {
		opacity: 0.5;
	}

	.bind-tips {
		text-align: center;
		font-size: 24rpx;
		color: #999;
		padding-top: 20rpx;
		border-top: 1rpx solid #f0f0f0;
	}
</style>
