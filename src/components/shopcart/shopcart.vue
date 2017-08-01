<template>
	<div>
		<div class="shopcart">
			<div class="content" @click="toggleList">
				<div class="content-left">
					<div class="logo-wrapper">
						<div class="logo" :class="{'highlight': totalCount>0}">
							<span class="icon-shopping_cart"  :class="{'highlight': totalCount>0}"></span>	
						</div>
						<div class="num" v-show="totalCount>0">{{ totalCount }}</div>
					</div>
					<div class="price" :class="{'highlight': totalPrice>0}">￥{{ totalPrice }}</div>
					<div class="desc">另需配送费￥{{deliveryPrice}}元</div>
				</div>
				<div class="content-right" :class="{'highlight': totalPrice>=minPrice}" @click.stop="pay">
					<div class="pay">
						{{ payDesc }}
					</div>
				</div>
			</div>
			<div class="ball-container">
				<transition-group name="drop" @before-enter="beforeEnter" @enter="enter" @after-enter="afterEnter">
				<div class="ball" v-for="ball in balls" :key="ball.key" v-show="ball.show">
					<div class="inner inner-hook">
					</div>
				</div>
				</transition-group>
			</div>
			<transition name="fold">
			<div class="shopcart-list" v-show="listShow">
				<div class="list-header">
					<h1 class="title">购物车</h1>
					<span class="empty" @click="empty">清空</span>
				</div>
				<div class="list-content" ref="listContent">
					<ul>
						<li class="food" v-for="food in selectFoods">
							<span class="name">{{ food.name }}</span>
							<div class="price">
								<span>￥{{ food.price*food.count }}</span>
							</div>
							<div class="cartcontrol-wrapper">
								<cartcontrol :food="food" @add="addFood"></cartcontrol>
							</div>
						</li>
					</ul>
				</div>
			</div>
			</transition>
		</div>
		<transition name="fade">
		<div class="list-mask" v-show="listShow" @click.stop="hideList"></div>
		</transition>
	</div>
</template>
<script>
	import BScroll from 'better-scroll'
	import cartcontrol from 'components/cartcontrol/cartcontrol'

	export default {
		props: {
			deliveryPrice: {
				type: Number,
				default: 0
			},
			minPrice: {
				type: Number,
				default: 0
			},
			selectFoods: {
				type: Array,
				default() {
					return []
				}
			}
		},
		data() {
			return {
				balls: [
					{
						key: 'b1',
						show: false
					},
					{
						key: 'b2',
						show: false
					},
					{
						key: 'b3',
						show: false
					},
					{
						key: 'b4',
						show: false
					},
					{
						key: 'b5',
						show: false
					}
				],
				dropBalls: [],
				fold: true
			}
		},
		methods: {
			addFood(target) {
				this.$emit('add', target)
			},
			pay() {
				if (this.totalPrice < this.minPrice) {
					return
				}
				window.alert('支付' + this.totalPrice + '元')
			},
			hideList() {
				this.fold = true
			},
			empty() {
				this.selectFoods.forEach((food) => {
					food.count = 0
				})
			},
			toggleList() {
				if (!this.totalCount) {
					return
				}
				this.fold = !this.fold
			},
			drop(el) {
				for (let i = 0; i < this.balls.length; i++) {
					let ball = this.balls[i]
					if (!ball.show) {
						ball.show = true
						ball.el = el
						this.dropBalls.push(ball)
						return
					}
				}
			},
			beforeEnter(el) {
				let count = this.balls.length
				while (count--) {
					let ball = this.balls[count]
					if (ball.show) {
						let rect = ball.el.getBoundingClientRect()
            let x = rect.left - 32
            // let y = -(window.innerHeight - rect.top - 22)
            let y = -rect.top
            el.style.display = ''
            el.style.webkitTransform = `translate3d(0, ${y}px, 0)`
            el.style.transform = `translate3d(0, ${y}px, 0)`
            let inner = el.getElementsByClassName('inner-hook')[0]
            inner.style.webkitTransform = `translate3d(${x}px, 0, 0)`
            inner.style.transform = `translate3d(${x}px, 0, 0)`
					}
				}
			},
			enter(el, done) {
				/* eslint-disable no-unused-vars */
				let rf = el.offsetHeight
				this.$nextTick(() => {
					el.style.webkitTransform = 'translate3d(0, 0, 0)'
					el.style.transform = 'translate3d(0, 0, 0)'
					let inner = el.getElementsByClassName('inner-hook')[0]
					inner.style.webkitTransform = 'translate3d(0, 0, 0)'
					inner.style.transform = 'translate3d(0, 0, 0)'
					el.addEventListener('transitionend', done)
				})
			},
			afterEnter(el) {
				let ball = this.dropBalls.shift()
				if (ball) {
					setTimeout(() => {
						ball.show = false
						el.style.display = 'none'
					}, 400)
				}
			}
		},
		computed: {
			totalPrice() {
				let total = 0
				this.selectFoods.forEach((food) => {
					total += food.price * food.count
				})
				return total
			},
			totalCount() {
				let count = 0
				this.selectFoods.forEach((food) => {
					count += food.count
				})
				return count
			},
			payDesc() {
				if (this.totalPrice === 0) {
					return `￥${this.minPrice}元起送`
				} else if (this.totalPrice < this.minPrice) {
					let diff = this.minPrice - this.totalPrice
					return `还差￥${diff}元起送`
				} else {
					return '去结算'
				}
			},
			listShow() {
				if (!this.totalCount) {
					this.fold = true
					return false
				}
				let show = !this.fold
				if (show) {
					this.$nextTick(() => {
						if (!this.scroll) {
							this.scroll = new BScroll(this.$refs.listContent, {
								click: true
							})
						} else {
							this.scroll.refresh()
						}
					})
				}
				return show
			}
		},
		components: {
			cartcontrol
		}
	}
