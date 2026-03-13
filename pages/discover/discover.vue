<template>
	<view class="page">
		<scroll-view class="content" scroll-y :show-scrollbar="false">
			<view class="banner">
				<uni-swiper-dot :info="banners" :current="bannerCurrent" mode="round" :dots-styles="dotsStyles">
					<swiper class="swiper" circular autoplay :interval="3200" :duration="400" @change="onBannerChange">
						<swiper-item v-for="item in banners" :key="item.id">
							<image class="banner-img" :src="item.image" mode="aspectFill" />
						</swiper-item>
					</swiper>
				</uni-swiper-dot>
			</view>

			<uni-segmented-control class="seg" :current="activeTabIndex" :values="tabs" style-type="text"
				active-color="#2979ff" @clickItem="onTabClick" />

			<view class="topic-card">
				<view class="topic-left">
					<text class="topic-title">{{ activeTabTitle }} · 今日推荐</text>
					<text class="topic-desc">精选内容持续更新，看看大家都在逛什么</text>
				</view>
				<view class="topic-right" @click="openTopic">
					<text class="topic-btn">去看看</text>
					<uni-icons type="right" size="16" color="#ffffff"></uni-icons>
				</view>
			</view>

			<view class="feed">
				<view v-for="item in filteredFeed" :key="item.id" class="feed-item" @click="openFeed(item)">
					<image class="feed-img" :src="item.image" mode="aspectFill" />
					<view class="feed-body">
						<text class="feed-title">{{ item.title }}</text>
						<view class="feed-meta">
							<text class="feed-tag">{{ item.tag }}</text>
							<text class="feed-dot">·</text>
							<text class="feed-author">{{ item.author }}</text>
						</view>
					</view>
				</view>
			</view>

			<view class="bottom-safe"></view>
		</scroll-view>

		<view v-if="showBackTop" class="back-top" @click="backToTop">
			<uni-icons type="arrow-up" size="22" color="#ffffff"></uni-icons>
		</view>
	</view>
</template>

<script setup>
	import { computed, ref } from 'vue'
	import { onPageScroll } from '@dcloudio/uni-app'

	const keyword = ref('')

	const bannerCurrent = ref(0)
	const banners = ref([{
		id: 'b1',
		image: '/static/c2.png'
	}, {
		id: 'b2',
		image: '/static/c3.png'
	}, {
		id: 'b3',
		image: '/static/c4.png'
	}])

	const dotsStyles = {
		backgroundColor: 'rgba(255, 255, 255, 0.6)',
		border: '1px rgba(255, 255, 255, 0.8) solid',
		selectedBackgroundColor: '#2979ff',
		selectedBorder: '1px #2979ff solid'
	}

	const tabs = ref(['推荐', '活动', '种草', '专题'])
	const activeTabIndex = ref(0)
	const activeTabTitle = computed(() => tabs.value[activeTabIndex.value] ?? '推荐')

	const feed = ref([{
		id: 'f1',
		tab: '推荐',
		title: '本周爆款清单：这些单品值得入手',
		tag: '清单',
		author: 'v-shop 编辑部',
		image: '/static/c5.png'
	}, {
		id: 'f2',
		tab: '活动',
		title: '限时秒杀：热门好物低至 5 折',
		tag: '促销',
		author: '官方活动',
		image: '/static/c6.png'
	}, {
		id: 'f3',
		tab: '种草',
		title: '通勤必备：轻薄笔记本怎么选',
		tag: '攻略',
		author: '数码达人',
		image: '/static/c7.png'
	}, {
		id: 'f4',
		tab: '专题',
		title: '家电焕新季：厨房神器合集',
		tag: '专题',
		author: '生活方式',
		image: '/static/c8.png'
	}, {
		id: 'f5',
		tab: '推荐',
		title: '新手必看：购物避坑指南',
		tag: '指南',
		author: 'v-shop 编辑部',
		image: '/static/c9.png'
	}])

	const filteredFeed = computed(() => {
		const tab = activeTabTitle.value
		if (tab === '推荐') return feed.value
		return feed.value.filter(i => i.tab === tab)
	})

	function onBannerChange(e) {
		bannerCurrent.value = e.detail.current
	}

	function onTabClick(e) {
		activeTabIndex.value = e.currentIndex
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

	function openTopic() {
		uni.showToast({
			title: `${activeTabTitle.value}（待实现）`,
			icon: 'none'
		})
	}

	function openFeed(item) {
		uni.showToast({
			title: item.title,
			icon: 'none'
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
</script>

<style scoped>
	.page {
		height: calc(100vh - var(--window-bottom) - var(--window-top));
		background-color: #f5f5f5;
	}


	.content {
		height: 100%;
		padding: 168rpx 24rpx 24rpx;
		padding-top: 0;
		box-sizing: border-box;
	}

	.banner {
		overflow: hidden;
		background-color: #ffffff;
		border-radius: 24rpx;
		margin-top: 24rpx;
	}

	.swiper {
		height: 320rpx;
	}

	.banner-img {
		width: 100%;
		height: 320rpx;
		display: block;
	}

	.seg {
		margin-top: 24rpx;
		background-color: #ffffff;
		border-radius: 24rpx;
		overflow: hidden;
		padding: 6rpx 0;
	}

	.topic-card {
		margin-top: 24rpx;
		background: linear-gradient(135deg, #2979ff 0%, #4aa3ff 100%);
		border-radius: 24rpx;
		padding: 24rpx;
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.topic-left {
		flex: 1;
		min-width: 0;
	}

	.topic-title {
		display: block;
		font-size: 30rpx;
		color: #ffffff;
		font-weight: 700;
		line-height: 42rpx;
	}

	.topic-desc {
		display: block;
		margin-top: 10rpx;
		font-size: 24rpx;
		color: rgba(255, 255, 255, 0.9);
		line-height: 34rpx;
	}

	.topic-right {
		margin-left: 16rpx;
		padding: 14rpx 18rpx;
		border-radius: 9999rpx;
		background-color: rgba(255, 255, 255, 0.22);
		display: flex;
		align-items: center;
	}

	.topic-btn {
		font-size: 24rpx;
		color: #ffffff;
		margin-right: 6rpx;
	}

	.feed {
		margin-top: 24rpx;
		display: flex;
		flex-direction: column;
	}

	.feed-item {
		background-color: #ffffff;
		border-radius: 24rpx;
		overflow: hidden;
		display: flex;
		margin-bottom: 24rpx;
	}

	.feed-img {
		width: 220rpx;
		height: 180rpx;
		background-color: #f5f5f5;
		flex-shrink: 0;
	}

	.feed-body {
		flex: 1;
		padding: 20rpx;
		min-width: 0;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
	}

	.feed-title {
		display: block;
		font-size: 28rpx;
		color: #333333;
		line-height: 40rpx;
		height: 80rpx;
		overflow: hidden;
	}

	.feed-meta {
		display: flex;
		align-items: center;
		margin-top: 12rpx;
	}

	.feed-tag {
		font-size: 22rpx;
		color: #2979ff;
		background-color: rgba(41, 121, 255, 0.1);
		padding: 6rpx 12rpx;
		border-radius: 9999rpx;
	}

	.feed-dot {
		margin: 0 10rpx;
		font-size: 22rpx;
		color: #cccccc;
	}

	.feed-author {
		font-size: 22rpx;
		color: #999999;
	}

	.bottom-safe {
		height: 60rpx;
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
