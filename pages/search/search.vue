<template>
	<view class="page">
		<view class="search-fixed">
			<view class="search-inner">
				<uni-search-bar v-model="keyword" :focus="true" placeholder="搜索商品" cancelButton="auto" clearButton="auto" radius="100"
					@confirm="onSearchConfirm" @cancel="onCancel" @clear="onSearchClear" />
			</view>
		</view>

		<scroll-view class="content" scroll-y :show-scrollbar="false">
			<view v-if="!hasSearched" class="section">
				<text class="section-title">热搜</text>
				<view class="chips">
					<view v-for="w in hotKeywords" :key="w" class="chip" @click="applyKeyword(w)">
						<text class="chip-text">{{ w }}</text>
					</view>
				</view>
			</view>

			<view v-if="!hasSearched && histories.length > 0" class="section">
				<view class="section-head">
					<text class="section-title">搜索历史</text>
					<view class="section-action" @click="clearHistories">
						<uni-icons type="trash" size="18" color="#999999"></uni-icons>
					</view>
				</view>
				<view class="chips">
					<view v-for="w in histories" :key="w" class="chip chip--ghost" @click="applyKeyword(w)">
						<text class="chip-text">{{ w }}</text>
					</view>
				</view>
			</view>

			<view v-if="hasSearched" class="section">
				<view class="result-head">
					<text class="result-title">搜索结果</text>
					<text class="result-sub">{{ results.length }} 件</text>
				</view>

				<view v-if="results.length === 0" class="empty">
					<text class="empty-text">没有找到相关商品</text>
				</view>

				<view v-else class="goods-list">
					<view v-for="g in results" :key="g.id" class="goods-card" @click="openGoods(g)">
						<image class="goods-img" :src="g.image" mode="aspectFill" />
						<view class="goods-body">
							<text class="goods-title">{{ g.title }}</text>
							<view class="goods-footer">
								<text class="goods-price">¥{{ g.price }}</text>
								<uni-tag v-if="g.tag" :text="g.tag" type="primary" size="mini" inverted></uni-tag>
							</view>
						</view>
					</view>
				</view>
			</view>

			<view class="bottom-safe"></view>
		</scroll-view>
	</view>
</template>

<script setup>
	import { computed, ref } from 'vue'
	import { onLoad } from '@dcloudio/uni-app'

	const STORAGE_SEARCH_HISTORY = 'vshop_search_history'

	const keyword = ref('')
	const hasSearched = ref(false)
	const hotKeywords = ref(['手机', '笔记本', '耳机', '空气炸锅', '键盘', '零食'])
	const histories = ref([])

	const allGoods = ref([
		{ id: 'g1', title: '旗舰手机 12GB+256GB', price: '3999.00', image: '/static/c4.png', tag: '新品' },
		{ id: 'g2', title: '轻薄笔记本 i7/16G/1T', price: '6299.00', image: '/static/c5.png', tag: '热卖' },
		{ id: 'g3', title: '智能空气炸锅 5L 大容量', price: '299.00', image: '/static/c6.png', tag: '优惠' },
		{ id: 'g4', title: '坚果礼盒 混合口味 30 包', price: '89.00', image: '/static/c7.png', tag: '' },
		{ id: 'g5', title: '真无线降噪耳机', price: '499.00', image: '/static/c8.png', tag: '' },
		{ id: 'g6', title: '机械键盘 RGB 104 键', price: '199.00', image: '/static/c9.png', tag: '' }
	])

	const results = ref([])

	onLoad((query) => {
		loadHistories()
		if (query?.q) {
			keyword.value = String(query.q)
			search()
		}
	})

	function loadHistories() {
		try {
			const raw = uni.getStorageSync(STORAGE_SEARCH_HISTORY)
			const list = raw ? JSON.parse(raw) : []
			histories.value = Array.isArray(list) ? list.slice(0, 12) : []
		} catch (e) {
			histories.value = []
		}
	}

	function saveHistory(word) {
		const w = String(word || '').trim()
		if (!w) return
		const set = new Set([w, ...histories.value])
		histories.value = Array.from(set).slice(0, 12)
		uni.setStorageSync(STORAGE_SEARCH_HISTORY, JSON.stringify(histories.value))
	}

	function clearHistories() {
		histories.value = []
		uni.removeStorageSync(STORAGE_SEARCH_HISTORY)
	}

	function applyKeyword(w) {
		keyword.value = w
		search()
	}

	function onSearchConfirm(e) {
		const v = (e?.value ?? keyword.value).trim()
		keyword.value = v
		search()
	}

	function onSearchClear() {
		keyword.value = ''
		results.value = []
		hasSearched.value = false
	}

	function onCancel() {
		uni.navigateBack({ delta: 1 })
	}

	function search() {
		const q = keyword.value.trim()
		if (!q) return
		saveHistory(q)
		results.value = allGoods.value.filter(g => g.title.includes(q))
		hasSearched.value = true
	}

	function openGoods(g) {
		uni.navigateTo({
			url: `/pages/goods-detail/goods-detail?id=${g.id}`
		})
	}
