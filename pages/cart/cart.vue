<template>
	<view class="page">
		<view v-if="!isLogin" class="blocked"></view>

		<template v-else>
			<view v-if="cartItems.length === 0" class="empty">
				<uni-icons type="cart" size="64" color="#c7c7c7"></uni-icons>
				<text class="empty-text">购物车还是空的</text>
				<button class="empty-btn" type="primary" @click="goHome">去逛逛</button>
			</view>

			<view v-else class="layout">
				<scroll-view class="list" scroll-y :show-scrollbar="false">
					<view class="group">
						<view class="group-title">
							<uni-icons type="shop" size="18" color="#333333"></uni-icons>
							<text class="group-title-text">v-shop 自营</text>
						</view>

						<view v-for="item in cartItems" :key="item.id" class="item">
							<view class="check" @click="toggleItem(item.id)">
								<view :class="['checkbox', item.checked ? 'checkbox--checked' : '']">
									<uni-icons v-if="item.checked" type="checkmarkempty" size="16" color="#ffffff"></uni-icons>
								</view>
							</view>

							<image class="item-img" :src="item.image" mode="aspectFill" />

							<view class="item-body">
								<text class="item-title">{{ item.title }}</text>
								<text class="item-sku">{{ item.sku }}</text>

								<view class="item-footer">
									<text class="item-price">¥{{ item.price }}</text>
									<view class="stepper">
										<view class="stepper-btn" :class="item.quantity <= 1 ? 'stepper-btn--disabled' : ''"
											@click="decrease(item.id)">-</view>
										<text class="stepper-num">{{ item.quantity }}</text>
										<view class="stepper-btn" @click="increase(item.id)">+</view>
									</view>
								</view>
							</view>

							<view class="remove" @click="removeItem(item.id)">
								<uni-icons type="trash" size="18" color="#999999"></uni-icons>
							</view>
						</view>
					</view>

					<view class="safe-area"></view>
				</scroll-view>

				<view class="bar">
					<view class="bar-left" @click="toggleAll">
						<view :class="['checkbox', allChecked ? 'checkbox--checked' : '']">
							<uni-icons v-if="allChecked" type="checkmarkempty" size="16" color="#ffffff"></uni-icons>
						</view>
						<text class="bar-text">全选</text>
					</view>

					<view class="bar-right">
						<view class="bar-total">
							<text class="bar-total-label">合计：</text>
							<text class="bar-total-value">¥{{ totalPrice }}</text>
						</view>
						<button class="bar-btn" type="primary" :disabled="checkedCount === 0" @click="checkout">
							去结算({{ checkedCount }})
						</button>
					</view>
				</view>
			</view>
		</template>
	</view>
</template>

<script setup>
	import { computed, ref } from 'vue'
	import { onShow } from '@dcloudio/uni-app'

	const TOKEN_KEY = 'vshop_token'
	const isLogin = ref(false)
	const isRedirecting = ref(false)

	onShow(() => {
		isLogin.value = Boolean(uni.getStorageSync(TOKEN_KEY))
		if (!isLogin.value && !isRedirecting.value) {
			isRedirecting.value = true
			uni.navigateTo({
				url: '/pages/login/login?redirect=/pages/cart/cart&redirectType=switchTab'
			})
			setTimeout(() => {
				isRedirecting.value = false
			}, 350)
		}
	})

	const cartItems = ref([{
		id: 'g1',
		title: '旗舰手机 12GB+256GB',
		sku: '黑色 / 256GB',
		price: 3999,
		image: '/static/c4.png',
		quantity: 1,
		checked: true
	}, {
		id: 'g5',
		title: '真无线降噪耳机',
		sku: '白色 / 标准版',
		price: 499,
		image: '/static/c8.png',
		quantity: 2,
		checked: true
	}, {
		id: 'g3',
		title: '智能空气炸锅 5L 大容量',
		sku: '5L / 黑色',
		price: 299,
		image: '/static/c6.png',
		quantity: 1,
		checked: false
	}])

	const checkedItems = computed(() => cartItems.value.filter(i => i.checked))
	const checkedCount = computed(() => checkedItems.value.reduce((sum, i) => sum + i.quantity, 0))
	const totalPrice = computed(() => {
		const total = checkedItems.value.reduce((sum, i) => sum + i.price * i.quantity, 0)
		return total.toFixed(2)
	})
	const allChecked = computed(() => cartItems.value.length > 0 && cartItems.value.every(i => i.checked))

	function toggleItem(id) {
		const item = cartItems.value.find(i => i.id === id)
		if (!item) return
		item.checked = !item.checked
	}

	function toggleAll() {
		const next = !allChecked.value
		cartItems.value.forEach(i => {
			i.checked = next
		})
	}

	function increase(id) {
		const item = cartItems.value.find(i => i.id === id)
		if (!item) return
		item.quantity += 1
	}

	function decrease(id) {
		const item = cartItems.value.find(i => i.id === id)
		if (!item) return
		if (item.quantity <= 1) return
		item.quantity -= 1
	}

	function removeItem(id) {
		const index = cartItems.value.findIndex(i => i.id === id)
		if (index === -1) return
		uni.showModal({
			title: '提示',
			content: '确定删除该商品吗？',
			success(res) {
				if (res.confirm) {
					cartItems.value.splice(index, 1)
				}
			}
		})
	}

	function checkout() {
		if (checkedCount.value === 0) return
		uni.showToast({
			title: `结算 ${checkedCount.value} 件，合计 ¥${totalPrice.value}`,
			icon: 'none'
		})
	}

	function goHome() {
		uni.switchTab({
			url: '/pages/index/index'
		})
	}
