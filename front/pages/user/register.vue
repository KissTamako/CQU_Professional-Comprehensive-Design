<template>
	<view class="container">
		<!-- 顶部导航栏 -->
		<uni-nav-bar title="注册绑定" left-icon="back" @clickLeft="goBack" backgroundColor="#007AFF" color="#fff" fixed="true"
			statusBar="true">
		</uni-nav-bar>

		<!-- 注册表单 -->
		<view class="register-form">
			<view class="form-title">用户注册</view>

			<view class="form-group">
				<view class="form-label">手机号码</view>
				<view class="input-with-verify">
					<input class="form-input" v-model="registerInfo.phone" placeholder="请输入手机号码" type="number" maxlength="11" />
					<button class="verify-btn" :disabled="countdown > 0" @click="sendVerifyCode">
						{{ countdown > 0 ? `${countdown}s后重发` : '获取验证码' }}
					</button>
				</view>
			</view>

			<view class="form-group" v-if="showVerifyCode">
				<view class="form-label">验证码</view>
				<input class="form-input" v-model="registerInfo.verifyCode" placeholder="请输入验证码" type="number" maxlength="6" />
			</view>

			<view class="form-group">
				<view class="form-label">密码</view>
				<input class="form-input" v-model="registerInfo.password" placeholder="请输入密码" type="password" />
			</view>

			<view class="form-group">
				<view class="form-label">确认密码</view>
				<input class="form-input" v-model="registerInfo.confirmPassword" placeholder="请再次输入密码" type="password" />
			</view>

			<view class="form-group">
				<view class="form-label">真实姓名</view>
				<input class="form-input" v-model="registerInfo.realName" placeholder="请输入真实姓名" />
			</view>

			<view class="form-group">
				<view class="form-label">身份证号</view>
				<input class="form-input" v-model="registerInfo.idCard" placeholder="请输入身份证号码" />
			</view>

			<view class="form-group">
				<view class="form-label">绑定方式</view>
				<view class="bind-options">
					<view class="bind-item" :class="{ active: registerInfo.bindType === 'phone' }"
						@click="registerInfo.bindType = 'phone'">
						<uni-icons type="phone" size="24" color="#007AFF"></uni-icons>
						<text>手机绑定</text>
					</view>
					<view class="bind-item" :class="{ active: registerInfo.bindType === 'face' }"
						@click="registerInfo.bindType = 'face'">
						<uni-icons type="camera" size="24" color="#FF9500"></uni-icons>
						<text>人脸绑定</text>
					</view>
					<view class="bind-item" :class="{ active: registerInfo.bindType === 'wechat' }"
						@click="registerInfo.bindType = 'wechat'">
						<uni-icons type="weixin" size="24" color="#07C160"></uni-icons>
						<text>微信绑定</text>
					</view>
				</view>
			</view>

			<view class="form-group" v-if="registerInfo.bindType === 'face'">
				<view class="form-label">人脸录入</view>
				<view class="face-upload">
					<view class="upload-placeholder" @click="uploadFace">
						<uni-icons type="camera" size="40" color="#007AFF"></uni-icons>
						<text class="upload-text">点击录入人脸</text>
					</view>
				</view>
			</view>

			<view class="agreement">
				<view class="agreement-check" @click="agreementChecked = !agreementChecked">
					<uni-icons :type="agreementChecked ? 'checkbox-filled' : 'circle'" size="20"
						:color="agreementChecked ? '#007AFF' : '#999'"></uni-icons>
					<text class="agreement-text">我已阅读并同意《用户服务协议》和《隐私政策》</text>
				</view>
			</view>

			<view class="form-actions">
				<button class="register-btn" :disabled="!canRegister" @click="handleRegister">注册并绑定</button>
			</view>

			<view class="login-link">
				<text>已有账号？</text>
				<text class="link-text" @click="goToLogin">立即登录</text>
			</view>
		</view>

		<!-- 注册成功 -->
		<view class="success-section" v-if="showSuccess">
			<view class="success-card">
				<uni-icons type="checkmarkempty" size="60" color="#34C759"></uni-icons>
				<text class="success-title">注册成功！</text>
				<text class="success-desc">您的账号已成功注册并绑定</text>

				<view class="user-info">
					<view class="info-item">
						<text class="info-label">手机号码</text>
						<text class="info-value">{{ registerInfo.phone }}</text>
					</view>
					<view class="info-item">
						<text class="info-label">真实姓名</text>
						<text class="info-value">{{ registerInfo.realName }}</text>
					</view>
					<view class="info-item">
						<text class="info-label">绑定方式</text>
						<text class="info-value">{{ bindTypeText }}</text>
					</view>
				</view>

				<view class="success-actions">
					<button class="action-btn" @click="goHome">返回首页</button>
					<button class="action-btn primary" @click="goToCourier">去快递员认证</button>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				registerInfo: {
					phone: '',
					verifyCode: '',
					password: '',
					confirmPassword: '',
					realName: '',
					idCard: '',
					bindType: 'phone'
				},
				showVerifyCode: false,
				countdown: 0,
				agreementChecked: false,
				showSuccess: false
			}
		},
		computed: {
			canRegister() {
				return this.registerInfo.phone.trim() &&
					this.registerInfo.password.trim() &&
					this.registerInfo.confirmPassword.trim() &&
					this.registerInfo.realName.trim() &&
					this.registerInfo.idCard.trim() &&
					this.agreementChecked &&
					this.registerInfo.password === this.registerInfo.confirmPassword
			},
			bindTypeText() {
				const types = {
					'phone': '手机绑定',
					'face': '人脸绑定',
					'wechat': '微信绑定'
				}
				return types[this.registerInfo.bindType] || '手机绑定'
			}
		},
		methods: {
			goBack() {
				uni.navigateBack()
			},
			sendVerifyCode() {
				if (!this.registerInfo.phone || this.registerInfo.phone.length !== 11) {
					uni.showToast({
						title: '请输入正确的手机号码',
						icon: 'none'
					})
					return
				}

				// 开始倒计时
				this.countdown = 60
				this.showVerifyCode = true

				const timer = setInterval(() => {
					this.countdown--
					if (this.countdown <= 0) {
						clearInterval(timer)
					}
				}, 1000)

				// 模拟发送验证码
				uni.showToast({
					title: '验证码已发送',
					icon: 'success'
				})
			},
			uploadFace() {
				uni.showToast({
					title: '人脸录入功能',
					icon: 'none'
				})
			},
			handleRegister() {
				if (!this.canRegister) return

				// 验证密码是否一致
				if (this.registerInfo.password !== this.registerInfo.confirmPassword) {
					uni.showToast({
						title: '两次密码不一致',
						icon: 'none'
					})
					return
				}

				// 验证身份证格式
				if (!this.validateIdCard(this.registerInfo.idCard)) {
					uni.showToast({
						title: '身份证格式不正确',
						icon: 'none'
					})
					return
				}

				uni.showLoading({
					title: '注册中...'
				})

				// 模拟注册请求
				setTimeout(() => {
					uni.hideLoading()

					// 保存用户信息到本地存储
					const userInfo = {
						phone: this.registerInfo.phone,
						realName: this.registerInfo.realName,
						bindType: this.registerInfo.bindType,
						role: 'user'
					}
					uni.setStorageSync('userInfo', userInfo)
					uni.setStorageSync('isLoggedIn', true)

					this.showSuccess = true
				}, 1500)
			},
			validateIdCard(idCard) {
				// 简单的身份证验证
				return idCard.length === 18 || idCard.length === 15
			},
			goToLogin() {
				uni.navigateTo({
					url: '/pages/login/login'
				})
			},
			goHome() {
				uni.switchTab({
					url: '/pages/index/index'
				})
			},
			goToCourier() {
				uni.showToast({
					title: '跳转到快递员认证',
					icon: 'none'
				})
				// 实际项目中这里应该是跳转到快递员认证页面
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

	.register-form {
		padding: 30rpx;
		margin-top: 80rpx;
	}

	.form-title {
		font-size: 36rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 40rpx;
		text-align: center;
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

	.bind-options {
		display: flex;
		justify-content: space-between;
	}

	.bind-item {
		flex: 1;
		height: 100rpx;
		border: 2rpx solid #e0e0e0;
		border-radius: 12rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		margin: 0 10rpx;
	}

	.bind-item.active {
		border-color: #007AFF;
		background-color: #e7f0ff;
	}

	.face-upload {
		text-align: center;
	}

	.upload-placeholder {
		width: 200rpx;
		height: 200rpx;
		border: 2rpx dashed #007AFF;
		border-radius: 12rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		margin: 0 auto;
	}

	.upload-text {
		font-size: 24rpx;
		color: #007AFF;
		margin-top: 15rpx;
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

	.form-actions {
		margin-bottom: 30rpx;
	}

	.register-btn {
		width: 100%;
		height: 90rpx;
		background: linear-gradient(135deg, #007AFF, #5856D6);
		border-radius: 12rpx;
		border: none;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 32rpx;
		font-weight: bold;
		color: white;
	}

	.register-btn:disabled {
		opacity: 0.5;
	}

	.login-link {
		text-align: center;
		font-size: 26rpx;
		color: #666;
	}

	.link-text {
		color: #007AFF;
		font-weight: bold;
		margin-left: 10rpx;
	}

	.success-section {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background-color: rgba(0, 0, 0, 0.5);
		display: flex;
		align-items: center;
		justify-content: center;
		z-index: 1000;
	}

	.success-card {
		background-color: white;
		border-radius: 20rpx;
		padding: 50rpx 30rpx;
		width: 80%;
		text-align: center;
		box-shadow: 0 10rpx 40rpx rgba(0, 0, 0, 0.2);
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

	.user-info {
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
</style>