</script>
<style lang="stylus" rel="text/stylus">
	@import "../../common/stylus/mixin.styl"
	
	.fade-enter-active, .fade-leave-active
		transition: all .4s
	.fade-enter, .fade-leave-to
		opacity: 0
	.fold-enter-active, .fold-leave-active
		transition: all .4s linear
	.fold-enter, .fold-leave-to
		transform: translate3d(0, 100%, 0)
	.fold-enter-to, fold-leave
		transform: translate3d(0, 0, 0)
	.shopcart
		position: fixed
		left: 0
		bottom: 0
		z-index: 50
		width: 100%
		height: 48px
		background: #000
		.content
			display: flex
			background: #141d27
			color: rgba(255, 255, 255, .4)
			.content-left
				flex: 1
				.logo-wrapper
					display: inline-block
					position: relative
					top: -10px
					margin: 0 12px
					padding: 6px
					width: 56px
					height: 56px
					box-sizing: border-box
					vertical-align: top
					border-radius: 50%
					background: #141d27
					.logo
						width: 100%
						height: 100%
						border-radius: 50%
						background: rgba(255, 255, 255, .2)
						text-align: center
						&.highlight
							background: rgb(0, 160, 220)
						.icon-shopping_cart
							font-size: 24px
							color: #80858a
							line-height: 44px
							&.highlight
								color: #fff
					.num
						position: absolute
						top: 0
						right: 0
						width: 24px
						height: 16px
						line-height: 16px
						border-radius: 16px
						text-align: center
						font-size: 9px
						font-weight: 700
						color: #fff
						background: rgb(240, 20, 20)
						box-shadow: 0 4px 8px 0 rgba(0, 0, 0, .4)
				.price
					display: inline-block
					vertical-align: top
					margin-top: 12px
					line-height: 24px
					padding-right: 12px
					box-sizing: border-box
					border-right: 1px solid rgba(255, 255, 255, .1)
					font-size: 16px
					font-weight: 700
					&.highlight
						color: #fff
				.desc
					display: inline-block
					vertical-align: top
					line-height: 24px
					margin: 12px 0 0 12px
					font-size: 10px
			.content-right
				flex: 0 0 105px
				width: 105px
				background: rgba(255, 255, 255, .2)
				&.highlight
					background: #00b43c
					color: #fff
				.pay
					line-height: 48px
					height: 48px
					text-align: center
					font-size: 12px
					font-weight: 700
		.ball-container
			.ball
				position: fixed
				left: 32px
				bottom: 22px
				z-index: 200
				transition: all .4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
				.inner
					width: 16px
					height: 16px
					border-radius: 50%
					background: rgb(0, 160, 220)
					transition: all .4s linear
		.shopcart-list
			position: absolute
			bottom: 48px
			left: 0
			z-index: -1
			width: 100% 
			.list-header
				height: 40px
				line-height: 40px
				padding: 0 18px
				background: #f3f5f7
				border-bottom: 1px solid rgba(7, 17, 21, 0.1)
				.title
					float: left
					font-size: 14px
					color: rgb(7, 17, 27)
				.empty
					float: right
					font-size: 12px
					color: rgb(0, 160, 220)
			.list-content
				padding: 0px 18px 40px 18px
				max-height: 217px
				background: #fff
				overflow: hidden
				.food
					position: relative
					padding: 12px 0
					box-sizing: border-box
					border-1px(rgba(7, 17, 21, 0.1))
					.name
						line-height: 24px
						font-size: 14px
						color: rgb(7, 17, 27)
					.price
						position: absolute
						right: 90px
						bottom: 12px
						line-height: 24px
						font-size: 14px
						font-weight: 700
						color: rgb(240, 20, 20)
					.cartcontrol-wrapper
						position: absolute
						right: 0
						bottom: 6px
	.list-mask
		position: fixed
		top: 0
		left: 0
		width: 100%
		height: 100%
		z-index: 40
		backdrop-filter: blur(10px)
		background: rgba(7, 17, 27, .6)
</style>