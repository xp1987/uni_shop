<template>
	<view>
		<!-- 搜索组件 -->
		<view class="search-box">
			<my-search @click="gotoSearch"></my-search>
		</view>
		
		<!-- 轮播图区域 -->
		<swiper :indicator-dots="true" 
				:autoplay="true" 
				:interval="3000"
				:duration="1000"
				:circular="true">
				<!-- 循环渲染轮播图item项 -->
				<swiper-item v-for="(item, index) in swiperList" :key="index">
					<navigator class="swiper-item"
							   :url="'/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id">
						<!-- 动态绑定图片 src 属性 -->
						<image :src="item.image_src"></image>
					</navigator>>
				</swiper-item>
		</swiper>
		
		<!-- 分类导航区域 -->
		<view class="nav-list">
			<view class="nav-item" v-for="(item, index) in navList" :key="index" @click="navClickHandler(item)">
				<image :src="item.image_src" class="nav-img"></image>
			</view>
		</view>
		
		<!-- 楼层区域 -->
		<view class="floor-list">
			<!-- 楼层 item 项 -->
			<view class="floor-item" v-for="(item, index) in floorList" :key="index">
				<!-- 楼层标题 -->
				<image :src="item.floor_title.image_src" class="floor-title"></image>
				<!-- 楼层图片区域 -->
				<view class="floor-img-box">
					<!-- 左侧大图片盒子 -->
					<navigator class="left-img-box" :url="item.product_list[0].url">
						<image :src="item.product_list[0].image_src"
							   :style="{ width: item.product_list[0].image_width + 'rpx' }"
							   mode="widthFix"></image>
					</navigator>
					<!-- 右侧4个小图片盒子 -->
					<view class="right-img-box">
						<navigator class="right-img-item" 
											 v-for="(item2, index2) in item.product_list" 
											 :key="index2" 
											 v-if="index2 !== 0"
											 :url="item2.url">
							<image :src="item2.image_src" mode="widthFix" :style="{ width: item2.image_width + 'rpx' }"></image>
						</navigator>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	// 导入自己封装的 mixin 模块
	import badgeMix from '@/mixins/tabbar-badge.js'
	export default {
		// 将 badgeMix 混入到当前的页面中进行使用
		mixins: [badgeMix],
		
		data() {
			return {
				// 轮播图数据列表，默认空数组
				swiperList: [],
				
				// 分类导航数据列表
				navList: [],
				
				// 楼层数据列表
				floorList: []
			};
		},
		onLoad() {
			// 调用获取轮播图方法
			this.getSwiperList()
			// 调用获取分类导航方法
			this.getNavList()
			// 调用获取楼层数据方法
			this.getFloorList()
		},
		methods: {
			// 获取轮播图数据方法
			async getSwiperList() {
				const {data: res} = await uni.$http.get('/api/public/v1/home/swiperdata')
				// 请求失败
				if(res.meta.status !== 200) return uni.$showMsg()
				// 请求成功
				this.swiperList = res.message
			},
			
			// 获取分类导航数据方法
			async getNavList() {
				const { data: res } = await uni.$http.get('/api/public/v1/home/catitems')
				// console.log(res)
				if(res.meta.status !== 200) return uni.$showMsg()
				this.navList = res.message
			},
			
			// nav-item 分类点击事件
			navClickHandler(item) {
				if(item.name === '分类') {
					uni.switchTab({
						url: '/pages/cate/cate'
					})
				}
			},
			
			// 获取楼层列表数据方法
			async getFloorList() {
				const { data: res } = await uni.$http.get('/api/public/v1/home/floordata')
				// console.log(res)
				if(res.meta.status !== 200) return uni.$showMsg()
				
				// 通过双层 forEach 循环 处理 URL 地址
				res.message.forEach(floor => {
					floor.product_list.forEach(prod => {
						prod.url = '/subpkg/goods_list/goods_list?' + prod.navigator_url.split('?')[1]
					})
				})
				
				this.floorList = res.message
				
			},
			
			// 导航到 search
			gotoSearch() {
				uni.navigateTo({
					url: '/subpkg/search/search'
				})
			}
		}
	}
</script>

<style lang="scss">
	swiper {
		height: 330rpx;
		.swiper-item, image {
			width: 100%;
			height: 100%;
		}
	}
	
	.nav-list {
		display: flex;
		justify-content: space-around;
		margin: 15px 0;
		.nav-img {
			width: 128rpx;
			height: 140rpx;
		}
	}
	
	.floor-title {
		height: 60rpx;
		width: 100%;
		display: flex;
	}
	
	.right-img-box {
		display: flex;
		flex-wrap: wrap;
		justify-content: space-around;
	}
	.floor-img-box {
		display: flex;
		padding-left: 10rpx;
	}
	
	.search-box {
		position: sticky;
		top: 0;
		z-index: 999;
	}
</style>
