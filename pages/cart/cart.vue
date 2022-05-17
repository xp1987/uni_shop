<template>
	<view class="cart-container" v-if="cart.length !== 0">
		<!-- 收货地址 -->
		<my-address></my-address>
		
		<!-- 购物车商品列表标题区域 -->
		<view class="cart-title">
			<!-- 左侧图标 -->
			<uni-icons type="shop" size="18"></uni-icons>
			<!-- 描述文本 -->
			<text class="cart-title-text">购物车</text>
		</view>
		
		<!-- 商品列表区域 -->
		<!-- uni-swipe-action 是最外层包裹性质的容器 -->
		<uni-swipe-action>
		  <block v-for="(goods, i) in cart" :key="i">
		    <!-- uni-swipe-action-item 可以为其子节点提供滑动操作的效果。需要通过 options 属性来指定操作按钮的配置信息 -->
		    <uni-swipe-action-item :options="options" @click="swipeActionClickHandler(goods)">
		      <my-goods :goods="goods" :show-radio="true" :show-num="true" @radio-change="radioChangeHandler" @num-change="numberChangeHandler"></my-goods>
		    </uni-swipe-action-item>
		  </block>
		</uni-swipe-action>
		
		<!-- 结算组件 -->
		<my-settle></my-settle>
	</view>
	
	<!-- 空白购物车时 -->
	<view class="empty-cart" v-else>
		<image src="../../static/tab_icons/cart_null.png" class="empty-img"></image>
		<text class="tip-text">空空如也~~~</text>
	</view>
</template>

<script>
	import badgeMix from '@/mixins/tabbar-badge.js'
	// 按需导入 mapState 这个辅助函数
	import { mapState, mapMutations } from 'vuex'
	
	export default {
		
		mixins: [badgeMix],
		
		data() {
			return {
				options: [{
				  text: '删除', // 显示的文本内容
				  style: {
				    backgroundColor: '#ff4400' // 按钮的背景颜色
				  }
				}]
			};
		},
		
		methods: {
			...mapMutations('m_cart', ['updateGoodsState']),
			
			...mapMutations('m_cart', ['updateGoodsState', 'updateGoodsCount']),
			
			...mapMutations('m_cart', ['updateGoodsState', 'updateGoodsCount', 'removeGoodsById']),
			
			// 商品的勾选状态发生了变化
			radioChangeHandler(e) {
				// console.log(e)
				this.updateGoodsState(e)
			},
			
			// 商品数量发生了变化
			numberChangeHandler(e) {
				this.updateGoodsCount(e)
			},
			
			// 点击了滑动操作按钮
			swipeActionClickHandler(goods) {
				console.log(123)
			  this.removeGoodsById(goods.goods_id)
			}
		},
		
		computed: {
			// 将 m_cart 模块中的 cart 数组映射到当前页面中使用
			...mapState('m_cart', ['cart'])
		}
		
	}
</script>

<style lang="scss">
	// 购物车标题列表样式
	.cart-title {
		height: 40px;
		display: flex;
		align-items: center;
		font-size: 14px;
		padding-left: 5px;
		border-bottom: 1px solid #efefef;
		
		.cart-title-text {
			margin-left: 10px;
		}
	}
	
	.cart-container {
		padding-bottom: 50px;
	}
	
	// 空白购物车
	.empty-cart {
		display: flex;
		flex-direction: column;
		align-items: center;
		padding-top: 150px;
		
		.empty-img {
			width: 90px;
			height: 90px;
		}
		
		.tip-text {
			font-size: 12px;
			color: gray;
			margin-top: 15px;
			
		}
	}
</style>
