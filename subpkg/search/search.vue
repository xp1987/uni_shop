<template>
	<view>
		<!-- 顶部搜索框 -->
		<view class="search-box">
			<uni-search-bar @input="input" 
											:radius="100"
											cancelButton="none"
											:show='show'
											:showSync='showSync'
											>
			</uni-search-bar>
		</view>		
		
		<!-- 搜索建议列表 -->
		<view class="sugg-list" v-if="searchResults.length !== 0">
			<view class="sugg-item" v-for="(item, index) in searchResults" 
				:key="index" 
				@click="gotoDetail(item.goods_id)">
				<view class="goods-name">{{ item.goods_name }}</view>
				<uni-icons type="arrowright" size="16"></uni-icons>
			</view>
		</view>
		
		<!-- 搜索历史 -->
		<view class="history-box" v-else>
			<!-- 标题区域 -->
			<view class="history-title">
				<text>搜索历史</text>
				<uni-icons type="trash" size="17" @click="cleanHistory"></uni-icons>
			</view>
			<!-- 列表区域 -->
			<view class="history-list">
				<uni-tag :text="item" v-for="(item, index) in historys" 
								 :key="index" @click="gotoGoodsList(item)"></uni-tag>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				show: true,
				showSync: true,
				searchVal: '',
				
				// 延时器的 timerID
				timer: null,
				
				// 搜索关键词
				kw: '',
				
				// 搜索结果列表
				searchResults: [],
				
				// 搜索关键字的历史记录 【假数据】
				historyList: ['a', 'app', 'apple']
			}
		},
		
		onLoad() {
			// 加载本地存储的搜索历史记录：
			this.historyList = JSON.parse(uni.getStorageSync('kw') || '[]')
		},
		
		computed: {
			historys() {
				// 注意：由于数组是引用类型，所以不要直接基于原数组调用 reverse 方法，以免修改原数组中元素的顺序
				// 而是应该新建一个内存无关的数组，再进行 reverse 反转
				return [...this.historyList].reverse()
			}
		},
		
		methods: {
			// input 输入事件处理函数
			input(e) {
				// 清除 timer 对应的延时器
				clearTimeout(this.timer)
				
				// 重新启动一个延时器 并把 timerID 赋值给 this.timer
				this.timer = setTimeout(() => {
					// 若500毫秒内 没有触发新的输入事件 则为搜索关键词赋值				
					this.kw = e
					// 根据关键词 查询搜索建议列表
					this.getSearchList()
				}, 500)
			},
			
			// 根据搜索关键词 搜索商品建议列表
			async getSearchList() {
				// 判断关键词是否为空
				if(this.kw === '') {
					this.searchResults = []
					return
				}
				// 发起请求 获取搜索建议列表
				const { data: res } = await uni.$http.get('/api/public/v1/goods/search', { query: this.kw })
				if(res.meta.status !== 200) return uni.$showMsg()
				this.searchResults = res.message.goods
				// console.log(this.searchResults)
				
				// 查询到搜索建议之后 调用 saveSearchHistory() 方法保存搜索关键词
				this.saveSearchHistory()
			},
			
			// 搜索建议的item项跳转到商品详情页面
			gotoDetail(goods_id) {
				uni.navigateTo({
					// 指定详情页面的 url 地址 并传递 goods_id 参数
					url: '/subpkg/goods_detail/goods_detail?good_id=' + goods_id
				})				
			},
			
			// 保存搜索关键词方法
			saveSearchHistory() {
				// 将 Array 数组转化为 Set 对象
				const set = new Set(this.historyList)
				// 调用 Set 对象的 delete 方法 移出对应的元素
				set.delete(this.kw)
				// 调用 Set 对象的 add 方法，向 Set 中添加元素
				set.add(this.kw)
				// 将 Set 对象转化为 Array 数组
				this.historyList = Array.from(set)
				// 调用 uni.setStorageSync(key, value) 将搜索历史记录持久化存储到本地
				uni.setStorageSync('kw', JSON.stringify(this.historyList))
			},
			
			// 清空搜索历史记录事件
			cleanHistory() {
				this.historyList = []
				// 清空本地存储中记录的搜索历史
				uni.setStorageSync('kw', '[]')
			},
			
			// 跳转到商品列表页面
			gotoGoodsList(kw) {
				uni.navigateTo({
					url: '/subpkg/goods_list/goods_list?query=' + kw
				})
			}
		}
	}
</script>

<style lang="scss">
	.uni-searchbar {
		background-color: #ff4400;
	}
	
	.search-box {
		position: sticky;
		top: 0;
		z-index: 999;
	}
	
	// 搜索建议列表样式
	.sugg-list {
		padding: 0 5px;
		
		.sugg-item {
			font-size: 12px;
			padding: 13px 0;
			border-bottom: 1px solid #efefef;
			display: flex;
			align-items: center;
			justify-content: space-between;
			
			.goods-name {
				// 文字不允许换行(单行文本)
				white-space: nowrap;
				// 溢出部分隐藏
				overflow: hidden;
				// 文本溢出后用...代替
				text-overflow: ellipsis;
				margin-right: 3px;
			}
		}
	}
	
	// 搜索历史区域样式
	.history-box {
		padding: 0 5px;
		
		.history-title {
			display: flex;
			justify-content: space-between;
			align-items: center;
			height: 40px;
			font-size: 13px;
			border-bottom: 1px solid #efefef;
		}
		
		.history-list {
			display: flex;
			flex-wrap: wrap;

			.uni-tag {
				margin-top: 5px;
				margin-right: 5px;
			}
		}
}
	
</style>
