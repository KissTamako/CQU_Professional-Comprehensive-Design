<template>
	<view class="login-container">
		<!-- 顶部logo和标题 -->
		<view class="login-header">
			<image class="logo" src="/static/logo.png"></image>
			<text class="app-title">快递柜综合应用系统</text>
			<text class="app-subtitle">快递员工作平台</text>
		</view>

		<!-- 登录表单 -->
		<view class="login-form">
			<view class="form-title">账号登录</view>
			
			<view class="input-group">
				<view class="input-item">
					<uni-icons type="person" size="20" color="#007AFF" class="input-icon"></uni-icons>
					<input 
						class="input" 
						type="text" 
						v-model="username" 
						placeholder="请输入工号/手机号"
						placeholder-class="placeholder"
						@focus="onInputFocus('username')"
						@blur="onInputBlur"
					/>
				</view>
				<view class="input-item">
					<uni-icons type="locked" size="20" color="#007AFF" class="input-icon"></uni-icons>
					<input 
						class="input" 
						type="password" 
						v-model="password" 
						placeholder="请输入密码"
						placeholder-class="placeholder"
						@focus="onInputFocus('password')"
						@blur="onInputBlur"
					/>
					<view class="password-toggle" @click="togglePasswordVisibility">
						<uni-icons :type="showPassword ? 'eye-slash' : 'eye'" size="20" color="#666"></uni-icons>
					</view>
				</view>
			</view>

			<!-- 记住密码和忘记密码 -->
			<view class="form-options">
				<view class="remember-me" @click="toggleRemember">
					<uni-icons :type="rememberMe ? 'checkbox-filled' : 'circle'" size="18" :color="rememberMe ? '#007AFF' : '#999'"></uni-icons>
					<text class="option-text">记住密码</text>
				</view>
				<view class="forgot-password" @click="navigateToForgot">
					<text class="option-text">忘记密码?</text>
				</view>
			</view>

			<!-- 登录按钮 -->
			<button 
				class="login-btn" 
				:class="{ 'login-btn-disabled': !canLogin }" 
				:disabled="!canLogin"
				@click="handleLogin"
			>
				<text class="login-btn-text">{{ loading ? '登录中...' : '登录' }}</text>
			</button>

			<!-- 其他登录方式 -->
			<view class="other-login">
				<view class="divider">
					<view class="divider-line"></view>
					<text class="divider-text">其他登录方式</text>
					<view class="divider-line"></view>
				</view>
				<view class="login-methods">
					<view class="method-item" @click="wechatLogin">
						<uni-icons type="weixin" size="30" color="#07C160"></uni-icons>
						<text class="method-text">微信</text>
					</view>
					<view class="method-item" @click="smsLogin">
						<uni-icons type="chat" size="30" color="#FF9500"></uni-icons>
						<text class="method-text">短信</text>
					</view>
					<view class="method-item" @click="faceLogin">
						<uni-icons type="camera" size="30" color="#5856D6"></uni-icons>
						<text class="method-text">刷脸</text>
					</view>
				</view>
			</view>

			<!-- 注册提示 -->
			<view class="register-tip">
				<text>还没有账号? </text>
				<text class="register-link" @click="navigateToRegister">立即注册</text>
			</view>
		</view>

		<!-- 版本信息 -->
		<view class="version-info">
			<text>v1.0.0 © 2024 快递柜系统</text>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				username: '',
				password: '',
				showPassword: false,
				rememberMe: false,
				loading: false,
				focusedInput: null
			}
		},
		computed: {
			canLogin() {
				return this.username.trim().length > 0 && this.password.trim().length > 0 && !this.loading
			}
		},
		methods: {
			onInputFocus(field) {
				this.focusedInput = field
			},
			onInputBlur() {
				this.focusedInput = null
			},
			togglePasswordVisibility() {
				this.showPassword = !this.showPassword
			},
			toggleRemember() {
				this.rememberMe = !this.rememberMe
			},
			async handleLogin() {
				if (!this.canLogin) return
				
				this.loading = true
				
				// 模拟登录请求
				setTimeout(() => {
					this.loading = false
					
					// 登录成功，设置登录状态
					uni.setStorageSync('isLoggedIn', true)
					
					// 如果勾选了记住密码，保存用户信息
					if (this.rememberMe) {
						uni.setStorageSync('rememberedUser', {
							username: this.username,
							password: this.password
						})
					} else {
						uni.removeStorageSync('rememberedUser')
					}
					
					// 登录成功提示
					uni.showToast({
						title: '登录成功',
						icon: 'success',
						duration: 2000
					})
					
					// 跳转到快递员工作台
					setTimeout(() => {
						uni.switchTab({
							url: '/pages/courier/courier'
						})
					}, 1500)
					
				}, 1500)
			},
			navigateToForgot() {
				uni.showToast({
					title: '跳转到忘记密码页面',
					icon: 'none'
				})
			},
			navigateToRegister() {
				uni.showToast({
					title: '跳转到注册页面',
					icon: 'none'
				})
			},
			wechatLogin() {
				uni.showToast({
					title: '微信登录',
					icon: 'none'
				})
			},
			smsLogin() {
				uni.showToast({
					title: '短信登录',
					icon: 'none'
				})
			},
			faceLogin() {
				uni.showToast({
					title: '刷脸登录',
					icon: 'none'
				})
			}
		},
		onLoad() {
			// 尝试从本地存储加载记住的账号密码
			const savedUser = uni.getStorageSync('rememberedUser')
			if (savedUser) {
				this.username = savedUser.username || ''
				this.password = savedUser.password || ''
				this.rememberMe = true
			}
		}
	}
