<template>
	<view class="page">
		<view class="header">
			<image class="logo" src="/static/logo.png" mode="aspectFill" />
			<text class="title">v-shop 登录</text>
			<text class="sub">登录后可进入购物车与个人中心</text>
		</view>

		<view class="card">
			<uni-forms :modelValue="form" label-position="top">
				<uni-forms-item label="账号">
					<uni-easyinput v-model="form.account" placeholder="手机号 / 用户名" :inputBorder="false" />
				</uni-forms-item>
				<uni-forms-item label="密码">
					<uni-easyinput v-model="form.password" type="password" placeholder="请输入密码" :inputBorder="false" />
				</uni-forms-item>
			</uni-forms>

			<button class="btn" type="primary" @click="submit">登录</button>
			<button class="btn btn--ghost" @click="skip">暂不登录</button>
		</view>

		<view class="tips">
			<text class="tips-text">演示项目：任意账号密码即可登录</text>
		</view>
	</view>
</template>

<script setup>
	import { ref } from 'vue'
	import { onLoad } from '@dcloudio/uni-app'

	const TOKEN_KEY = 'vshop_token'
	const USER_KEY = 'vshop_user'

	const redirectUrl = ref('')
	const redirectType = ref('')

	const form = ref({
		account: '',
		password: ''
	})

	onLoad((query) => {
		redirectUrl.value = query?.redirect ? String(query.redirect) : ''
		redirectType.value = query?.redirectType ? String(query.redirectType) : ''
	})

	function submit() {
		const account = String(form.value.account || '').trim()
		const password = String(form.value.password || '').trim()
		if (!account || !password) {
			uni.showToast({
				title: '请输入账号和密码',
				icon: 'none'
			})
			return
		}

		uni.setStorageSync(TOKEN_KEY, `token_${Date.now()}`)
		uni.setStorageSync(USER_KEY, JSON.stringify({
			id: '10001',
			nickname: account,
			avatar: '/static/logo.png'
		}))

		uni.showToast({
			title: '登录成功',
			icon: 'none'
		})

		setTimeout(() => {
			if (redirectUrl.value) {
				if (redirectType.value === 'switchTab') {
					uni.switchTab({ url: redirectUrl.value })
				} else {
					uni.redirectTo({ url: redirectUrl.value })
				}
				return
			}

			const pages = getCurrentPages()
			if (pages.length > 1) {
				uni.navigateBack({ delta: 1 })
			} else {
				uni.switchTab({ url: '/pages/index/index' })
			}
		}, 250)
	}

	function skip() {
		uni.switchTab({
			url: '/pages/index/index'
		})
	}
</script>

<style scoped>
	.page {
		height: calc(100vh - var(--window-bottom) - var(--window-top));
		background-color: #f5f5f5;
		padding: 48rpx 24rpx 24rpx;
		box-sizing: border-box;
	}

	.header {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.logo {
		width: 120rpx;
		height: 120rpx;
		border-radius: 28rpx;
		background-color: #ffffff;
	}

	.title {
		margin-top: 20rpx;
		font-size: 36rpx;
		color: #333333;
		font-weight: 800;
	}

	.sub {
		margin-top: 10rpx;
		font-size: 24rpx;
		color: #999999;
	}

	.card {
		margin-top: 36rpx;
		background-color: #ffffff;
		border-radius: 24rpx;
		padding: 24rpx;
	}

	.btn {
		margin-top: 16rpx;
		height: 84rpx;
		line-height: 84rpx;
		border-radius: 9999rpx;
		font-size: 28rpx;
	}

	.btn--ghost {
		background-color: #f5f5f5;
		color: #333333;
	}

	.tips {
		margin-top: 28rpx;
		display: flex;
		justify-content: center;
	}

	.tips-text {
		font-size: 22rpx;
		color: #999999;
	}
</style>
