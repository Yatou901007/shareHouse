<template>
	<view class="zx-sorts">
		<view class="zx-sorts-item" @click="sortBy(list)" v-for="(list,index) in sortParam" :key="index">
			<text :class="(list.up||list.down)?'item-active':''">{{list.title}}</text>	
			<view class="sort-icon">
				<uni-icon class="up" :type="upIcon" :color="list.up?'#b22222':'#000'" size="10" />
				<uni-icon class="down" :type="downIcon" :color="list.down?'#b22222':'#000'" size="10" />
			</view>
		</view>
	</view>
</template>

<script>
	import uniIcon from '@/components/uni-icon/uni-icon.vue'
	export default {
		name: 'zxSorts',
		components: {
			uniIcon
		},
		props: {
			sortlist: {
				type: Array,
				default () {
					return [];
				}
			}
		},
		data() {
			return {
				sortParam: [],
				upIcon: 'arrowup',
				downIcon: 'arrowdown'
			}
		},
		created() {
			for (var i = 0; i < this.sortlist.length; ++i) {
				this.sortlist[i].down = false
				this.sortlist[i].up = false
			}
			// 初始化排序列表
			this.sortParam = JSON.parse(JSON.stringify(this.sortlist))
		},
		methods: {
			sortBy (item) {
				if (item.down) {
					item.down = false
					item.up = true
				} else {
					item.down = true
					item.up = false
				}
				for (var i = 0; i < this.sortParam.length; ++i) {
					if (item.field != this.sortParam[i].field) {
						this.sortParam[i].down = false
						this.sortParam[i].up = false
					}
				}
				let resultSort = {
					SortType: item.down ? 'desc' : 'asc',
					SortField: item.key
				}
				this.$emit("result", resultSort)
				this.sortParam = JSON.parse(JSON.stringify(this.sortParam))
			}
		},
		watch: {
			sortlist(newValue, oldValue) {
				this.sortParam = JSON.parse(JSON.stringify(newValue))
			}
		}
	}
</script>

<style lang="scss">
	@charset "UTF-8";

	.zx-sorts {
		position:fixed;
		height: 50rpx;
		top: 80rpx;
		left: 0;
		width:100vw;
		z-index:99;
		display: flex;
		align-items: center;
		background:#fff;
		font-size: 24rpx;
		padding:10rpx;
		border-bottom:1rpx solid #ccc;
		.zx-sorts-item {
			text-align: center;
			flex: 1;
			.item-active {
				color: #b22222;
			}
			.sort-icon {
				display: inline-block;
				position: relative;
				width:30rpx;
				height: 30rpx;
				display:inline-block;
				uni-icon {
					position: absolute;
					left: 0;
					width:30rpx;
					height: 30rpx;
					display:inline-block;
					&.up {
						top: -4rpx;
					}
					&.down {
						top: 8rpx;
					}
				}
			}
		}
	}
</style>