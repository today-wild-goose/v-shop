<template>
	<view class="page">
		<view class="search-fixed">
			<view class="search-inner">
				<uni-search-bar v-model="keyword" placeholder="搜索商品" cancelButton="none" clearButton="auto" radius="100"
					@focus="onSearchFocus" @confirm="onSearchConfirm" @clear="onSearchClear" />
			</view>
		</view>
		<view class="div-line"></view>
		<view class="banner">
			<uni-swiper-dot :info="banners" :current="bannerCurrent" mode="round" :dots-styles="dotsStyles">
				<swiper class="swiper" circular autoplay :interval="3000" :duration="400" @change="onBannerChange">
					<swiper-item v-for="item in banners" :key="item.id">
						<image class="banner-img" :src="item.image" mode="aspectFill" />
					</swiper-item>
				</swiper>
			</uni-swiper-dot>
		</view>
		<view class="div-line"></view>
		<uni-section title="分类" type="line" padding>
			<scroll-view class="tabs" scroll-x :show-scrollbar="false">
				<view class="tabs-inner">
					<view v-for="(item, index) in categories" :key="item.id"
						:class="['tab', index === activeCategoryIndex ? 'tab--active' : '']" @click="selectCategory(index)">
						<text :class="['tab-text', index === activeCategoryIndex ? 'tab-text--active' : '']">
							{{ item.name }}
						</text>
					</view>
				</view>
			</scroll-view>
		</uni-section>
		<view class="div-line"></view>
		<uni-section title="为你推荐" type="line" padding>
			<view class="goods-list">
				<view v-for="goods in filteredGoods" :key="goods.id" class="goods-card" @click="openGoods(goods)">
					<image class="goods-img" :src="goods.image" mode="aspectFill" />
					<view class="goods-body">
						<text class="goods-title">{{ goods.title }}</text>
						<view class="goods-footer">
							<text class="goods-price">¥{{ goods.price }}</text>
							<uni-tag v-if="goods.tag" :text="goods.tag" type="primary" size="mini" inverted></uni-tag>
						</view>
					</view>
				</view>
			</view>
		</uni-section>

		<view v-if="showBackTop" class="back-top" @click="backToTop">
			<uni-icons type="arrow-up" size="22" color="#ffffff"></uni-icons>
		</view>
	</view>
</template>

