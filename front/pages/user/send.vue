<template>
	<view class="container">
		<!-- 顶部导航栏 -->
		<uni-nav-bar title="发快递" left-icon="back" @clickLeft="goBack" backgroundColor="#007AFF" color="#fff" fixed="true"
			statusBar="true">
		</uni-nav-bar>

		<!-- 步骤指示器 -->
		<view class="steps">
			<view class="step-item" :class="{ active: currentStep >= 1 }">
				<view class="step-number">1</view>
				<text class="step-text">填写信息</text>
			</view>
			<view class="step-line"></view>
			<view class="step-item" :class="{ active: currentStep >= 2 }">
				<view class="step-number">2</view>
				<text class="step-text">选择仓体</text>
			</view>
			<view class="step-line"></view>
			<view class="step-item" :class="{ active: currentStep >= 3 }">
				<view class="step-number">3</view>
				<text class="step-text">确认支付</text>
			</view>
		</view>

		<!-- 步骤1: 填写信息 -->
		<view class="step-section" v-if="currentStep === 1">
			<view class="section-title">填写快递信息</view>

			<!-- 发件人信息 -->
			<view class="info-card">
				<view class="card-header">
					<uni-icons type="person" size="24" color="#007AFF"></uni-icons>
					<text class="card-title">发件人信息</text>
					<text class="auto-fill" @click="autoFillSender">自动填充</text>
				</view>
				<view class="form-group">
					<view class="form-label">姓名</view>
					<input class="form-input" v-model="senderInfo.name" placeholder="请输入发件人姓名" />
				</view>
				<view class="form-group">
					<view class="form-label">电话</view>
					<input class="form-input" v-model="senderInfo.phone" placeholder="请输入发件人电话" type="number" />
				</view>
				<view class="form-group">
					<view class="form-label">地址</view>
					<textarea class="form-textarea" v-model="senderInfo.address" placeholder="请输入详细地址" />
				</view>
			</view>

			<!-- 收件人信息 -->
			<view class="info-card">
				<view class="card-header">
					<uni-icons type="person" size="24" color="#FF9500"></uni-icons>
					<text class="card-title">收件人信息</text>
				</view>
				<view class="form-group">
					<view class="form-label">姓名</view>
					<input class="form-input" v-model="receiverInfo.name" placeholder="请输入收件人姓名" />
				</view>
				<view class="form-group">
					<view class="form-label">电话</view>
					<input class="form-input" v-model="receiverInfo.phone" placeholder="请输入收件人电话" type="number" />
				</view>
				<view class="form-group">
					<view class="form-label">地址</view>
					<textarea class="form-textarea" v-model="receiverInfo.address" placeholder="请输入详细地址" />
				</view>
			</view>

			<!-- 快递信息 -->
			<view class="info-card">
				<view class="card-header">
					<uni-icons type="cube" size="24" color="#34C759"></uni-icons>
					<text class="card-title">快递信息</text>
				</view>
				<view class="form-group">
					<view class="form-label">物品类型</view>
					<view class="type-options">
						<view class="type-item" :class="{ active: packageInfo.type === 'document' }"
							@click="packageInfo.type = 'document'">
							<text>文件</text>
						</view>
						<view class="type-item" :class="{ active: packageInfo.type === 'clothes' }"
							@click="packageInfo.type = 'clothes'">
							<text>衣物</text>
						</view>
						<view class="type-item" :class="{ active: packageInfo.type === 'electronics' }"
							@click="packageInfo.type = 'electronics'">
							<text>电子产品</text>
						</view>
						<view class="type-item" :class="{ active: packageInfo.type === 'other' }"
							@click="packageInfo.type = 'other'">
							<text>其他</text>
						</view>
					</view>
				</view>
				<view class="form-group">
					<view class="form-label">物品描述</view>
					<textarea class="form-textarea" v-model="packageInfo.description" placeholder="请描述物品内容" />
				</view>
				<view class="form-group">
					<view class="form-label">预估重量</view>
					<view class="weight-selector">
						<view class="weight-item" :class="{ active: packageInfo.weight === 1 }"
							@click="packageInfo.weight = 1">
							<text>1kg以内</text>
						</view>
						<view class="weight-item" :class="{ active: packageInfo.weight === 2 }"
							@click="packageInfo.weight = 2">
							<text>1-3kg</text>
						</view>
						<view class="weight-item" :class="{ active: packageInfo.weight === 3 }"
							@click="packageInfo.weight = 3">
							<text>3-5kg</text>
						</view>
						<view class="weight-item" :class="{ active: packageInfo.weight === 5 }"
							@click="packageInfo.weight = 5">
							<text>5kg以上</text>
						</view>
					</view>
				</view>
			</view>

			<view class="step-actions">
				<button class="next-btn" :disabled="!canProceedToStep2" @click="goToStep2">下一步</button>
			</view>
		</view>

		<!-- 步骤2: 选择仓体 -->
		<view class="step-section" v-if="currentStep === 2">
			<view class="section-title">选择可用仓体</view>

			<view class="cabinet-info">
				<text class="cabinet-name">当前快递柜: {{ currentCabinet.name }}</text>
				<text class="cabinet-location">位置: {{ currentCabinet.location }}</text>
			</view>

			<view class="cabinet-grid">
				<view class="cabinet-item" v-for="cell in availableCells" :key="cell.id"
					:class="{ selected: selectedCellId === cell.id, disabled: !cell.available, recommended: cell.recommended }"
					@click="selectCell(cell)">
					<text class="cell-number">{{ cell.number }}</text>
					<text class="cell-size">{{ cell.size }}</text>
					<text class="cell-status" v-if="!cell.available">占用</text>
					<text class="cell-recommended" v-if="cell.recommended">推荐</text>
				</view>
			</view>

			<view class="size-guide">
				<view class="guide-title">仓体尺寸参考</view>
				<view class="guide-items">
					<view class="guide-item">
						<view class="size-icon small"></view>
						<text class="size-text">小仓: 文件/小件</text>
					</view>
					<view class="guide-item">
						<view class="size-icon medium"></view>
						<text class="size-text">中仓: 鞋盒/书籍</text>
					</view>
					<view class="guide-item">
						<view class="size-icon large"></view>
						<text class="size-text">大仓: 背包/衣物</text>
					</view>
				</view>
			</view>

			<view class="step-actions">
				<button class="prev-btn" @click="goToStep1">上一步</button>
				<button class="next-btn" :disabled="!selectedCellId" @click="goToStep3">下一步</button>
			</view>
		</view>

		<!-- 步骤3: 确认支付 -->
		<view class="step-section" v-if="currentStep === 3">
			<view class="section-title">确认订单信息</view>

			<view class="order-summary">
				<view class="summary-item">
					<text class="item-label">快递单号</text>
					<text class="item-value">{{ orderInfo.trackingNumber }}</text>
				</view>
				<view class="summary-item">
					<text class="item-label">发件人</text>
					<text class="item-value">{{ senderInfo.name }} {{ senderInfo.phone }}</text>
				</view>
				<view class="summary-item">
					<text class="item-label">收件人</text>
					<text class="item-value">{{ receiverInfo.name }} {{ receiverInfo.phone }}</text>
				</view>
				<view class="summary-item">
					<text class="item-label">仓体位置</text>
					<text class="item-value">{{ selectedCellInfo.cabinetName }} {{ selectedCellInfo.cellNumber }}号仓</text>
				</view>
				<view class="summary-item">
					<text class="item-label">物品类型</text>
					<text class="item-value">{{ packageTypeText }}</text>
				</view>
				<view class="summary-item">
					<text class="item-label">预估重量</text>
					<text class="item-value">{{ weightText }}</text>
				</view>
			</view>

			<view class="price-breakdown">
				<view class="price-item">
					<text class="price-label">快递费</text>
					<text class="price-value">¥{{ orderInfo.shippingFee }}</text>
				</view>
				<view class="price-item">
					<text class="price-label">柜体使用费</text>
					<text class="price-value">¥{{ orderInfo.cabinetFee }}</text>
				</view>
				<view class="price-item total">
					<text class="price-label">总计</text>
					<text class="price-value">¥{{ orderInfo.totalFee }}</text>
				</view>
			</view>

			<view class="payment-methods">
				<view class="payment-title">选择支付方式</view>
				<view class="method-list">
					<view class="method-item" :class="{ selected: paymentMethod === 'wechat' }"
						@click="paymentMethod = 'wechat'">
						<uni-icons type="weixin" size="30" color="#07C160"></uni-icons>
						<text class="method-name">微信支付</text>
						<uni-icons type="checkmark" size="20" color="#007AFF" v-if="paymentMethod === 'wechat'"></uni-icons>
					</view>
					<view class="method-item" :class="{ selected: paymentMethod === 'alipay' }"
						@click="paymentMethod = 'alipay'">
						<uni-icons type="wallet" size="30" color="#007AFF"></uni-icons>
						<text class="method-name">支付宝</text>
						<uni-icons type="checkmark" size="20" color="#007AFF" v-if="paymentMethod === 'alipay'"></uni-icons>
					</view>
					<view class="method-item" :class="{ selected: paymentMethod === 'balance' }"
						@click="paymentMethod = 'balance'">
						<uni-icons type="money" size="30" color="#FF9500"></uni-icons>
						<text class="method-name">账户余额</text>
						<uni-icons type="checkmark" size="20" color="#007AFF" v-if="paymentMethod === 'balance'"></uni-icons>
					</view>
				</view>
			</view>

			<view class="agreement">
				<view class="agreement-check" @click="agreementChecked = !agreementChecked">
					<uni-icons :type="agreementChecked ? 'checkbox-filled' : 'circle'" size="20"
						:color="agreementChecked ? '#007AFF' : '#999'"></uni-icons>
					<text class="agreement-text">我已阅读并同意《快递服务协议》</text>
				</view>
			</view>

			<view class="step-actions">
				<button class="prev-btn" @click="goToStep2">上一步</button>
				<button class="pay-btn" :disabled="!canSubmitOrder" @click="submitOrder">确认支付</button>
			</view>
		</view>

		<!-- 成功页面 -->
		<view class="success-section" v-if="currentStep === 4">
			<view class="success-card">
				<uni-icons type="checkmarkempty" size="60" color="#34C759"></uni-icons>
				<text class="success-title">下单成功！</text>
				<text class="success-desc">快递员将在2小时内取件</text>

				<view class="order-details">
					<view class="detail-item">
						<text class="detail-label">快递单号</text>
						<text class="detail-value">{{ orderInfo.trackingNumber }}</text>
					</view>
					<view class="detail-item">
						<text class="detail-label">仓门编号</text>
						<text class="detail-value">{{ selectedCellInfo.cabinetName }} {{ selectedCellInfo.cellNumber }}号仓</text>
					</view>
					<view class="detail-item">
						<text class="detail-label">取件码</text>
						<text class="detail-value">{{ orderInfo.pickupCode }}</text>
					</view>
					<view class="detail-item">
						<text class="detail-label">支付金额</text>
						<text class="detail-value">¥{{ orderInfo.totalFee }}</text>
					</view>
				</view>

				<view class="instructions">
					<text class="instructions-title">操作指引</text>
					<text class="instructions-text">1. 请将物品放入 {{ selectedCellInfo.cabinetName }} {{ selectedCellInfo.cellNumber }}号仓</text>
					<text class="instructions-text">2. 关闭仓门，系统将自动通知快递员</text>
					<text class="instructions-text">3. 快递员将在2小时内上门取件</text>
				</view>

				<view class="success-actions">
					<button class="action-btn" @click="goHome">返回首页</button>
					<button class="action-btn primary" @click="viewOrderDetail">查看订单</button>
				</view>
			</view>
		</view>

		<!-- 底部提示 -->
		<view class="bottom-tips">
			<text class="tip-text">客服电话: 400-123-4567 (08:00-22:00)</text>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				currentStep: 1,
				senderInfo: {
					name: '',
					phone: '',
					address: ''
				},
				receiverInfo: {
					name: '',
					phone: '',
					address: ''
				},
				packageInfo: {
					type: 'document',
					description: '',
					weight: 1
				},
				currentCabinet: {
					name: 'A区快递柜',
					location: '1号楼大厅'
				},
				availableCells: [{
						id: 1,
						number: '01',
						size: '小',
						available: true,
						recommended: true,
						cabinetName: 'A区快递柜',
						cellNumber: '01'
					},
					{
						id: 2,
						number: '02',
						size: '小',
						available: true,
						recommended: false,
						cabinetName: 'A区快递柜',
						cellNumber: '02'
					},
					{
						id: 3,
						number: '03',
						size: '中',
						available: false,
						recommended: false,
						cabinetName: 'A区快递柜',
						cellNumber: '03'
					},
					{
						id: 4,
						number: '04',
						size: '中',
						available: true,
						recommended: true,
						cabinetName: 'A区快递柜',
						cellNumber: '04'
					},
					{
						id: 5,
						number: '05',
						size: '大',
						available: true,
						recommended: false,
						cabinetName: 'A区快递柜',
						cellNumber: '05'
					},
					{
						id: 6,
						number: '06',
						size: '大',
						available: true,
						recommended: true,
						cabinetName: 'A区快递柜',
						cellNumber: '06'
					}
				],
				selectedCellId: null,
				selectedCellInfo: {},
				orderInfo: {
					trackingNumber: '',
					shippingFee: 12,
					cabinetFee: 0,
					totalFee: 12,
					pickupCode: ''
				},
				paymentMethod: 'wechat',
				agreementChecked: false
			}
		},
		computed: {
			canProceedToStep2() {
				return this.senderInfo.name.trim() &&
					this.senderInfo.phone.trim() &&
					this.senderInfo.address.trim() &&
					this.receiverInfo.name.trim() &&
					this.receiverInfo.phone.trim() &&
					this.receiverInfo.address.trim() &&
					this.packageInfo.description.trim()
			},
			packageTypeText() {
				const types = {
					'document': '文件',
					'clothes': '衣物',
					'electronics': '电子产品',
					'other': '其他'
				}
				return types[this.packageInfo.type] || '其他'
			},
			weightText() {
				const weights = {
					1: '1kg以内',
					2: '1-3kg',
					3: '3-5kg',
					5: '5kg以上'
				}
				return weights[this.packageInfo.weight] || '1kg以内'
			},
			canSubmitOrder() {
				return this.selectedCellId && this.paymentMethod && this.agreementChecked
			}
		},
		onLoad() {
			this.generateTrackingNumber()
			this.generatePickupCode()
		},
		methods: {
			goBack() {
				if (this.currentStep > 1) {
					this.currentStep--
				} else {
					uni.navigateBack()
				}
			},
			autoFillSender() {
				// 模拟自动填充发件人信息
				this.senderInfo = {
					name: '张三',
					phone: '13800138000',
					address: '重庆市沙坪坝区重庆大学A区1号楼'
				}
				uni.showToast({
					title: '已填充发件人信息',
					icon: 'success'
				})
			},
			generateTrackingNumber() {
				// 生成快递单号
				const date = new Date()
				const year = date.getFullYear().toString().substr(2, 2)
				const month = (date.getMonth() + 1).toString().padStart(2, '0')
				const day = date.getDate().toString().padStart(2, '0')
				const random = Math.floor(100000 + Math.random() * 900000)
				this.orderInfo.trackingNumber = `YT${year}${month}${day}${random}`
			},
			generatePickupCode() {
				// 生成6位取件码
				this.orderInfo.pickupCode = Math.floor(100000 + Math.random() * 900000).toString()
			},
			goToStep1() {
				this.currentStep = 1
			},
			goToStep2() {
				if (this.canProceedToStep2) {
					this.currentStep = 2
				} else {
					uni.showToast({
						title: '请填写完整信息',
						icon: 'none'
					})
				}
			},
			goToStep3() {
				if (this.selectedCellId) {
					// 根据选择的仓体计算费用
					const selectedCell = this.availableCells.find(cell => cell.id === this.selectedCellId)
					let cabinetFee = 0
					if (selectedCell.size === '大') {
						cabinetFee = 5
					} else if (selectedCell.size === '中') {
						cabinetFee = 3
					}
					// 根据重量计算快递费
					let shippingFee = 12
					if (this.packageInfo.weight === 2) {
						shippingFee = 15
					} else if (this.packageInfo.weight === 3) {
						shippingFee = 18
					} else if (this.packageInfo.weight === 5) {
						shippingFee = 25
					}

					this.orderInfo.shippingFee = shippingFee
					this.orderInfo.cabinetFee = cabinetFee
					this.orderInfo.totalFee = shippingFee + cabinetFee

					this.currentStep = 3
				} else {
					uni.showToast({
						title: '请选择仓体',
						icon: 'none'
					})
				}
			},
			selectCell(cell) {
				if (!cell.available) return
				this.selectedCellId = cell.id
				this.selectedCellInfo = cell
			},
			submitOrder() {
				if (!this.canSubmitOrder) return

				uni.showLoading({
					title: '支付中...'
				})

				// 模拟支付请求
				setTimeout(() => {
					uni.hideLoading()

					// 更新仓体状态为占用
					const cellIndex = this.availableCells.findIndex(cell => cell.id === this.selectedCellId)
					if (cellIndex !== -1) {
						this.availableCells[cellIndex].available = false
					}

					// 跳转到成功页面
					this.currentStep = 4

					// 模拟通知快递员
					setTimeout(() => {
						uni.showToast({
							title: '已通知快递员取件',
							icon: 'success'
						})
					}, 500)
				}, 2000)
			},
			goHome() {
				uni.switchTab({
					url: '/pages/index/index'
				})
			},
			viewOrderDetail() {
				uni.showModal({
					title: '订单详情',
					content: `
快递单号: ${this.orderInfo.trackingNumber}
仓门位置: ${this.selectedCellInfo.cabinetName} ${this.selectedCellInfo.cellNumber}号仓
取件码: ${this.orderInfo.pickupCode}
支付金额: ¥${this.orderInfo.totalFee}
发件人: ${this.senderInfo.name} ${this.senderInfo.phone}
收件人: ${this.receiverInfo.name} ${this.receiverInfo.phone}
					`,
					showCancel: false,
					confirmText: '好的'
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

	.steps {
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 40rpx 30rpx 0;
		margin-top: 80rpx;
	}

	.step-item {
		display: flex;
		flex-direction: column;
		align-items: center;
		position: relative;
	}

	.step-number {
		width: 60rpx;
		height: 60rpx;
		border-radius: 50%;
		background-color: #ddd;
		color: white;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 28rpx;
		font-weight: bold;
		margin-bottom: 10rpx;
	}

	.step-item.active .step-number {
		background-color: #007AFF;
	}

	.step-text {
		font-size: 24rpx;
		color: #999;
	}

	.step-item.active .step-text {
		color: #007AFF;
		font-weight: bold;
	}

	.step-line {
		width: 100rpx;
		height: 4rpx;
		background-color: #ddd;
		margin: 0 20rpx;
	}

	.step-section {
		padding: 30rpx;
	}

	.section-title {
		font-size: 32rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 30rpx;
	}

	.info-card {
		background-color: white;
		border-radius: 20rpx;
		padding: 30rpx;
		margin-bottom: 30rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.card-header {
		display: flex;
		align-items: center;
		margin-bottom: 30rpx;
	}

	.card-title {
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
		margin-left: 15rpx;
		flex: 1;
	}

	.auto-fill {
		font-size: 24rpx;
		color: #007AFF;
	}

	.form-group {
		margin-bottom: 30rpx;
	}

	.form-group:last-child {
		margin-bottom: 0;
	}

	.form-label {
		font-size: 26rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 15rpx;
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
		height: 120rpx;
		border: 2rpx solid #e0e0e0;
		border-radius: 12rpx;
		padding: 20rpx 30rpx;
		font-size: 28rpx;
		color: #333;
	}

	.type-options {
		display: grid;
		grid-template-columns: repeat(2, 1fr);
		gap: 20rpx;
	}

	.type-item {
		height: 80rpx;
		border: 2rpx solid #e0e0e0;
		border-radius: 12rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 28rpx;
		color: #666;
	}

	.type-item.active {
		border-color: #007AFF;
		background-color: #e7f0ff;
		color: #007AFF;
	}

	.weight-selector {
		display: grid;
		grid-template-columns: repeat(2, 1fr);
		gap: 20rpx;
	}

	.weight-item {
		height: 80rpx;
		border: 2rpx solid #e0e0e0;
		border-radius: 12rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 28rpx;
		color: #666;
	}

	.weight-item.active {
		border-color: #34C759;
		background-color: #e7f7ef;
		color: #34C759;
	}

	.step-actions {
		display: flex;
		gap: 20rpx;
		margin-top: 40rpx;
	}

	.prev-btn,
	.next-btn,
	.pay-btn {
		flex: 1;
		height: 90rpx;
		border-radius: 12rpx;
		font-size: 32rpx;
		font-weight: bold;
	}

	.prev-btn {
		background-color: white;
		border: 2rpx solid #007AFF;
		color: #007AFF;
	}

	.next-btn {
		background: linear-gradient(135deg, #007AFF, #5856D6);
		color: white;
		border: none;
	}

	.next-btn:disabled {
		opacity: 0.5;
	}

	.pay-btn {
		background: linear-gradient(135deg, #34C759, #2ECC71);
		color: white;
		border: none;
	}

	.pay-btn:disabled {
		opacity: 0.5;
	}

	.cabinet-info {
		background-color: white;
		border-radius: 20rpx;
		padding: 30rpx;
		margin-bottom: 30rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.cabinet-name {
		display: block;
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 10rpx;
	}

	.cabinet-location {
		display: block;
		font-size: 24rpx;
		color: #666;
	}

	.cabinet-grid {
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		gap: 20rpx;
		margin-bottom: 40rpx;
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
		position: relative;
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

	.cabinet-item.recommended {
		border-color: #FF9500;
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
		position: absolute;
		top: 10rpx;
		right: 10rpx;
		font-size: 20rpx;
		color: #FF3B30;
	}

	.cell-recommended {
		position: absolute;
		bottom: 10rpx;
		font-size: 20rpx;
		color: #FF9500;
	}

	.size-guide {
		background-color: white;
		border-radius: 20rpx;
		padding: 30rpx;
		margin-bottom: 40rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.guide-title {
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 20rpx;
	}

	.guide-items {
		display: flex;
		justify-content: space-between;
	}

	.guide-item {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.size-icon {
		width: 60rpx;
		height: 60rpx;
		border-radius: 50%;
		margin-bottom: 10rpx;
	}

	.size-icon.small {
		background-color: #e7f7ef;
		border: 2rpx solid #34C759;
	}

	.size-icon.medium {
		background-color: #e7f0ff;
		border: 2rpx solid #007AFF;
	}

	.size-icon.large {
		background-color: #fff3e0;
		border: 2rpx solid #FF9500;
	}

	.size-text {
		font-size: 22rpx;
		color: #666;
	}

	.order-summary {
		background-color: white;
		border-radius: 20rpx;
		padding: 30rpx;
		margin-bottom: 30rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.summary-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 20rpx 0;
		border-bottom: 1rpx solid #f0f0f0;
	}

	.summary-item:last-child {
		border-bottom: none;
	}

	.item-label {
		font-size: 26rpx;
		color: #666;
	}

	.item-value {
		font-size: 26rpx;
		font-weight: bold;
		color: #333;
	}

	.price-breakdown {
		background-color: white;
		border-radius: 20rpx;
		padding: 30rpx;
		margin-bottom: 30rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.price-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 20rpx 0;
	}

	.price-item.total {
		border-top: 1rpx solid #f0f0f0;
		margin-top: 20rpx;
		padding-top: 30rpx;
	}

	.price-label {
		font-size: 26rpx;
		color: #666;
	}

	.price-item.total .price-label {
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
	}

	.price-value {
		font-size: 26rpx;
		color: #333;
	}

	.price-item.total .price-value {
		font-size: 32rpx;
		font-weight: bold;
		color: #FF3B30;
	}

	.payment-methods {
		background-color: white;
		border-radius: 20rpx;
		padding: 30rpx;
		margin-bottom: 30rpx;
		box-shadow: 0 4rpx 12rpx rgba(0, 0, 0, 0.05);
	}

	.payment-title {
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 20rpx;
	}

	.method-list {
		display: flex;
		flex-direction: column;
		gap: 20rpx;
	}

	.method-item {
		display: flex;
		align-items: center;
		padding: 25rpx;
		border: 2rpx solid #e0e0e0;
		border-radius: 12rpx;
	}

	.method-item.selected {
		border-color: #007AFF;
		background-color: #e7f0ff;
	}

	.method-name {
		flex: 1;
		font-size: 28rpx;
		color: #333;
		margin-left: 20rpx;
	}

	.agreement {
		margin-bottom: 40rpx;
	}

	.agreement-check {
		display: flex;
		align-items: center;
	}

	.agreement-text {
		font-size: 24rpx;
		color: #666;
		margin-left: 10rpx;
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
	}

	.order-details {
		background-color: #f8f8f8;
		border-radius: 12rpx;
		padding: 30rpx;
		margin-bottom: 40rpx;
		text-align: left;
	}

	.detail-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 15rpx 0;
	}

	.detail-label {
		font-size: 26rpx;
		color: #666;
	}

	.detail-value {
		font-size: 26rpx;
		font-weight: bold;
		color: #333;
	}

	.instructions {
		text-align: left;
		margin-bottom: 40rpx;
	}

	.instructions-title {
		display: block;
		font-size: 28rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 20rpx;
	}

	.instructions-text {
		display: block;
		font-size: 24rpx;
		color: #666;
		margin-bottom: 10rpx;
		line-height: 1.5;
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

	.bottom-tips {
		padding: 0 30rpx;
		text-align: center;
		margin-top: 30rpx;
	}

	.tip-text {
		font-size: 22rpx;
		color: #999;
	}
</style>
