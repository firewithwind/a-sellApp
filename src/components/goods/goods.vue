<template>
	<div class="goods">
		<div class="menu-wrapper" ref="menuWrapper">
			<ul>
				<li v-for="item,index in goods" class="menu-item" :class="{'current': currentIndex === index}" @click="selectMenu(index)">
					<span class="text border-1px">
						<span class="icon" v-if="item.type>0" :class="classMap[item.type]"></span>
						{{ item.name }}
					</span>
				</li>
			</ul>
		</div>
		<div class="foods-wrapper" ref="foodsWrapper">
			<ul>
				<li v-for="item in goods" class="food-list food-list-hook">
					<h1 class="title">{{ item.name }}</h1>
					<ul>
						<li v-for="food in item.foods" class="food-item border-1px" @click="selectFood(food)">
							<div class="icon">
								<img :src="food.icon" width="57" height="57">
							</div>
							<div class="content">
								<h2 class="name">{{ food.name }}</h2>
								<p class="desc">{{ food.description }}</p>
								<div class="extra">
									<span class="count">月售{{ food.sellCount }}份</span><span>好评率{{ food.rating }}%</span>
								</div>
								<div class="price">
									<span class="now">￥{{ food.price }}</span>
									<span class="old" v-show="food.oldPrice">￥{{ food.oldPrice }}</span>
								</div>
								<div class="cartcontrol-wrapper">
									<cartcontrol @add="addFood" :food="food"></cartcontrol>
								</div>
							</div>
						</li>
					</ul>
				</li>
			</ul>
		</div>
		<shopcart ref="shopcart" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice" :select-foods="selectFoods" @add="addFood"></shopcart>
		<food :food="selectedFood" ref="food" @add="addFood"></food>
	</div>
</template>
<script>
	import BScroll from 'better-scroll'
	import shopcart from 'components/shopcart/shopcart'
	import cartcontrol from 'components/cartcontrol/cartcontrol'
	import food from 'components/food/food'

	const REE_OK = 0

	export default {
		props: {
			seller: {
				type: Object
			}
		},
		data() {
			return {
				goods: [],
				listHeight: [],
				scrollY: 0,
				selectedFood: {}
			}
		},
		computed: {
			currentIndex() {
				for (let i = 0; i < this.listHeight.length; i++) {
					let height = this.listHeight[i]
					let height2 = this.listHeight[i + 1]
					if (!height2 || (this.scrollY >= height && this.scrollY < height2)) {
						return i
					}
				}
				return 0
			},
			selectFoods() {
				let foods = []
				this.goods.forEach((good) => {
					good.foods.forEach((food) => {
						if (food.count) {
							foods.push(food)
						}
					})
				})
				return foods
			}
		},
		created() {
			this.classMap = ['decrease', 'discount', 'guarantee', 'invoice', 'special']
			this.$http.get('/api/goods').then((res) => {
				res = res.body
				if (res.errno === REE_OK) {
					this.goods = res.data
					this.$nextTick(() => {
						this._initScroll()
						this._calculateHeight()
					})
				}
			})
		},
		methods: {
			_initScroll() {
				this.menuScroll = new BScroll(this.$refs.menuWrapper, {
					click: true
				})
				this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
					probeType: 3,
					click: true
				})
				this.foodsScroll.on('scroll', (pos) => {
					this.scrollY = Math.abs(Math.round(pos.y))
				})
			},
			_calculateHeight() {
				let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
				let height = 0
				this.listHeight.push(height)
				for (let i = 0; i < foodList.length; i++) {
					let item = foodList[i]
					height += item.clientHeight
					this.listHeight.push(height)
				}
				this.listHeight[2] = 1190
			},
			selectMenu(index) {
				if (!event._constructed) {
					return
				}
				let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
				let el = foodList[index]
				this.foodsScroll.scrollToElement(el, 10)
			},
			selectFood(food) {
				if (!event._constructed) {
					return
				}
				this.selectedFood = food
				this.$refs.food.show()
			},
			addFood(target) {
				this._drop(target)
			},
			_drop(target) {
				// 体验优化，异步执行小球落下
				this.$nextTick(() => {
					this.$refs.shopcart.drop(target)
				})
			}
		},
		components: {
			shopcart,
			cartcontrol,
			food
		}
	}
</script>
<style lang="stylus" rel="text/stylus">
	@import '../../common/stylus/mixin.styl'
	.goods
		display: flex
		position: absolute
		top: 174px
		bottom: 46px
		width: 100%
		overflow: hidden
		.menu-wrapper
			flex: 0 0 80px
			width: 80px
			background: #f3f5f7
			.menu-item
				display: table
				height: 54px
				width: 56px
				line-height: 14px
				padding: 0 12px
				&.current
					position: relative
					z-index: 10
					margin-top: -1px
					background: #fff
					font-weight: 700
					.text
						border-none()
				.icon
					display: inline-block
					vertical-align: top
					width: 12px
					height: 12px
					margin-right: 2px
					background-size: 12px 12px
					background-repeat: no-repeat
					&.decrease
						bg-image('decrease_3')
					&.discount
						bg-image('discount_3')
					&.guarantee
						bg-image('guarantee_3')
					&.invoice
						bg-image('invoice_3')
					&.special
						bg-image('special_3')
				.text
					display: table-cell
					font-size: 12px
					vertical-align: middle
					width: 56px
					border-1px(rgba(7, 17, 27, .1))
		.foods-wrapper
			flex: 1
			.title
				padding-left: 14px
				height: 26px
				line-height: 26px
				border-left: 2px solid #d9dd1
				font-size: 12px
				color: rgb(147, 153, 159)
				background: #f3f5f7
			.food-item
				display: flex
				margin: 18px
				padding-bottom: 18px
				border-1px(rgba(7, 17, 27, .1))
				&:last-child
					border-none()
					margin-bottom: 0
				.icon
					flex: 0 0 57px
					margin-right: 10px
				.content
					flex: 1
					.name
						margin: 2px 0 8px 0
						height: 14px
						line-height: 14px
						font-size: 14px
						color: rgb(7, 17, 27)
					.desc, .extra
						line-height: 10px
						font-size: 10px
						color: rgb(147, 153, 159)
					.desc
						line-height: 12px
						margin-bottom: 8px
					.extra
						.count
							margin-right: 12px
					.price
						font-weight: 700
						line-height: 24px
						.now
							margin-right: 8px
							font-size: 14px
							color: rgb(240, 20, 20)
						.old
							text-decoration: line-through
							font-size: 10px
							color: rgb(147, 153, 159)
					.cartcontrol-wrapper
						position: absolute
						bottom: 12px
						right: 0
</style>