<script setup>
	import { computed, ref } from 'vue'
	import { onPageScroll } from '@dcloudio/uni-app'
	const data = ref(null)
	const keyword = ref('')
	const bannerCurrent = ref(0)
	const banners = ref([{
		id: 'b1',
		image: '/static/c1.png'
	}, {
		id: 'b2',
		image: '/static/c2.png'
	}, {
		id: 'b3',
		image: '/static/c3.png'
	}])

	const dotsStyles = {
		backgroundColor: 'rgba(255, 255, 255, 0.6)',
		border: '1px rgba(255, 255, 255, 0.8) solid',
		selectedBackgroundColor: '#2979ff',
		selectedBorder: '1px #2979ff solid'
	}

	const activeCategoryIndex = ref(0)
	const categories = ref([{
		id: 'featured',
		name: '精选'
	}, {
		id: 'phone',
		name: '手机'
	}, {
		id: 'computer',
		name: '电脑'
	}, {
		id: 'appliance',
		name: '家电'
	}, {
		id: 'snack',
		name: '零食'
	}])

	const goodsList = ref([{
		id: 'g1',
		categoryId: 'phone',
		title: '旗舰手机 12GB+256GB',
		price: '3999.00',
		image: '/static/c4.png',
		tag: '新品'
	}, {
		id: 'g2',
		categoryId: 'computer',
		title: '轻薄笔记本 i7/16G/1T',
		price: '6299.00',
		image: '/static/c5.png',
		tag: '热卖'
	}, {
		id: 'g3',
		categoryId: 'appliance',
		title: '智能空气炸锅 5L 大容量',
		price: '299.00',
		image: '/static/c6.png',
		tag: '优惠'
	}, {
		id: 'g4',
		categoryId: 'snack',
		title: '坚果礼盒 混合口味 30 包',
		price: '89.00',
		image: '/static/c7.png',
		tag: ''
	}, {
		id: 'g5',
		categoryId: 'phone',
		title: '真无线降噪耳机',
		price: '499.00',
		image: '/static/c8.png',
		tag: ''
	}, {
		id: 'g6',
		categoryId: 'computer',
		title: '机械键盘 RGB 104 键',
		price: '199.00',
		image: '/static/c9.png',
		tag: ''
	}])

	const filteredGoods = computed(() => {
		const categoryId = categories.value[activeCategoryIndex.value]?.id
		if (!categoryId || categoryId === 'featured') {
			return goodsList.value
		}
		return goodsList.value.filter(item => item.categoryId === categoryId)
	})

	function onBannerChange(e) {
		bannerCurrent.value = e.detail.current
	}

	function selectCategory(index) {
		activeCategoryIndex.value = index
	}

	function openGoods(goods) {
		uni.navigateTo({
			url: `/pages/goods-detail/goods-detail?id=${goods.id}`
		})
	}

	function onSearchConfirm(e) {
		const value = (e?.value ?? keyword.value).trim()
		if (!value) return
		uni.showToast({
			title: `搜索：${value}`,
			icon: 'none'
		})
	}

	function onSearchClear() {
		keyword.value = ''
	}

	function onSearchFocus() {
		uni.navigateTo({
			url: '/pages/search/search'
		})
	}

	const showBackTop = ref(false)

	onPageScroll((e) => {
		showBackTop.value = (e?.scrollTop ?? 0) > 260
	})

	function backToTop() {
		uni.pageScrollTo({
			scrollTop: 0,
			duration: 300
		})
	}
	uni.request({
		url:"http://127.0.0.1:4523/m1/6364752-6060938-default/api",
		success:(res)=>{
			data.value =  res.data
		}
	})
	console.log(data.value);
</script>

<style scoped>
	.div-line {
		height: 24rpx;
		background-color: #f5f5f5;
	}

	.page {
		padding: 24rpx;
		padding-top: 168rpx;
	}

	.search-fixed {
		position: fixed;
		left: 0;
		right: 0;
		top: 0;
		z-index: 1000;
		padding: 24rpx;
		background-color: #f5f5f5;
		margin-top: 44px;
	}

	.search-inner {
		background-color: #ffffff;
		border-radius: 24rpx;
		overflow: hidden;
	}

	.banner {
		overflow: hidden;
		background-color: #ffffff;
		border-radius: 24rpx;
	}

	.swiper {
		height: 320rpx;
	}

	.banner-img {
		width: 100%;
		height: 320rpx;
		display: block;
	}

	.tabs {
		width: 100%;
	}

	.tabs-inner {
		display: flex;
		padding: 8rpx 0 16rpx;
	}

	.tab {
		padding: 12rpx 24rpx;
		margin-right: 20rpx;
		background-color: #f5f5f5;
		border-radius: 9999rpx;
		white-space: nowrap;
	}

	.tab--active {
		background-color: #2979ff;
	}

	.tab-text {
		font-size: 26rpx;
		color: #333333;
		line-height: 36rpx;
	}

	.tab-text--active {
		color: #ffffff;
	}

	.goods-list {
		display: flex;
		flex-wrap: wrap;
		justify-content: space-between;
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

	.back-top {
		position: fixed;
		right: 24rpx;
		bottom: 140rpx;
		width: 88rpx;
		height: 88rpx;
		border-radius: 9999rpx;
		background-color: rgba(41, 121, 255, 0.9);
		display: flex;
		align-items: center;
		justify-content: center;
		box-shadow: 0 12rpx 24rpx rgba(0, 0, 0, 0.12);
	}
</style>
