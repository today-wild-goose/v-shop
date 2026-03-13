<template>
	<view class="page">
		<view class="header">
			<view class="user" @click="openProfile">
				<image class="avatar" :src="user.avatar" mode="aspectFill" />
				<view class="user-info">
					<text class="nickname">{{ isLogin ? user.nickname : '未登录' }}</text>
					<text class="desc">{{ isLogin ? `ID: ${user.id}` : '点击登录 / 注册' }}</text>
				</view>
				<view class="user-arrow">
					<uni-icons type="right" size="18" color="#ffffff"></uni-icons>
				</view>
			</view>

			<view class="header-actions">
				<button v-if="!isLogin" class="header-btn" type="primary" @click.stop="login">登录</button>
				<button v-else class="header-btn header-btn--ghost" @click.stop="logout">退出登录</button>
			</view>
		</view>

		<view class="card">
			<view class="card-title">
				<text class="card-title-text">我的订单</text>
				<view class="card-title-more" @click="openOrders('all')">
					<text class="more-text">全部订单</text>
					<uni-icons type="right" size="16" color="#999999"></uni-icons>
				</view>
			</view>

			<uni-grid :column="5" :highlight="false" :show-border="false" :square="false">
				<uni-grid-item v-for="item in orderEntries" :key="item.id" @click.native="openOrders(item.id)">
					<view class="order-entry">
						<view class="order-icon">
							<text class="order-icon-text">{{ item.short }}</text>
							<uni-badge v-if="item.badge > 0" class="order-badge" :text="item.badge" type="error" size="small" />
						</view>
						<text class="order-text">{{ item.text }}</text>
					</view>
				</uni-grid-item>
			</uni-grid>
		</view>

		<view class="card">
			<view class="card-title">
				<text class="card-title-text">常用功能</text>
			</view>
			<uni-list :border="false">
				<uni-list-item title="收货地址" showArrow clickable @click="openFeature('收货地址')"></uni-list-item>
				<uni-list-item title="优惠券" showArrow clickable @click="openFeature('优惠券')"></uni-list-item>
				<uni-list-item title="联系客服" showArrow clickable @click="openFeature('联系客服')"></uni-list-item>
				<uni-list-item title="设置" showArrow clickable @click="openFeature('设置')"></uni-list-item>
			</uni-list>
		</view>

		<view class="footer">
			<text class="footer-text">v-shop · 1.0.0</text>
		</view>
	</view>
</template>

<script setup>
	import { ref } from 'vue'

	const isLogin = ref(false)
	const user = ref({
		avatar: '/static/logo.png',
		nickname: 'v-shop 用户',
		id: '10001'
	})

	const orderEntries = ref([{
		id: 'pay',
		text: '待付款',
		short: '付',
		badge: 2
	}, {
		id: 'ship',
		text: '待发货',
		short: '发',
		badge: 1
	}, {
		id: 'receive',
		text: '待收货',
		short: '收',
		badge: 0
	}, {
		id: 'comment',
		text: '待评价',
		short: '评',
		badge: 0
	}, {
		id: 'refund',
		text: '售后',
		short: '退',
		badge: 0
	}])

	function login() {
		isLogin.value = true
		uni.showToast({
			title: '已登录（示例）',
			icon: 'none'
		})
	}

	function logout() {
		uni.showModal({
			title: '提示',
			content: '确定退出登录吗？',
			success(res) {
				if (res.confirm) {
					isLogin.value = false
				}
			}
		})
	}

	function openProfile() {
		uni.showToast({
			title: isLogin.value ? '个人资料（待实现）' : '请先登录',
			icon: 'none'
		})
	}

	function openOrders(type) {
		if (!isLogin.value) {
			uni.showToast({
				title: '请先登录',
				icon: 'none'
			})
			return
		}
		uni.showToast({
			title: `订单：${type}`,
			icon: 'none'
		})
	}

	function openFeature(name) {
		uni.showToast({
			title: `${name}（待实现）`,
			icon: 'none'
		})
	}
</script>

<style scoped>
	.page {
		height: calc(100vh - var(--window-bottom) - var(--window-top));
		background-color: #f5f5f5;
		padding-bottom: 40rpx;
	}

	.header {
		padding: 40rpx 24rpx 24rpx;
		background: linear-gradient(180deg, #2979ff 0%, #4aa3ff 100%);
	}

	.user {
		display: flex;
		align-items: center;
	}

	.avatar {
		width: 112rpx;
		height: 112rpx;
		border-radius: 9999rpx;
		background-color: rgba(255, 255, 255, 0.3);
		border: 4rpx solid rgba(255, 255, 255, 0.55);
	}

	.user-info {
		flex: 1;
		margin-left: 20rpx;
		min-width: 0;
	}

	.nickname {
		display: block;
		font-size: 34rpx;
		color: #ffffff;
		font-weight: 700;
		line-height: 46rpx;
	}

	.desc {
		display: block;
		margin-top: 8rpx;
		font-size: 24rpx;
		color: rgba(255, 255, 255, 0.9);
		line-height: 34rpx;
	}

	.user-arrow {
		margin-left: 12rpx;
	}

	.header-actions {
		margin-top: 18rpx;
		display: flex;
	}

	.header-btn {
		height: 64rpx;
		line-height: 64rpx;
		font-size: 26rpx;
		border-radius: 9999rpx;
		padding: 0 28rpx;
	}

	.header-btn--ghost {
		color: #ffffff;
		background-color: rgba(255, 255, 255, 0.18);
	}

	.card {
		margin: 24rpx;
		background-color: #ffffff;
		border-radius: 24rpx;
		overflow: hidden;
	}

	.card-title {
		padding: 22rpx 24rpx;
		display: flex;
		align-items: center;
		justify-content: space-between;
		border-bottom: 2rpx solid #f2f2f2;
	}

	.card-title-text {
		font-size: 30rpx;
		color: #333333;
		font-weight: 700;
	}

	.card-title-more {
		display: flex;
		align-items: center;
	}

	.more-text {
		font-size: 24rpx;
		color: #999999;
		margin-right: 4rpx;
	}

	.order-entry {
		padding: 24rpx 0;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.order-icon {
		position: relative;
		width: 72rpx;
		height: 72rpx;
		border-radius: 9999rpx;
		background-color: rgba(41, 121, 255, 0.1);
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.order-icon-text {
		font-size: 30rpx;
		color: #2979ff;
		font-weight: 700;
		line-height: 30rpx;
	}

	.order-badge {
		position: absolute;
		right: -10rpx;
		top: -10rpx;
	}

	.order-text {
		margin-top: 12rpx;
		font-size: 24rpx;
		color: #333333;
		line-height: 34rpx;
	}

	.footer {
		padding: 10rpx 0 30rpx;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.footer-text {
		font-size: 22rpx;
		color: #999999;
	}
</style>
