<template>
	<view class="uni-filters">
		<view class="uni-filters-item" @click="filterBy(list)" v-for="(list,index) in filterParam" :key="index">
			<text :class="(list.up||list.down)?'item-active':''">{{list.name}}</text>	
			<view class="filter-icon">
				<uni-icon class="up" :type="upIcon" :color="list.up?'#b22222':'#000'" size="10" />
				<uni-icon class="down" :type="downIcon" :color="list.down?'#b22222':'#000'" size="10" />
			</view>
		</view>
	</view>
</template>

<script>
	import uniIcon from '@/components/uni-icon/uni-icon.vue'
	export default {
		name: 'UniFilters',
		components: {
			uniIcon
		},
		props: {
			filterlist: {
				type: Array,
				default () {
					return [];
				}
			}
		},
		data() {
			return {
				filterParam: [],
				upIcon: 'arrowup',
				downIcon: 'arrowdown'
			}
		},
		created() {
			for (var i = 0; i < this.filterlist.length; ++i) {
				this.filterlist[i].down = false
				this.filterlist[i].up = false
			}
			// 初始化排序列表
			this.filterParam = JSON.parse(JSON.stringify(this.filterlist))
		},
		methods: {
			filterBy (item) {
				if (item.down) {
					item.down = false
					item.up = true
				} else {
					item.down = true
					item.up = false
				}
				for (var i = 0; i < this.filterParam.length; ++i) {
					if (item.field != this.filterParam[i].field) {
						this.filterParam[i].down = false
						this.filterParam[i].up = false
					}
				}
				this.filterParam = JSON.parse(JSON.stringify(this.filterParam))
			}
		},
		watch: {
			filterlist(newValue, oldValue) {
				this.filterParam = JSON.parse(JSON.stringify(newValue))
			}
		}
	}
</script>

<style lang="scss">
	@charset "UTF-8";

	.uni-filters {
		display: flex;
		align-items: center;
		background:#fff;
		padding:10rpx;
		font-weight:bold;
		border-bottom:1rpx solid #ccc;
		.uni-filters-item {
			text-align: center;
			flex: 1;
			.item-active {
				color: #b22222;
			}
			.filter-icon {
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