<template>
	<view class="uni-common-pb">
		<view>
			<uni-load-more v-if="loading" status="loading" />
		</view>
		<view v-if="!loading">
			<view class="uni-padding-wrap uni-common-mt">
				<uni-segmented-control :current="current" :values="items" style-type="button" active-color="#dd524d" @clickItem="onClickItem" />
			</view>
			<view class="content">
				<view v-show="current === 0">
					<view class="uni-card" v-for="(list,index) in Projects" :key="index">
						<view class="uni-list">
							<view class="uni-list-cell" hover-class="uni-list-cell-hover" @click="goDetailPage(list)">
								<view :class="!list.HID ? 'uni-list-cell-navigate' : 'uni-list-cell-navigate uni-navigate-right'">
									<view>
										{{list.HouseEstateName}} 
										<view class="small tip-none" v-if="list.HID">
											<text class="tip-success mr30">剩余房源数：{{list.Surplus}}</text>
											<text class="subtitle">总房源数：{{list.Total}}</text>
										</view>
										<view class="small subtitle" v-if="!list.HID">当前无剩余房源 | 总房源数：{{list.Total}}</view>
									</view>
								</view>
							</view>
						</view>
					</view>
				</view>
				<view v-show="current === 1">
					<view class="uni-card" v-if="Periods.length == 0">
						<view class="uni-list">
							<view class="uni-list-cell">
								<view class="uni-list-cell-navigate">
									<view class="small subtitle" v-if="!list.HID">无即将选房时间表</view>
								</view>
							</view>
						</view>
					</view>
					<view class="uni-table" v-if="Periods.length > 0">
						<view class="uni-row-list">
							<view class="uni-row head">
								<view class="uni-column">
									日期
								</view>
								<view class="uni-column">
									时间段
								</view>
								<view class="uni-column">
									组
								</view>
								<view class="uni-column">
									号码
								</view>
							</view>
						</view>
						<view class="uni-row-list" v-for="(list,index) in Periods" :key="index">
							<view class="uni-row">
								<view class="uni-column">
									{{list.Date}} 
								</view>
								<view class="uni-column">
									<view>
										{{list.BeginTime}} 
									</view>
									<view>
										{{list.EndTime}} 
									</view>
								</view>
								<view class="uni-column">
									{{list.GroupId}} 
								</view>
								<view class="uni-column">
									{{list.Number}} 
								</view>
							</view>
						</view>
					</view>
				</view>
			</view>
		</view>
		
	</view>
</template>

<script>
	import { CONFIG } from '../../common/config.js'
	import uniLoadMore from '@/components/uni-load-more/uni-load-more.vue'
	import uniSegmentedControl from '@/components/uni-segmented-control/uni-segmented-control.vue'
	export default {
		components: {
			uniLoadMore,
			uniSegmentedControl
		},
		data() {
			return {
				CONFIG: CONFIG,
				loading: false,
				items: [
					'楼盘',
					'即将选房时间表'
				],
				current: 0,
				Projects: [],
				Periods: [],
				Number: ''
			}
		},
		onLoad(option) {
			// console.log(option.id, option.title)
			this.Number = option.id
			 wx.setNavigationBarTitle({
			  title: option.title
			})
			this.loadData()
		},
		onPullDownRefresh() {
			console.log('onPullDownRefresh');
			this.loadData(true)
		},
		methods: {
			onClickItem(index) {
				if (this.current !== index) {
					this.current = index
				}
			},
			goDetailPage (e) {
				if (e.HID) {
					uni.navigateTo({
						url: '/pages/index/house?HID=' + e.HID + '&Number=' + this.Number + '&title=' + e.HouseEstateName
					})
				}
			},
			loadData (isPull) {
				if (!isPull) {
					this.loading = true
				}
				uni.request({
					url: CONFIG.URL.subProject,
					method: 'GET',
					data: {
						Number: this.Number
					},
					success: (res) => {
						if (isPull) {
							uni.stopPullDownRefresh();
						} else {
							this.loading = false
						}
						this.Projects = res.data.Projects
						this.Periods = res.data.Periods
						// console.log(this.Projects)
					},
					fail: (err) => {
						if (isPull) {
							uni.stopPullDownRefresh();
						} else {
							this.loading = false
						}
						console.log('request fail', err);
						uni.showModal({
							content: err.errMsg,
							showCancel: false
						});
					}
				});
			}
		}
	}
</script>

<style>
</style>
