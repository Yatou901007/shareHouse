<template>
	<view class="uni-common-pb">
		<view>
			<uni-load-more v-if="loading" status="loading" />
		</view>
		<view class="uni-card" v-for="(list,index) in lists" :key="index">
			<view class="uni-list">
				<view class="uni-list-cell uni-collapse">
					<view class="uni-list-cell-navigate uni-navigate-bottom" hover-class="uni-list-cell-hover" :class="list.open ? 'uni-active' : ''"
					 @click="triggerCollapse(index)">
						{{list.Name}}
					</view>
					<view class="uni-list uni-collapse" :class="list.open ? 'uni-active' : ''">
						<view class="uni-list-cell no-data" v-if="!list.Project || list.Project.length == 0">
							暂无数据
						</view>
						<view class="uni-list-cell" hover-class="uni-list-cell-hover" v-for="(item,key) in list.Project" :key="key" @click="goDetailPage(item)">
							<view :class="item.IsEnd ? 'uni-list-cell-navigate' : 'uni-list-cell-navigate uni-navigate-right'">
								<view>
									{{item.Name}}
									<view class="small tip-success" v-if="item.IsEnd">已完成</view>
									<view class="small tip-none" v-if="!item.IsEnd">未完成</view>
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
	export default {
		components: {
			uniLoadMore
		},
		data() {
			return {
				CONFIG: CONFIG,
				loading: false,
				lists: []
			}
		},
		onLoad() {
			this.loadData()
		},
		onPullDownRefresh() {
			console.log('onPullDownRefresh');
			this.loadData(true)
		},
		methods: {
			triggerCollapse(e) {
				for (var i = 0; i < this.lists.length; ++i) {
					if (e === i) {
						this.lists[i].open = !this.lists[e].open;
					}
				}
				this.lists = JSON.parse(JSON.stringify(this.lists))
			},
			goDetailPage (e) {
				if (!e.IsEnd) {
					uni.navigateTo({
						url: '/pages/index/projects?id=' + e.Number + '&title=' + e.Name
					})
				}
			},
			loadData (isPull) {
				if (!isPull) {
					this.loading = true
				}
				uni.request({
					url: CONFIG.URL.project,
					method: 'GET',
					data: {
						user: 'admin'
					},
					success: (res) => {
						if (isPull) {
							uni.stopPullDownRefresh();
						} else {
							this.loading = false
						}
						this.lists = res.data
						for (var i = 0; i < this.lists.length; ++i) {
							if (this.lists[i].Project && this.lists[i].Project.length > 0) {
								this.lists[i].open = true;
							} else {
								this.lists[i].open = false;
							}
						}
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
