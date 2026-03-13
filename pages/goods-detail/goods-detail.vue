<template>
	<view class="page">
		<scroll-view class="content" scroll-y :show-scrollbar="false">
			<view class="banner">
				<uni-swiper-dot :info="goods.images" :current="bannerCurrent" mode="round" :dots-styles="dotsStyles">
					<swiper class="swiper" circular autoplay :interval="3200" :duration="400" @change="onBannerChange">
						<swiper-item v-for="(img, index) in goods.images" :key="`${goods.id}-${index}`">
							<image class="banner-img" :src="img" mode="aspectFill" />
						</swiper-item>
					</swiper>
				</uni-swiper-dot>
			</view>

			<view class="info">
				<view class="price-row">
					<text class="price">¥{{ goods.price }}</text>
					<view class="tags">
						<uni-tag v-for="t in goods.tags" :key="t" :text="t" type="primary" size="mini" inverted />
					</view>
				</view>
				<text class="title">{{ goods.title }}</text>
				<text class="sub">{{ goods.subtitle }}</text>
			</view>

			<view class="block" @click="openSkuPopup">
				<view class="block-left">
					<text class="block-label">已选</text>
					<text class="block-value">{{ selectedSkuText }}，{{ quantity }}件</text>
				</view>
				<uni-icons type="right" size="16" color="#999999"></uni-icons>
			</view>

			<view class="block">
				<view class="block-left">
					<text class="block-label">服务</text>
					<text class="block-value">7天无理由 · 极速退款 · 正品保障</text>
				</view>
			</view>

			<view class="seg">
				<uni-segmented-control :current="activeSegment" :values="segmentValues" style-type="text"
					active-color="#2979ff" @clickItem="onSegmentClick" />
			</view>

			<view v-if="activeSegment === 0" class="detail">
				<view class="detail-card">
					<text class="detail-title">商品详情</text>
					<text class="detail-text">{{ goods.detail }}</text>
				</view>
			</view>

			<view v-else-if="activeSegment === 1" class="detail">
				<view class="detail-card">
					<text class="detail-title">参数信息</text>
					<view class="params">
						<view v-for="row in goods.params" :key="row.k" class="param-row">
							<text class="param-k">{{ row.k }}</text>
							<text class="param-v">{{ row.v }}</text>
						</view>
					</view>
				</view>
			</view>

			<view v-else class="detail">
				<view class="detail-card">
					<text class="detail-title">用户评价</text>
					<view v-if="goods.comments.length === 0" class="empty-comment">
						<text class="empty-comment-text">暂无评价</text>
					</view>
					<view v-else class="comments">
						<view v-for="c in goods.comments" :key="c.id" class="comment">
							<view class="comment-head">
								<image class="comment-avatar" :src="c.avatar" mode="aspectFill" />
								<view class="comment-user">
									<text class="comment-name">{{ c.name }}</text>
									<text class="comment-time">{{ c.time }}</text>
								</view>
							</view>
							<text class="comment-content">{{ c.content }}</text>
						</view>
					</view>
				</view>
			</view>

			<view class="bottom-safe"></view>
		</scroll-view>

		<view class="nav">
			<uni-goods-nav :options="navOptions" :button-group="buttonGroup" @click="onNavClick" @buttonClick="onButtonClick" />
		</view>

		<uni-popup ref="skuPopupRef" type="bottom" background-color="#ffffff">
			<view class="sku">
				<view class="sku-head">
					<image class="sku-img" :src="goods.images[0]" mode="aspectFill" />
					<view class="sku-head-info">
						<text class="sku-price">¥{{ goods.price }}</text>
						<text class="sku-selected">已选：{{ selectedSkuText }}</text>
					</view>
					<view class="sku-close" @click="closeSkuPopup">
						<uni-icons type="closeempty" size="20" color="#999999"></uni-icons>
					</view>
				</view>

				<view class="sku-section">
					<text class="sku-label">规格</text>
					<view class="sku-options">
						<view v-for="(s, idx) in goods.skus" :key="s.id"
							:class="['sku-option', idx === selectedSkuIndex ? 'sku-option--active' : '']"
							@click="selectSku(idx)">
							<text :class="['sku-option-text', idx === selectedSkuIndex ? 'sku-option-text--active' : '']">
								{{ s.text }}
							</text>
						</view>
					</view>
				</view>

				<view class="sku-section sku-qty">
					<text class="sku-label">数量</text>
					<uni-number-box v-model="quantity" :min="1" :max="99" />
				</view>

				<view class="sku-actions">
					<button class="sku-btn sku-btn--cart" @click="addToCart(true)">加入购物车</button>
					<button class="sku-btn sku-btn--buy" type="primary" @click="buyNow(true)">立即购买</button>
				</view>
			</view>
		</uni-popup>
	</view>
