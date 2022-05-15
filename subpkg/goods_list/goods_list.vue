<template>
	<view>
		<!-- 商品列表 -->
		<view class="goods-list">
			<view v-for="(item, index) in goodsList" :key="index" @click="gotoDetail(item)">
				<my-goods :goods="item"></my-goods>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				// 商铺列表请求参数
				queryObj: {
					// 查询关键词
					query: '',
					// 商品分类id
					cid: '',
					// 页码值
					pagenum: 1,
					// 每页显示多少条数据
					pagesize: 10
				},
				
				// 商品列表的数据
				goodsList: [],
				// 总数量 实现分页用于
				total: 0,
				
				// 是否正在请求数据 用于节流阀防止发起额外的请求
				isloading: false
			};
		},
		
		onLoad(options) {
			// 将页面参数转存到 this.queryObj 对象中
			this.queryObj.query = options.query || ''
			this.queryObj.cid = options.cid || ''
			
			// 调用获取商品列表数据方法
			this.getGoodsList()
		},
		
		methods: {
			// 获取商铺列表数据
			async getGoodsList(cb) {
				// 打开节流阀
				this.isloading = true
				const { data: res } = await uni.$http.get('/api/public/v1/goods/search', this.queryObj)
				// 关闭节流阀
				this.isloading = false
				// 只要数据请求完毕 就立即按需调用 cb 回调函数
				cb && cb()
				
				// 判断是否请求成功
				if (res.meta.status !== 200) return uni.$showMsg()
				// 为数据赋值
				this.goodsList = [...this.goodsList, ...res.message.goods]
				this.total = res.message.total
			},
			
			// 点击跳转到商品详情页面
			gotoDetail(item) {
				uni.navigateTo({
					url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
				})
			}
		},
		
		// 上拉触底事件函数
		onReachBottom() {
			
			// 当前的页码值 * 每页显示多少条数据 >= 总数条数
			// pagenum * pagesize >= total
			if (this.queryObj.pagenum * this.queryObj.pagesize >= this.total) return uni.$showMsg('数据加载完毕...')
			
			// 判断是否正在请求其它数据 若是 则不发起额外的请求
			if (this.isloading) return
			// 让页码自增 +1
			this.queryObj.pagenum += 1
			// 重新获取列表数据
			this.getGoodsList()
		},
		
		// 下拉刷新事件
		onPullDownRefresh() {
			// 重置关键数据
			this.queryObj.pagenum = 1
			this.total = 0
			this.isloading = false
			this.goodsList = []
			
			// 重新发送请求
			this.getGoodsList(() => uni.stopPullDownRefresh())
		},
		
	}
</script>

<style lang="scss">
	
</style>