</script>

<style scoped>
	.page {
		height: calc(100vh - var(--window-bottom) - var(--window-top));
		background-color: #f5f5f5;
	}

	.search-fixed {
		position: fixed;
		left: 0;
		right: 0;
		top: 0;
		z-index: 1000;
		padding: 24rpx;
		background-color: #f5f5f5;
	}

	.search-inner {
		background-color: #ffffff;
		border-radius: 24rpx;
		overflow: hidden;
	}

	.content {
		height: 100%;
		padding: 96rpx 24rpx 24rpx;
		box-sizing: border-box;
	}

	.section {
		margin-top: 24rpx;
		background-color: #ffffff;
		border-radius: 24rpx;
		padding: 24rpx;
	}

	.section-head {
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.section-title {
		font-size: 30rpx;
		color: #333333;
		font-weight: 700;
	}

	.section-action {
		padding: 8rpx;
	}

	.chips {
		margin-top: 16rpx;
		display: flex;
		flex-wrap: wrap;
		gap: 16rpx;
	}

	.chip {
		padding: 12rpx 20rpx;
		background-color: rgba(41, 121, 255, 0.1);
		border-radius: 9999rpx;
	}

	.chip--ghost {
		background-color: #f5f5f5;
	}

	.chip-text {
		font-size: 24rpx;
		color: #2979ff;
	}

	.result-head {
		display: flex;
		align-items: baseline;
		justify-content: space-between;
		margin-bottom: 12rpx;
	}

	.result-title {
		font-size: 30rpx;
		color: #333333;
		font-weight: 700;
	}

	.result-sub {
		font-size: 24rpx;
		color: #999999;
	}

	.empty {
		padding: 24rpx 0 8rpx;
	}

	.empty-text {
		font-size: 24rpx;
		color: #999999;
	}

	.goods-list {
		display: flex;
		flex-wrap: wrap;
		justify-content: space-between;
		margin-top: 12rpx;
	}

	.goods-card {
		width: calc(50% - 12rpx);
		margin-bottom: 24rpx;
		overflow: hidden;
		background-color: #ffffff;
		border-radius: 24rpx;
	}

	.goods-img {
		width: 100%;
		height: 280rpx;
		display: block;
	}

	.goods-body {
		padding: 20rpx;
	}

	.goods-title {
		display: block;
		font-size: 28rpx;
		color: #333333;
		line-height: 40rpx;
		height: 80rpx;
		overflow: hidden;
	}

	.goods-footer {
		margin-top: 16rpx;
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.goods-price {
		font-size: 30rpx;
		color: #e64340;
		font-weight: 600;
	}

	.bottom-safe {
		height: 60rpx;
	}
</style>