</template>

<script setup>
	import { computed, ref } from 'vue'
	import { onLoad, onShow } from '@dcloudio/uni-app'

	const STORAGE_CART_KEY = 'vshop_cart'

	const goodsId = ref('g1')
	const bannerCurrent = ref(0)
	const activeSegment = ref(0)
	const segmentValues = ['详情', '参数', '评价']

	const selectedSkuIndex = ref(0)
	const quantity = ref(1)
	const cartCount = ref(0)
	const skuPopupRef = ref()

	const dotsStyles = {
		backgroundColor: 'rgba(255, 255, 255, 0.6)',
		border: '1px rgba(255, 255, 255, 0.8) solid',
		selectedBackgroundColor: '#2979ff',
		selectedBorder: '1px #2979ff solid'
	}

	const goodsData = {
		g1: {
			id: 'g1',
			title: '旗舰手机 12GB+256GB',
			subtitle: '高刷屏幕 · 旗舰芯片 · 影像升级',
			price: '3999.00',
			images: ['/static/c4.png', '/static/c1.png', '/static/c2.png'],
			tags: ['新品', '包邮'],
			skus: [{ id: 'g1-s1', text: '黑色 / 256GB' }, { id: 'g1-s2', text: '白色 / 256GB' }, { id: 'g1-s3', text: '蓝色 / 512GB' }],
			detail: '旗舰性能与轻薄手感兼得，适合游戏、拍摄与日常高强度使用。',
			params: [{ k: '屏幕', v: '6.7英寸 高刷屏' }, { k: '内存', v: '12GB' }, { k: '存储', v: '256GB/512GB' }, { k: '电池', v: '5000mAh' }],
			comments: [{
				id: 'c1',
				name: '小明',
				time: '2026-03-13',
				avatar: '/static/logo.png',
				content: '手感不错，屏幕很细腻，拍照也好看。'
			}]
		},
		g2: {
			id: 'g2',
			title: '轻薄笔记本 i7/16G/1T',
			subtitle: '轻薄便携 · 高性能办公 · 长续航',
			price: '6299.00',
			images: ['/static/c5.png', '/static/c8.png', '/static/c9.png'],
			tags: ['热卖', '分期'],
			skus: [{ id: 'g2-s1', text: '银色 / 16G / 1T' }, { id: 'g2-s2', text: '灰色 / 32G / 1T' }],
			detail: '适合通勤与学习，兼顾性能与便携，日常办公更高效。',
			params: [{ k: '处理器', v: 'i7' }, { k: '内存', v: '16GB/32GB' }, { k: '存储', v: '1TB SSD' }, { k: '重量', v: '约1.3kg' }],
			comments: []
		},
		g3: {
			id: 'g3',
			title: '智能空气炸锅 5L 大容量',
			subtitle: '少油更健康 · 多档控温 · 容量够用',
			price: '299.00',
			images: ['/static/c6.png', '/static/c9.png', '/static/c1.png'],
			tags: ['优惠', '次日达'],
			skus: [{ id: 'g3-s1', text: '5L / 黑色' }, { id: 'g3-s2', text: '5L / 白色' }],
			detail: '告别油烟，外酥里嫩，一键搞定多种美味。',
			params: [{ k: '容量', v: '5L' }, { k: '功率', v: '1500W' }, { k: '控温', v: '80-200℃' }],
			comments: []
		},
		g4: {
			id: 'g4',
			title: '坚果礼盒 混合口味 30 包',
			subtitle: '每日一包 · 办公零食 · 健康加餐',
			price: '89.00',
			images: ['/static/c7.png', '/static/c2.png', '/static/c3.png'],
			tags: ['爆款'],
			skus: [{ id: 'g4-s1', text: '混合口味 / 30包' }],
			detail: '多种口味一次满足，送礼自用都合适。',
			params: [{ k: '规格', v: '30包' }, { k: '口味', v: '混合' }],
			comments: []
		},
		g5: {
			id: 'g5',
			title: '真无线降噪耳机',
			subtitle: '主动降噪 · 低延迟 · 久戴舒适',
			price: '499.00',
			images: ['/static/c8.png', '/static/c4.png', '/static/c5.png'],
			tags: ['推荐'],
			skus: [{ id: 'g5-s1', text: '白色 / 标准版' }, { id: 'g5-s2', text: '黑色 / 标准版' }],
			detail: '沉浸降噪，通勤旅行更安静，游戏观影更尽兴。',
			params: [{ k: '降噪', v: '主动降噪' }, { k: '续航', v: '约24小时（含充电盒）' }],
			comments: []
		},
		g6: {
			id: 'g6',
			title: '机械键盘 RGB 104 键',
			subtitle: '手感清脆 · 全键无冲 · 氛围灯效',
			price: '199.00',
			images: ['/static/c9.png', '/static/c6.png', '/static/c7.png'],
			tags: ['性价比'],
			skus: [{ id: 'g6-s1', text: '青轴 / RGB' }, { id: 'g6-s2', text: '红轴 / RGB' }],
			detail: '办公游戏两相宜，手感扎实，灯效丰富。',
			params: [{ k: '轴体', v: '青轴/红轴' }, { k: '键位', v: '104键' }, { k: '连接', v: '有线' }],
			comments: []
		}
	}

	const goods = computed(() => goodsData[goodsId.value] ?? goodsData.g1)
	const selectedSkuText = computed(() => goods.value.skus[selectedSkuIndex.value]?.text ?? '默认规格')

	const navOptions = computed(() => [{
		icon: 'shop',
		text: '店铺'
	}, {
		icon: 'cart',
		text: '购物车',
		info: cartCount.value > 0 ? String(cartCount.value) : ''
	}])

	const buttonGroup = [{
		text: '加入购物车',
		backgroundColor: '#ff9900',
		color: '#ffffff'
	}, {
		text: '立即购买',
		backgroundColor: '#2979ff',
		color: '#ffffff'
	}]

	onLoad((query) => {
		if (query?.id) goodsId.value = String(query.id)
		refreshCartCount()
	})

	onShow(() => {
		refreshCartCount()
	})

	function onBannerChange(e) {
		bannerCurrent.value = e.detail.current
	}

	function onSegmentClick(e) {
		activeSegment.value = e.currentIndex
	}

	function openSkuPopup() {
		skuPopupRef.value?.open?.()
	}

	function closeSkuPopup() {
		skuPopupRef.value?.close?.()
	}

	function selectSku(index) {
		selectedSkuIndex.value = index
	}

	function onNavClick(e) {
		if (e.index === 1) {
			uni.switchTab({
				url: '/pages/cart/cart'
			})
		}
	}

	function onButtonClick(e) {
		if (e.index === 0) addToCart(false)
		if (e.index === 1) buyNow(false)
	}

	function readCart() {
		try {
			const raw = uni.getStorageSync(STORAGE_CART_KEY)
			const list = raw ? JSON.parse(raw) : []
			return Array.isArray(list) ? list : []
		} catch (e) {
			return []
		}
	}

	function writeCart(list) {
		uni.setStorageSync(STORAGE_CART_KEY, JSON.stringify(list))
	}

	function refreshCartCount() {
		const list = readCart()
		cartCount.value = list.reduce((sum, i) => sum + (Number(i.quantity) || 0), 0)
	}

	function addToCart(fromPopup) {
		if (!fromPopup) {
			openSkuPopup()
			return
		}

		const sku = goods.value.skus[selectedSkuIndex.value]
		const list = readCart()
		const idx = list.findIndex(i => i.goodsId === goods.value.id && i.skuId === sku?.id)
		if (idx >= 0) {
			list[idx].quantity = Math.min(99, (Number(list[idx].quantity) || 0) + Number(quantity.value || 1))
		} else {
			list.unshift({
				goodsId: goods.value.id,
				skuId: sku?.id ?? '',
				title: goods.value.title,
				skuText: sku?.text ?? '',
				price: Number(goods.value.price),
				image: goods.value.images[0],
				quantity: Number(quantity.value || 1),
				checked: true
			})
		}
		writeCart(list)
		refreshCartCount()
		closeSkuPopup()
		uni.showToast({
			title: '已加入购物车',
			icon: 'none'
		})
	}

	function buyNow(fromPopup) {
		if (!fromPopup) {
			openSkuPopup()
			return
		}
		closeSkuPopup()
		uni.showToast({
			title: '购买流程（待实现）',
			icon: 'none'
		})
	}

	function onSearchConfirm(e) {
		const value = (e?.value ?? '').trim()
		if (!value) return
		uni.showToast({
			title: `搜索：${value}`,
			icon: 'none'
		})
	}

	function onSearchClear() {
	}
