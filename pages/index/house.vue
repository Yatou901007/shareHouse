<template>
	<view class="uni-common-pb">
		<sl-filter v-if="menuList.length > 0" :themeColor="themeColor" :menuList="menuList" @result="result"></sl-filter>
		<zx-sorts :sortlist="sortList" v-if="filterResult.IsSelected === 0" @result="resultSort"></zx-sorts>
		<view :style="{'marginTop':filterResult.IsSelected === 0 ? '160rpx': '80rpx'}">
			<view>
				<uni-load-more v-if="loading" status="loading" />
			</view>
			<view v-if="!loading">
				<view class="uni-card" v-for="(list,index) in houses" :key="index">
					<view class="uni-list">
						<view class="uni-list-cell" hover-class="uni-list-cell-hover" @click="goDetailPage(list)">
							<view :class="!list.HID ? 'uni-list-cell-navigate' : 'uni-list-cell-navigate uni-navigate-right'">
								<view>
									<view>
										{{list.Building}} 号楼 {{list.Unit}}单元 {{list.RoomNumber}}
									</view>
									<view class="small subtitle">
										{{list.Group}} | {{list.Toward}} | {{list.RoomType}} | {{list.RoomTypeCode}}
									</view>
									<view class="small subtitle" v-if="filterResult.IsSelected === 0">
										单价:{{CONFIG.strfmoney(list.AreaUnitPrice)}} | 总价:{{CONFIG.strfmoney(list.TotalPrice)}}
									</view>
									<view class="small subtitle" v-if="filterResult.IsSelected === 0">
										建筑面积:{{list.EstimateBuiltUpArea}} | 使用面积:{{list.EstimateLivingArea}}
									</view>
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
	// import uniFilters from '@/components/uni-filters/uni-filters.vue'
	import zxSorts from '@/components/zx-sorts/zx-sorts.vue'
	import slFilter from '@/components/sl-filter/sl-filter.vue'
	export default {
		components: {
			uniLoadMore,
			// uniFilters,
			zxSorts,
			slFilter
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
				houses: [],
				HID: '',
				Number: '',
				themeColor: '#b22222',
				searchParams: {},
                filterResult: {},
				menuList: [],
				sortResult: {},
				sortList: [
					{
						title: '单价',
						key: 'AreaUnitPrice'
					},
					{
						title: '总价',
						key: 'TotalPrice'
					},
					{
						title: '建筑面积',
						key: 'EstimateBuiltUpArea'
					},
					{
						title: '使用面积',
						key: 'EstimateLivingArea'
					}
				]
			}
		},
		onLoad(option) {
			// console.log(option.HID, option.Number)
			this.HID = option.HID
			this.Number = option.Number
			 wx.setNavigationBarTitle({
			  title: option.title
			})
			this.loadFilter()
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
				console.log('1111', e)
			},
			buildSearchParams () {
				let param = Object.assign({}, this.filterResult, this.sortResult);
				console.log('param:', param)
				this.loadData(false, param)
			},
			result(val) {
                // this.filterResult = JSON.stringify(val, null, 2)
				this.filterResult = JSON.parse(JSON.stringify(val))
                console.log('filter_result:', val);
				for (let key in this.filterResult) {
					// 如果是数组，转换成字符串
					if (Array.isArray(this.filterResult[key])) {
						this.filterResult[key] = this.filterResult[key].join(',')
					}
				}
				if (this.filterResult.IsSelected === '') {
					this.sortResult = {}
				}
				// console.log(this.filterResult)
				this.buildSearchParams()
            },
			resultSort (val) {
				this.sortResult = val
				this.buildSearchParams()
				console.log('sort_result', val)
			},
			loadFilter() {
				uni.request({
					url: CONFIG.URL.filter,
					method: 'GET',
					data: {
						Number: this.Number,
						HID: this.HID
					},
					success: (res) => {
						// console.log('filter detail', res)
						let menuList = res.data
						this.menuList = [];
						const SHOW_LEN = 3
						if (menuList.length > SHOW_LEN) {
							let menuTemp = []
							for (var i = 0; i < SHOW_LEN; i++) {
								let _ml = menuList[i]
								menuTemp.push(_ml)
							}
							menuTemp.push({
								'title': '全部筛选',
								'key': 'more',
								'isMore': true,
								'menuList': []
							})
							for (var i = 0; i < menuList.length; i++) {
								let _ml = menuList[i]
								menuTemp[SHOW_LEN].menuList.push(_ml)
							}
							this.menuList = menuTemp;
							console.log(this.menuList)
						} else {
							this.menuList = res.data
						}
					},
					fail: (err) => {
					}
				});
			},
			loadData (isPull, params) {
				if (!isPull) {
					this.loading = true
				}
				if (!params) params = {}
				params['Number'] = this.Number
				params['HID'] = this.HID
				uni.request({
					url: CONFIG.URL.house,
					method: 'GET',
					data: params,
					success: (res) => {
						if (isPull) {
							uni.stopPullDownRefresh();
						} else {
							this.loading = false
						}
						this.houses = res.data
						// console.log(this.houses)
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
