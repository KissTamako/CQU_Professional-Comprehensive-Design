<template>
	<view class="container">
		<!-- 顶部导航栏 -->
		<uni-nav-bar title="寄存物品" left-icon="back" @clickLeft="goBack" backgroundColor="#007AFF" color="#fff" fixed="true"
			statusBar="true">
		</uni-nav-bar>

		<!-- 扫描区域 -->
		<view class="scan-section">
			<view class="scan-card">
				<view class="scan-header">
					<uni-icons type="scan" size="30" color="#007AFF"></uni-icons>
					<text class="scan-title">扫描柜门二维码开始寄存</text>
				</view>
				<view class="scan-desc">
					<text>扫描快递柜上的二维码，选择可用仓门存放物品</text>
				</view>
				<button class="scan-btn" @click="scanQRCode">
					<text class="scan-btn-text">开始扫描</text>
				</button>
			</view>
		</view>

		<!-- 寄存表单 -->
		<view class="form-section" v-if="showForm">
			<view class="form-title">寄存信息</view>

			<view class="form-group">
				<view class="form-label">物品名称</view>
				<input class="form-input" v-model="storageInfo.itemName" placeholder="请输入物品名称" />
			</view>

			<view class="form-group">
				<view class="form-label">物品描述</view>
				<textarea class="form-textarea" v-model="storageInfo.itemDesc" placeholder="请输入物品描述（可选）" maxlength="100" />
			</view>

			<view class="form-group">
				<view class="form-label">收件人信息</view>
				<view class="recipient-options">
					<view class="option-item" :class="{ active: storageInfo.recipientType === 'self' }"
						@click="storageInfo.recipientType = 'self'">
						<text>本人取件</text>
					</view>
					<view class="option-item" :class="{ active: storageInfo.recipientType === 'other' }"
						@click="storageInfo.recipientType = 'other'">
						<text>他人取件</text>
					</view>
				</view>

				<view class="recipient-form" v-if="storageInfo.recipientType === 'other'">
					<input class="form-input" v-model="storageInfo.recipientName" placeholder="收件人姓名" />
					<input class="form-input" v-model="storageInfo.recipientPhone" placeholder="收件人手机号" type="number" />
				</view>
			</view>

			<view class="form-group">
				<view class="form-label">选择仓体</view>
				<view class="cabinet-grid">
					<view class="cabinet-item" v-for="cell in availableCells" :key="cell.id"
						:class="{ selected: storageInfo.selectedCell === cell.id, disabled: !cell.available }"
						@click="selectCell(cell)">
						<text class="cell-number">{{ cell.number }}</text>
						<text class="cell-size">{{ cell.size }}</text>
						<text class="cell-status" v-if="!cell.available">占用</text>
					</view>
				</view>
			</view>

			<view class="form-group">
				<view class="form-label">寄存时长</view>
				<view class="duration-options">
					<view class="duration-item" :class="{ active: storageInfo.duration === 24 }"
						@click="storageInfo.duration = 24">
						<text>24小时</text>
						<text class="duration-price">免费</text>
					</view>
					<view class="duration-item" :class="{ active: storageInfo.duration === 48 }"
						@click="storageInfo.duration = 48">
						<text>48小时</text>
						<text class="duration-price">¥5</text>
					</view>
					<view class="duration-item" :class="{ active: storageInfo.duration === 72 }"
						@click="storageInfo.duration = 72">
						<text>72小时</text>
						<text class="duration-price">¥10</text>
					</view>
				</view>
			</view>

			<view class="form-actions">
				<button class="submit-btn" :disabled="!canSubmit" @click="submitStorage">确认寄存</button>
			</view>
		</view>

		<!-- 寄存成功 -->
		<view class="success-section" v-if="showSuccess">
			<view class="success-card">
				<uni-icons type="checkmarkempty" size="60" color="#34C759"></uni-icons>
				<text class="success-title">寄存成功！</text>
				<text class="success-desc">您的物品已存入 {{ selectedCellInfo.cabinetName }} {{ selectedCellInfo.cellNumber }}号仓</text>

				<view class="pickup-code">
					<text class="code-label">取件码</text>
					<text class="code-value">{{ pickupCode }}</text>
				</view>

				<view class="success-info">
					<view class="info-item">
						<text class="info-label">物品名称</text>
						<text class="info-value">{{ storageInfo.itemName }}</text>
					</view>
					<view class="info-item">
						<text class="info-label">寄存时间</text>
						<text class="info-value">{{ storageTime }}</text>
					</view>
					<view class="info-item">
						<text class="info-label">过期时间</text>
						<text class="info-value">{{ expiryTime }}</text>
					</view>
				</view>

				<view class="share-section">
					<text class="share-tip">您可以将取件码分享给他人取件</text>
					<button class="share-btn" @click="sharePickupCode">
						<uni-icons type="upload" size="20" color="#fff"></uni-icons>
						<text>分享取件码</text>
					</button>
				</view>

				<view class="success-actions">
					<button class="action-btn" @click="goBack">返回首页</button>
					<button class="action-btn primary" @click="createNew">继续寄存</button>
				</view>
			</view>
		</view>

		<!-- 寄存记录 -->
		<view class="history-section" v-if="storageHistory.length > 0">
			<view class="section-title">寄存记录</view>
			<view class="history-list">
				<view class="history-item" v-for="(item, index) in storageHistory.slice(0, 3)" :key="index">
					<view class="item-header">
						<text class="item-name">{{ item.itemName }}</text>
						<text class="item-status" :class="'status-' + item.status">{{ item.statusText }}</text>
					</view>
					<view class="item-info">
						<text class="info-text">取件码: {{ item.pickupCode }}</text>
						<text class="info-text">仓门: {{ item.cellLocation }}</text>
						<text class="info-text">时间: {{ item.storageTime }}</text>
					</view>
					<view class="item-actions">
						<button class="small-btn" size="mini" @click="viewDetail(item)">详情</button>
						<button class="small-btn" size="mini" @click="shareItem(item)" v-if="item.status === 'stored'">分享</button>
					</view>
				</view>
			</view>
		</view>

		<!-- 注意事项 -->
		<view class="notice-section">
			<view class="notice-title">寄存注意事项</view>
			<view class="notice-list">
				<view class="notice-item">
					<uni-icons type="info" size="20" color="#FF9500"></uni-icons>
					<text class="notice-text">禁止寄存易燃易爆、危险品、违禁品</text>
				</view>
				<view class="notice-item">
					<uni-icons type="info" size="20" color="#FF9500"></uni-icons>
					<text class="notice-text">贵重物品请随身携带，遗失概不负责</text>
				</view>
				<view class="notice-item">
					<uni-icons type="info" size="20" color="#FF9500"></uni-icons>
					<text class="notice-text">超时未取将收取额外费用</text>
				</view>
				<view class="notice-item">
					<uni-icons type="info" size="20" color="#FF9500"></uni-icons>
					<text class="notice-text">取件码请妥善保管，勿泄露给他人</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				showForm: false,
				showSuccess: false,
				storageInfo: {
					itemName: '',
					itemDesc: '',
					recipientType: 'self',
					recipientName: '',
					recipientPhone: '',
					selectedCell: null,
					duration: 24
				},
				availableCells: [{
						id: 1,
						number: '01',
						size: '小',
						available: true,
						cabinetName: 'A区快递柜'
					},
					{
						id: 2,
						number: '02',
						size: '小',
						available: true,
						cabinetName: 'A区快递柜'
					},
					{
						id: 3,
						number: '03',
						size: '中',
						available: false,
						cabinetName: 'A区快递柜'
					},
					{
						id: 4,
						number: '04',
						size: '中',
						available: true,
						cabinetName: 'A区快递柜'
					},
					{
						id: 5,
						number: '05',
						size: '大',
						available: true,
						cabinetName: 'A区快递柜'
					},
					{
						id: 6,
						number: '06',
						size: '大',
						available: true,
						cabinetName: 'A区快递柜'
					}
				],
				pickupCode: '',
				storageTime: '',
				expiryTime: '',
				selectedCellInfo: {},
				storageHistory: [{
						itemName: '文件袋',
						pickupCode: '123456',
						cellLocation: 'A区-01',
						storageTime: '2024-01-18 10:30',
						status: 'stored',
						statusText: '寄存中'
					},
					{
						itemName: '书籍',
						pickupCode: '654321',
						cellLocation: 'B区-05',
						storageTime: '2024-01-17 14:20',
						status: 'picked',
						statusText: '已取件'
					},
					{
						itemName: '雨伞',
						pickupCode: '888888',
						cellLocation: 'C区-08',
						storageTime: '2024-01-16 09:15',
						status: 'expired',
						statusText: '已过期'
					}
				]
			}
		},
		computed: {
			canSubmit() {
				return this.storageInfo.itemName.trim() &&
					this.storageInfo.selectedCell &&
					(this.storageInfo.recipientType === 'self' ||
						(this.storageInfo.recipientName.trim() && this.storageInfo.recipientPhone.trim()))
			}
		},
		methods: {
			goBack() {
				uni.navigateBack()
			},
			scanQRCode() {
				uni.scanCode({
					success: (res) => {
						console.log('扫描结果:', res.result)
						uni.showToast({
							title: '扫描成功',
							icon: 'success'
						})
						this.showForm = true
					},
					fail: (err) => {
						console.error('扫描失败:', err)
						uni.showToast({
							title: '扫描失败',
							icon: 'none'
						})
						// 测试用：直接显示表单
						this.showForm = true
					}
				})
			},
			selectCell(cell) {
				if (!cell.available) return
				this.storageInfo.selectedCell = cell.id
				this.selectedCellInfo = cell
			},
			generatePickupCode() {
				// 生成6位随机数字
				return Math.floor(100000 + Math.random() * 900000).toString()
			},
			formatTime(date) {
				const year = date.getFullYear()
				const month = String(date.getMonth() + 1).padStart(2, '0')
				const day = String(date.getDate()).padStart(2, '0')
				const hour = String(date.getHours()).padStart(2, '0')
				const minute = String(date.getMinutes()).padStart(2, '0')
				return `${year}-${month}-${day} ${hour}:${minute}`
			},
			submitStorage() {
				if (!this.canSubmit) return

				uni.showLoading({
					title: '处理中...'
				})

				// 模拟API请求
				setTimeout(() => {
					uni.hideLoading()

					// 生成取件码
					this.pickupCode = this.generatePickupCode()

					// 设置时间
					const now = new Date()
					this.storageTime = this.formatTime(now)

					const expiry = new Date(now.getTime() + this.storageInfo.duration * 60 * 60 * 1000)
					this.expiryTime = this.formatTime(expiry)

					// 添加到历史记录
					this.storageHistory.unshift({
						itemName: this.storageInfo.itemName,
						pickupCode: this.pickupCode,
						cellLocation: `${this.selectedCellInfo.cabinetName} ${this.selectedCellInfo.number}`,
						storageTime: this.storageTime,
						status: 'stored',
						statusText: '寄存中'
					})

					// 更新仓门状态
					const cellIndex = this.availableCells.findIndex(cell => cell.id === this.storageInfo.selectedCell)
					if (cellIndex !== -1) {
						this.availableCells[cellIndex].available = false
					}

					this.showSuccess = true
					this.showForm = false
				}, 1500)
			},
			sharePickupCode() {
				uni.showModal({
					title: '分享取件码',
					content: `取件码：${this.pickupCode}\n\n请将取件码发送给取件人`,
					confirmText: '复制取件码',
					cancelText: '取消',
					success: (res) => {
						if (res.confirm) {
							uni.setClipboardData({
								data: this.pickupCode,
								success: () => {
									uni.showToast({
										title: '已复制到剪贴板',
										icon: 'success'
									})
								}
							})
						}
					}
				})
			},
			goHome() {
				uni.switchTab({
					url: '/pages/index/index'
				})
			},
			createNew() {
				// 重置表单
				this.storageInfo = {
					itemName: '',
					itemDesc: '',
					recipientType: 'self',
					recipientName: '',
					recipientPhone: '',
					selectedCell: null,
					duration: 24
				}
				this.pickupCode = ''
				this.showSuccess = false
				this.showForm = true
			},
			viewDetail(item) {
				uni.showModal({
					title: '寄存详情',
					content: `
物品名称: ${item.itemName}
取件码: ${item.pickupCode}
仓门位置: ${item.cellLocation}
寄存时间: ${item.storageTime}
状态: ${item.statusText}
					`,
					showCancel: false
				})
			},
			shareItem(item) {
				uni.showModal({
					title: '分享取件码',
					content: `取件码：${item.pickupCode}\n\n请将取件码发送给取件人`,
					confirmText: '复制取件码',
					success: (res) => {
						if (res.confirm) {
							uni.setClipboardData({
								data: item.pickupCode,
								success: () => {
									uni.showToast({
										title: '已复制到剪贴板',
										icon: 'success'
									})
								}
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

	.scan-section {
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

	.scan-header {
		display: flex;
		align-items: center;
		justify-content: center;
		margin-bottom: 20rpx;
	}

	.scan-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-left: 15rpx;
	}

	.scan-desc {
		font-size: 26rpx;
		color: #666;
		margin-bottom: 40rpx;
		line-height: 1.5;
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

	.form-section {
		padding: 30rpx;
	}

	.form-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 30rpx;
	}

	.form-group {
		background-color: white;
		border-radius: 20rpx;
		padding: 30rpx;
		margin-bottom: 30rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.form-label {
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 20rpx;
	}

	.form-input {
		height: 80rpx;
		border: 2rpx solid #e0e0e0;
		border-radius: 12rpx;
		padding: 0 30rpx;
		font-size: 28rpx;
		color: #333;
	}

	.form-textarea {
		height: 150rpx;
		border: 2rpx solid #e0e0e0;
		border-radius: 12rpx;
		padding: 20rpx 30rpx;
		font-size: 28rpx;
		color: #333;
	}

	.recipient-options {
		display: flex;
		gap: 20rpx;
		margin-bottom: 20rpx;
	}

	.option-item {
		flex: 1;
		height: 80rpx;
		border: 2rpx solid #e0e0e0;
		border-radius: 12rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 28rpx;
		color: #666;
	}

	.option-item.active {
		border-color: #007AFF;
		background-color: #e7f0ff;
		color: #007AFF;
	}

	.recipient-form {
		display: flex;
		flex-direction: column;
		gap: 20rpx;
	}

	.cabinet-grid {
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		gap: 20rpx;
	}

	.cabinet-item {
		aspect-ratio: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		border: 2rpx solid #e0e0e0;
		border-radius: 12rpx;
		background-color: white;
	}

	.cabinet-item.selected {
		border-color: #007AFF;
		background-color: #e7f0ff;
	}

	.cabinet-item.disabled {
		background-color: #f8f8f8;
		border-color: #ddd;
		color: #999;
	}

	.cell-number {
		font-size: 36rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 10rpx;
	}

	.cabinet-item.disabled .cell-number {
		color: #999;
	}

	.cell-size {
		font-size: 24rpx;
		color: #666;
	}

	.cell-status {
		font-size: 20rpx;
		color: #FF3B30;
		margin-top: 10rpx;
	}

	.duration-options {
		display: flex;
		flex-direction: column;
		gap: 20rpx;
	}

	.duration-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		height: 80rpx;
		border: 2rpx solid #e0e0e0;
		border-radius: 12rpx;
		padding: 0 30rpx;
		font-size: 28rpx;
		color: #666;
	}

	.duration-item.active {
		border-color: #007AFF;
		background-color: #e7f0ff;
		color: #007AFF;
	}

	.duration-price {
		font-size: 24rpx;
		color: #FF9500;
	}

	.duration-item.active .duration-price {
		color: #FF9500;
		font-weight: bold;
	}

	.form-actions {
		padding: 0 30rpx;
	}

	.submit-btn {
		width: 100%;
		height: 90rpx;
		background: linear-gradient(135deg, #34C759, #2ECC71);
		border-radius: 12rpx;
		border: none;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 32rpx;
		font-weight: bold;
		color: white;
	}

	.submit-btn:disabled {
		opacity: 0.5;
	}

	.success-section {
		padding: 30rpx;
	}

	.success-card {
		background-color: white;
		border-radius: 20rpx;
		padding: 50rpx 30rpx;
		text-align: center;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.success-title {
		display: block;
		font-size: 36rpx;
		font-weight: bold;
		color: #333;
		margin: 20rpx 0 10rpx;
	}

	.success-desc {
		display: block;
		font-size: 26rpx;
		color: #666;
		margin-bottom: 40rpx;
		line-height: 1.5;
	}

	.pickup-code {
		background-color: #f8f8f8;
		border-radius: 12rpx;
		padding: 30rpx;
		margin-bottom: 40rpx;
	}

	.code-label {
		display: block;
		font-size: 24rpx;
		color: #666;
		margin-bottom: 10rpx;
	}

	.code-value {
		display: block;
		font-size: 48rpx;
		font-weight: bold;
		color: #007AFF;
		letter-spacing: 10rpx;
	}

	.success-info {
		text-align: left;
		margin-bottom: 40rpx;
	}

	.info-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 20rpx 0;
		border-bottom: 1rpx solid #f0f0f0;
	}

	.info-item:last-child {
		border-bottom: none;
	}

	.info-label {
		font-size: 26rpx;
		color: #666;
	}

	.info-value {
		font-size: 26rpx;
		font-weight: bold;
		color: #333;
	}

	.share-section {
		margin-bottom: 40rpx;
	}

	.share-tip {
		display: block;
		font-size: 24rpx;
		color: #666;
		margin-bottom: 20rpx;
	}

	.share-btn {
		width: 100%;
		height: 80rpx;
		background: linear-gradient(135deg, #FF9500, #FF5E3A);
		border-radius: 12rpx;
		border: none;
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 10rpx;
		font-size: 28rpx;
		color: white;
	}

	.success-actions {
		display: flex;
		gap: 20rpx;
	}

	.action-btn {
		flex: 1;
		height: 80rpx;
		border: 2rpx solid #007AFF;
		border-radius: 12rpx;
		background-color: white;
		color: #007AFF;
		font-size: 28rpx;
	}

	.action-btn.primary {
		background-color: #007AFF;
		color: white;
		border: none;
	}

	.history-section {
		padding: 30rpx;
	}

	.section-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 30rpx;
	}

	.history-list {
		display: flex;
		flex-direction: column;
		gap: 20rpx;
	}

	.history-item {
		background-color: white;
		border-radius: 20rpx;
		padding: 30rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.item-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 20rpx;
	}

	.item-name {
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
	}

	.item-status {
		font-size: 22rpx;
		padding: 6rpx 16rpx;
		border-radius: 20rpx;
	}

	.status-stored {
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

	.item-info {
		display: flex;
		flex-direction: column;
		gap: 10rpx;
		margin-bottom: 20rpx;
	}

	.info-text {
		font-size: 24rpx;
		color: #666;
	}

	.item-actions {
		display: flex;
		justify-content: flex-end;
		gap: 20rpx;
	}

	.small-btn {
		padding: 8rpx 24rpx;
		font-size: 24rpx;
		border-radius: 20rpx;
		border: 1rpx solid #ddd;
		background-color: white;
		color: #666;
	}

	.notice-section {
		padding: 30rpx;
	}

	.notice-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 30rpx;
	}

	.notice-list {
		display: flex;
		flex-direction: column;
		gap: 20rpx;
	}

	.notice-item {
		display: flex;
		align-items: flex-start;
		gap: 15rpx;
	}

	.notice-text {
		flex: 1;
		font-size: 26rpx;
		color: #666;
		line-height: 1.5;
	}
</style>