</script>

<style lang="scss">
	.login-container {
		min-height: 100vh;
		background: linear-gradient(135deg, #007AFF, #5856D6);
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 40rpx;
	}

	.login-header {
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-bottom: 80rpx;
	}

	.logo {
		width: 120rpx;
		height: 120rpx;
		border-radius: 24rpx;
		margin-bottom: 30rpx;
		background-color: white;
		padding: 20rpx;
	}

	.app-title {
		font-size: 40rpx;
		font-weight: bold;
		color: white;
		margin-bottom: 10rpx;
	}

	.app-subtitle {
		font-size: 28rpx;
		color: rgba(255, 255, 255, 0.9);
	}

	.login-form {
		width: 100%;
		background-color: white;
		border-radius: 24rpx;
		padding: 60rpx 50rpx;
		box-shadow: 0 20rpx 60rpx rgba(0, 0, 0, 0.1);
	}

	.form-title {
		font-size: 36rpx;
		font-weight: bold;
		color: #333;
		margin-bottom: 50rpx;
		text-align: center;
	}

	.input-group {
		margin-bottom: 40rpx;
	}

	.input-item {
		display: flex;
		align-items: center;
		border: 2rpx solid #e0e0e0;
		border-radius: 12rpx;
		padding: 30rpx;
		margin-bottom: 30rpx;
		position: relative;
		transition: all 0.3s ease;
	}

	.input-item:focus-within {
		border-color: #007AFF;
		box-shadow: 0 0 0 2rpx rgba(0, 122, 255, 0.2);
	}

	.input-icon {
		margin-right: 20rpx;
	}

	.input {
		flex: 1;
		font-size: 28rpx;
		color: #333;
		height: 40rpx;
		line-height: 40rpx;
	}

	.placeholder {
		color: #999;
		font-size: 28rpx;
	}

	.password-toggle {
		padding: 10rpx;
		margin-left: 10rpx;
	}

	.form-options {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 50rpx;
	}

	.remember-me {
		display: flex;
		align-items: center;
	}

	.option-text {
		font-size: 26rpx;
		color: #666;
		margin-left: 10rpx;
	}

	.forgot-password .option-text {
		color: #007AFF;
	}

	.login-btn {
		width: 100%;
		height: 90rpx;
		background: linear-gradient(135deg, #007AFF, #5856D6);
		border-radius: 12rpx;
		border: none;
		display: flex;
		align-items: center;
		justify-content: center;
		transition: all 0.3s ease;
	}

	.login-btn-disabled {
		opacity: 0.6;
	}

	.login-btn-text {
		font-size: 32rpx;
		font-weight: bold;
		color: white;
	}

	.other-login {
		margin-top: 60rpx;
	}

	.divider {
		display: flex;
		align-items: center;
		justify-content: center;
		margin-bottom: 40rpx;
	}

	.divider-line {
		flex: 1;
		height: 1rpx;
		background-color: #e0e0e0;
	}

	.divider-text {
		font-size: 24rpx;
		color: #999;
		padding: 0 20rpx;
	}

	.login-methods {
		display: flex;
		justify-content: space-around;
	}

	.method-item {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.method-text {
		font-size: 22rpx;
		color: #666;
		margin-top: 10rpx;
	}

	.register-tip {
		text-align: center;
		margin-top: 50rpx;
		font-size: 26rpx;
		color: #666;
	}

	.register-link {
		color: #007AFF;
		font-weight: bold;
	}

	.version-info {
		margin-top: 60rpx;
		text-align: center;
		font-size: 22rpx;
		color: rgba(255, 255, 255, 0.7);
	}
</style>