</script>

<style scoped>
	.page {
		height: calc(100vh - var(--window-bottom) - var(--window-top));
		background-color: #f5f5f5;
	}

	.content {
		height: 100%;
		box-sizing: border-box;
	}

	.banner {
		background-color: #ffffff;
		overflow: hidden;
	}

	.swiper {
		height: 640rpx;
	}

	.banner-img {
		width: 100%;
		height: 640rpx;
		display: block;
	}

	.info {
		padding: 24rpx;
		background-color: #ffffff;
	}

	.price-row {
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.price {
		font-size: 44rpx;
		color: #e64340;
		font-weight: 800;
	}

	.tags {
		display: flex;
		align-items: center;
		gap: 12rpx;
	}

	.title {
		display: block;
		margin-top: 12rpx;
		font-size: 32rpx;
		color: #333333;
		font-weight: 700;
		line-height: 46rpx;
	}

	.sub {
		display: block;
		margin-top: 8rpx;
		font-size: 24rpx;
		color: #999999;
		line-height: 34rpx;
	}

	.block {
		margin-top: 24rpx;
		background-color: #ffffff;
		padding: 22rpx 24rpx;
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.block-left {
		display: flex;
		align-items: center;
		min-width: 0;
	}

	.block-label {
		width: 76rpx;
		font-size: 24rpx;
		color: #999999;
	}

	.block-value {
		flex: 1;
		font-size: 26rpx;
		color: #333333;
		line-height: 36rpx;
		overflow: hidden;
	}

	.seg {
		margin-top: 24rpx;
		background-color: #ffffff;
		padding: 8rpx 0;
	}

	.detail {
		margin-top: 24rpx;
		padding: 0 24rpx;
	}

	.detail-card {
		background-color: #ffffff;
		border-radius: 24rpx;
		padding: 24rpx;
	}

	.detail-title {
		font-size: 30rpx;
		color: #333333;
		font-weight: 700;
		line-height: 42rpx;
	}

	.detail-text {
		margin-top: 16rpx;
		display: block;
		font-size: 26rpx;
		color: #666666;
		line-height: 40rpx;
	}

	.params {
		margin-top: 16rpx;
		display: flex;
		flex-direction: column;
	}

	.param-row {
		padding: 16rpx 0;
		display: flex;
		align-items: flex-start;
		border-bottom: 2rpx solid #f2f2f2;
	}

	.param-row:last-child {
		border-bottom: none;
	}

	.param-k {
		width: 160rpx;
		font-size: 24rpx;
		color: #999999;
		line-height: 34rpx;
	}

	.param-v {
		flex: 1;
		font-size: 24rpx;
		color: #333333;
		line-height: 34rpx;
	}

	.empty-comment {
		padding: 24rpx 0 8rpx;
	}

	.empty-comment-text {
		font-size: 24rpx;
		color: #999999;
	}

	.comments {
		margin-top: 16rpx;
		display: flex;
		flex-direction: column;
	}

	.comment {
		padding: 18rpx 0;
		border-bottom: 2rpx solid #f2f2f2;
	}

	.comment:last-child {
		border-bottom: none;
	}

	.comment-head {
		display: flex;
		align-items: center;
	}

	.comment-avatar {
		width: 56rpx;
		height: 56rpx;
		border-radius: 9999rpx;
		background-color: #f5f5f5;
	}

	.comment-user {
		margin-left: 14rpx;
		display: flex;
		flex-direction: column;
	}

	.comment-name {
		font-size: 24rpx;
		color: #333333;
		line-height: 34rpx;
	}

	.comment-time {
		font-size: 22rpx;
		color: #999999;
		line-height: 32rpx;
	}

	.comment-content {
		margin-top: 12rpx;
		display: block;
		font-size: 26rpx;
		color: #666666;
		line-height: 38rpx;
	}

	.bottom-safe {
		height: calc(140rpx + var(--window-bottom));
	}

	.nav {
		position: fixed;
		left: 0;
		right: 0;
		bottom: 0;
		padding-bottom: var(--window-bottom);
		background-color: #ffffff;
	}

	.sku {
		padding: 24rpx;
	}

	.sku-head {
		display: flex;
		align-items: flex-start;
	}

	.sku-img {
		width: 160rpx;
		height: 160rpx;
		border-radius: 16rpx;
		background-color: #f5f5f5;
		flex-shrink: 0;
	}

	.sku-head-info {
		flex: 1;
		margin-left: 18rpx;
		min-width: 0;
	}

	.sku-price {
		display: block;
		font-size: 36rpx;
		color: #e64340;
		font-weight: 800;
		line-height: 46rpx;
	}

	.sku-selected {
		display: block;
		margin-top: 10rpx;
		font-size: 24rpx;
		color: #666666;
		line-height: 34rpx;
	}

	.sku-close {
		padding-left: 12rpx;
	}

	.sku-section {
		margin-top: 24rpx;
	}

	.sku-label {
		display: block;
		font-size: 26rpx;
		color: #333333;
		font-weight: 700;
		line-height: 38rpx;
	}

	.sku-options {
		margin-top: 14rpx;
		display: flex;
		flex-wrap: wrap;
		gap: 16rpx;
	}

	.sku-option {
		padding: 12rpx 20rpx;
		background-color: #f5f5f5;
		border-radius: 16rpx;
	}

	.sku-option--active {
		background-color: rgba(41, 121, 255, 0.12);
		border: 2rpx solid rgba(41, 121, 255, 0.5);
	}

	.sku-option-text {
		font-size: 24rpx;
		color: #333333;
	}

	.sku-option-text--active {
		color: #2979ff;
		font-weight: 700;
	}

	.sku-qty {
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.sku-actions {
		margin-top: 28rpx;
		display: flex;
		gap: 16rpx;
	}

	.sku-btn {
		flex: 1;
		height: 80rpx;
		line-height: 80rpx;
		border-radius: 9999rpx;
		font-size: 28rpx;
		color: #ffffff;
	}

	.sku-btn--cart {
		background-color: #ff9900;
	}

	.sku-btn--buy {
	}
</style>