</script>

<style scoped>
	.page {
		height: calc(100vh - var(--window-bottom) - var(--window-top));
		background-color: #f5f5f5;
	}

	.blocked {
		height: 100%;
	}

	.layout {
		height: 100%;
		position: relative;
	}

	.list {
		height: 100%;
		padding: 24rpx;
		box-sizing: border-box;
	}

	.group {
		background-color: #ffffff;
		background-color: #ffffff;
		border-radius: 24rpx;
		overflow: hidden;
	}

	.group-title {
		display: flex;
		align-items: center;
		padding: 20rpx 24rpx;
		border-bottom: 2rpx solid #f2f2f2;
	}

	.group-title-text {
		margin-left: 10rpx;
		font-size: 28rpx;
		color: #333333;
		font-weight: 700;
	}

	.item {
		display: flex;
		align-items: flex-start;
		padding: 24rpx;
		border-bottom: 2rpx solid #f2f2f2;
	}

	.item:last-child {
		border-bottom: none;
	}

	.check {
		padding-top: 52rpx;
		margin-right: 16rpx;
	}

	.checkbox {
		width: 40rpx;
		height: 40rpx;
		border-radius: 10rpx;
		border: 2rpx solid #d0d0d0;
		background-color: #ffffff;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.checkbox--checked {
		border-color: #2979ff;
		background-color: #2979ff;
	}

	.item-img {
		width: 180rpx;
		height: 180rpx;
		border-radius: 16rpx;
		background-color: #f5f5f5;
		margin-right: 20rpx;
		flex-shrink: 0;
	}

	.item-body {
		flex: 1;
		min-width: 0;
	}

	.item-title {
		display: block;
		font-size: 28rpx;
		color: #333333;
		line-height: 40rpx;
		height: 80rpx;
		overflow: hidden;
	}

	.item-sku {
		display: inline-block;
		margin-top: 10rpx;
		padding: 6rpx 12rpx;
		background-color: #f5f5f5;
		border-radius: 9999rpx;
		font-size: 22rpx;
		color: #666666;
		line-height: 32rpx;
	}

	.item-footer {
		margin-top: 16rpx;
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.item-price {
		font-size: 30rpx;
		color: #e64340;
		font-weight: 600;
	}

	.stepper {
		display: flex;
		align-items: center;
		border-radius: 16rpx;
		overflow: hidden;
		border: 2rpx solid #e9e9e9;
	}

	.stepper-btn {
		width: 56rpx;
		height: 48rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 32rpx;
		color: #333333;
		background-color: #ffffff;
	}

	.stepper-btn--disabled {
		color: #c7c7c7;
		background-color: #f5f5f5;
	}

	.stepper-num {
		width: 64rpx;
		text-align: center;
		font-size: 26rpx;
		color: #333333;
		line-height: 48rpx;
	}

	.remove {
		padding-top: 50rpx;
		padding-left: 12rpx;
	}

	.safe-area {
		height: 160rpx;
	}

	.bar {
		position: fixed;
		left: 0;
		right: 0;
		bottom: 50px;
		padding: 16rpx 24rpx;
		background-color: #ffffff;
		border-top: 2rpx solid #f2f2f2;
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.bar-left {
		display: flex;
		align-items: center;
	}

	.bar-text {
		margin-left: 12rpx;
		font-size: 26rpx;
		color: #333333;
	}

	.bar-right {
		display: flex;
		align-items: center;
	}

	.bar-total {
		margin-right: 18rpx;
		display: flex;
		align-items: baseline;
	}

	.bar-total-label {
		font-size: 24rpx;
		color: #666666;
	}

	.bar-total-value {
		font-size: 34rpx;
		color: #e64340;
		font-weight: 700;
	}

	.bar-btn {
		height: 72rpx;
		padding: 0 26rpx;
		border-radius: 9999rpx;
		font-size: 28rpx;
		line-height: 72rpx;
	}

	.empty {
		height: 100%;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 24rpx;
		box-sizing: border-box;
	}

	.empty-text {
		margin-top: 24rpx;
		font-size: 28rpx;
		color: #666666;
	}

	.empty-btn {
		margin-top: 24rpx;
		width: 240rpx;
		border-radius: 9999rpx;
	}
</style>
