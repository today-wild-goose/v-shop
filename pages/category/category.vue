<template>
	<view class="page">
		<view class="layout">
			<scroll-view class="left" scroll-y :show-scrollbar="false">
				<view class="left-inner">
					<view v-for="(item, index) in categoryList" :key="item.id"
						:class="['left-item', index === activeCategoryIndex ? 'left-item--active' : '']"
						@click="selectCategory(index)">
						<text :class="['left-text', index === activeCategoryIndex ? 'left-text--active' : '']">
							{{ item.name }}
						</text>
					</view>
				</view>
			</scroll-view>

			<scroll-view class="right" scroll-y :show-scrollbar="false" :scroll-top="rightScrollTop"
				@scroll="onRightScroll">
				<view class="right-inner">
					<view class="right-header">
						<text class="right-title">{{ activeCategory?.name }}</text>
						<text class="right-subtitle">精选二级分类</text>
					</view>

					<view class="subcats">
						<uni-grid :column="4" :highlight="false" :show-border="false" :square="false">
							<uni-grid-item v-for="item in activeSubCategories" :key="item.id" @click.native="openSubCategory(item)">
								<view class="subcat-item">
									<image class="subcat-icon" :src="item.icon" mode="aspectFill" />
									<text class="subcat-name">{{ item.name }}</text>
								</view>
							</uni-grid-item>
						</uni-grid>
					</view>

					<view class="goods-header">
						<text class="goods-title">推荐商品</text>
						<text class="goods-count">{{ activeGoods.length }} 件</text>
					</view>

					<view class="goods-list">
						<view v-for="goods in activeGoods" :key="goods.id" class="goods-item" @click="openGoods(goods)">
							<image class="goods-img" :src="goods.image" mode="aspectFill" />
							<view class="goods-body">
								<text class="goods-name">{{ goods.title }}</text>
								<view class="goods-footer">
									<text class="goods-price">¥{{ goods.price }}</text>
									<uni-tag v-if="goods.tag" :text="goods.tag" type="primary" size="mini" inverted></uni-tag>
								</view>
							</view>
						</view>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script setup>
	import { computed, nextTick, ref } from 'vue'

	const activeCategoryIndex = ref(0)
	const rightScrollTop = ref(0)
	const rightScrollTopSnapshot = ref(0)

	const categoryList = ref([{
		id: 'c-phone',
		name: '手机数码',
		children: [{
			id: 's-phone',
			name: '手机',
			icon: '/static/c1.png'
		}, {
			id: 's-earphone',
			name: '耳机',
			icon: '/static/c2.png'
		}, {
			id: 's-watch',
			name: '手表',
			icon: '/static/c3.png'
		}, {
			id: 's-accessory',
			name: '配件',
			icon: '/static/c4.png'
		}]
	}, {
		id: 'c-computer',
		name: '电脑办公',
		children: [{
			id: 's-laptop',
			name: '笔记本',
			icon: '/static/c5.png'
		}, {
			id: 's-keyboard',
			name: '键盘',
			icon: '/static/c6.png'
		}, {
			id: 's-mouse',
			name: '鼠标',
			icon: '/static/c7.png'
		}, {
			id: 's-monitor',
			name: '显示器',
			icon: '/static/c8.png'
		}]
	}, {
		id: 'c-appliance',
		name: '家用电器',
		children: [{
			id: 's-kitchen',
			name: '厨房电器',
			icon: '/static/c9.png'
		}, {
			id: 's-clean',
			name: '清洁电器',
			icon: '/static/logo.png'
		}, {
			id: 's-air',
			name: '空气净化',
			icon: '/static/uni.png'
		}, {
			id: 's-other',
			name: '其他',
			icon: '/static/c1.png'
		}]
	}, {
		id: 'c-food',
		name: '食品饮料',
		children: [{
			id: 's-snack',
			name: '零食',
			icon: '/static/c2.png'
		}, {
			id: 's-drink',
			name: '饮料',
			icon: '/static/c3.png'
		}, {
			id: 's-tea',
			name: '茶饮',
			icon: '/static/c4.png'
		}, {
			id: 's-coffee',
			name: '咖啡',
			icon: '/static/c5.png'
		}]
	}, {
		id: 'c-clothes',
		name: '服饰箱包',
		children: [{
			id: 's-tshirt',
			name: '上衣',
			icon: '/static/c6.png'
		}, {
			id: 's-pants',
			name: '裤装',
			icon: '/static/c7.png'
		}, {
			id: 's-shoes',
			name: '鞋靴',
			icon: '/static/c8.png'
		}, {
			id: 's-bag',
			name: '箱包',
			icon: '/static/c9.png'
		}]
	}])

	const goodsList = ref([{
		id: 'g1',
		categoryId: 'c-phone',
		title: '旗舰手机 12GB+256GB',
		price: '3999.00',
		image: '/static/c4.png',
		tag: '新品'
	}, {
		id: 'g2',
		categoryId: 'c-phone',
		title: '真无线降噪耳机',
		price: '499.00',
		image: '/static/c8.png',
		tag: ''
	}, {
		id: 'g3',
		categoryId: 'c-computer',
		title: '轻薄笔记本 i7/16G/1T',
		price: '6299.00',
		image: '/static/c5.png',
		tag: '热卖'
	}, {
		id: 'g4',
		categoryId: 'c-computer',
		title: '机械键盘 RGB 104 键',
		price: '199.00',
		image: '/static/c9.png',
		tag: ''
	}, {
		id: 'g5',
		categoryId: 'c-appliance',
		title: '智能空气炸锅 5L 大容量',
		price: '299.00',
		image: '/static/c6.png',
		tag: '优惠'
	}, {
		id: 'g6',
		categoryId: 'c-food',
		title: '坚果礼盒 混合口味 30 包',
		price: '89.00',
		image: '/static/c7.png',
		tag: ''
	}, {
		id: 'g7',
		categoryId: 'c-clothes',
		title: '经典白色短袖 T 恤',
		price: '59.00',
		image: '/static/c1.png',
		tag: ''
	}])

	const activeCategory = computed(() => categoryList.value[activeCategoryIndex.value])
	const activeSubCategories = computed(() => activeCategory.value?.children ?? [])
	const activeGoods = computed(() => {
		const id = activeCategory.value?.id
		if (!id) return []
		return goodsList.value.filter(item => item.categoryId === id)
	})

	function selectCategory(index) {
		activeCategoryIndex.value = index
		resetRightScroll()
	}

	function onRightScroll(e) {
		rightScrollTopSnapshot.value = e?.detail?.scrollTop ?? 0
	}

	function resetRightScroll() {
		rightScrollTop.value = rightScrollTopSnapshot.value
		nextTick(() => {
			rightScrollTop.value = 0
			rightScrollTopSnapshot.value = 0
		})
	}

	function openSubCategory(item) {
		uni.showToast({
			title: item.name,
			icon: 'none'
		})
	}

	function openGoods(goods) {
		uni.navigateTo({
			url: `/pages/goods-detail/goods-detail?id=${goods.id}`
		})
	}
