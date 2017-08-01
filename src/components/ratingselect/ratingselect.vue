<template>
	<div class="ratingselect">
		<div class="rating-type">
			<span class="block positive" :class="{'active': selectType===2}" @click="select(2)">{{ desc.all }}<span class="count">{{ ratings.length }}</span></span>
			<span class="block positive" :class="{'active': selectType===0}" @click="select(0)">{{ desc.positive }}<span class="count">{{ positives.length }}</span></span>
			<span class="block negative" :class="{'active': selectType===1}" @click="select(1)">{{ desc.negative }}<span class="count">{{ negatives.length }}</span></span>
		</div>
		<div class="switch" :class="{'on': onlyContent}" @click="toggleContent">
			<span class="icon-check_circle"></span>
			<span class="text">只看有内容的评价</span>
		</div>
	</div>
</template>
<script>
	const POSITIVE = 0
	const NEGATIVE = 1
	const ALL = 2
	export default {
		props: {
			ratings: {
				type: Array,
				default() {
					return []
				}
			},
			selectType: {
				type: Number,
				default: ALL
			},
			onlyContent: {
				type: Boolean,
				default: true
			},
			desc: {
				type: Object,
				default() {
					return {
						all: '全部',
						positive: '满意',
						negative: '不满意'
					}
				}
			}
		},
		methods: {
			select(type) {
				if (!event._constructed) {
					return
				}
				this.$emit('select', type)
			},
			toggleContent() {
				if (!event._constructed) {
					return
				}
				this.$emit('toggle', this.onlyContent)
			}
		},
		computed: {
			positives() {
				return this.ratings.filter((rating) => {
					return rating.rateType === POSITIVE
				})
			},
			negatives() {
				return this.ratings.filter((rating) => {
					return rating.rateType === NEGATIVE
				})
			}
		}
	}
</script>
<style lang="stylus" rel="text/stylus">
	@import "../../common/stylus/mixin.styl"

	.ratingselect
		.rating-type
			padding: 18px 0
			margin: 0 18px
			font-size: 0
			border-1px(rgba(7, 17, 27, .1))
			.block
				display: inline-block
				line-height: 12px
				padding: 8px 12px
				margin-right: 8px
				border-radius: 1px
				color: rgb(77, 85, 93)
				font-size: 12px
				font-weight: 700
				&.active
					color: #fff
				&.positive
					background: rgba(0, 160, 220, .2)
					&.active
						background: rgb(0, 160, 220)
				&.negative
					background: rgba(77, 85, 93, .2)
					&.active
						background: rgb(77, 85, 93)
				.count
					margin-left: 2px
					font-size: 8px
		.switch
			padding: 12px 18px
			line-height: 24px
			border-bottom: 1px solid rgba(7, 17, 27, .1)
			color: rgb(147, 153, 159)
			font-size: 12px
			&.on
				.icon-check_circle
					color: #00c850
			.icon-check_circle
				display: inline-block
				vertical-align: top
				font-size: 24px
				margin-right: 4px
			.text
				font-size: 12px
				
</style>