</script>

<style scoped>
	.page {
		height: calc(100vh - var(--window-bottom) - var(--window-top));
		background-color: #f5f5f5;
	}

	.layout {
		height: 100%;
		display: flex;
	}

	.left {
		width: 200rpx;
		background-color: #ffffff;
	}

	.left-inner {
		padding: 16rpx 0;
	}

	.left-item {
		padding: 26rpx 16rpx;
		position: relative;
	}

	.left-item--active {
		background-color: #f5f5f5;
	}

	.left-item--active::before {
		content: '';
		position: absolute;
		left: 0;
		top: 24rpx;
		bottom: 24rpx;
		width: 8rpx;
		background-color: #2979ff;
		border-radius: 9999rpx;
	}

	.left-text {
		font-size: 28rpx;
		color: #666666;
		line-height: 40rpx;
	}

	.left-text--active {
		color: #333333;
		font-weight: 600;
	}

	.right {
		flex: 1;
	}

	.right-inner {
		padding: 24rpx;
	}

	.right-header {
		margin-bottom: 16rpx;
	}

	.right-title {
		font-size: 32rpx;
		color: #333333;
		font-weight: 700;
		line-height: 44rpx;
	}

	.right-subtitle {
		margin-top: 8rpx;
		display: block;
		font-size: 24rpx;
		color: #999999;
		line-height: 34rpx;
	}

	.subcats {
		background-color: #ffffff;
		border-radius: 24rpx;
		overflow: hidden;
		padding: 8rpx 0;
	}

	.subcat-item {
		padding: 16rpx 0;
		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;
	}

	.subcat-icon {
		width: 90rpx;
		height: 90rpx;
		border-radius: 18rpx;
		background-color: #f5f5f5;
	}

	.subcat-name {
		margin-top: 12rpx;
		font-size: 24rpx;
		color: #333333;
		line-height: 34rpx;
	}

	.goods-header {
		margin-top: 24rpx;
		margin-bottom: 16rpx;
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.goods-title {
		font-size: 30rpx;
		color: #333333;
		font-weight: 700;
	}

	.goods-count {
		font-size: 24rpx;
		color: #999999;
	}

	.goods-list {
		display: flex;
		flex-direction: column;
	}

	.goods-item {
		display: flex;
		background-color: #ffffff;
		border-radius: 24rpx;
		overflow: hidden;
		margin-bottom: 24rpx;
	}

	.goods-img {
		width: 180rpx;
		height: 180rpx;
		flex-shrink: 0;
	}

	.goods-body {
		flex: 1;
		padding: 20rpx;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
	}

	.goods-name {
		display: block;
		font-size: 28rpx;
		color: #333333;
		line-height: 40rpx;
		height: 80rpx;
		overflow: hidden;
	}

	.goods-footer {
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.goods-price {
		font-size: 30rpx;
		color: #e64340;
		font-weight: 600;
	}
</style>
