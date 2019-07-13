<template>
	<view class="content">
		<view class="select-tab" :style="{height: tabHeight+'rpx'}">
			<view class="select-tab-item" :style="{width: itemWidth, backgroundColor:statusList[index].hasSelect ? themeColor :'#fff'}" v-for="(item,index) in menuList" :key="index" @tap="showMenuClick(index, item)">
				
				<text v-if="item.key != 'IsSelected'">
					<text :style="{color:statusList[index].hasSelect ? '#fff' : color}">{{item.title}}</text>
					<text class="arrows sl-font" :style="{color:statusList[index].hasSelect ? '#fff': themeColor}" v-if="statusList[index].isActive" :class="up"></text>
					<text class="arrows sl-font" :style="{color:statusList[index].hasSelect ? '#fff': color}" v-if="!statusList[index].isActive" :class="down"></text>
				</text>
				<text v-else>
					<text :style="{color:statusList[index].hasSelect ? '#fff' : color}">仅看可选</text>
				</text>
			</view>
		</view>
		<popup-layer ref="popupRef" :direction="'bottom'" @close="close" :isTransNav="isTransNav" :navHeight="navHeight"
		 :tabHeight="tabHeight">
			<sl-filter-view :independence="independence" :themeColor="themeColor" :menuList="menuListTemp" ref="slFilterView" @confirm="filterResult"></sl-filter-view>
		</popup-layer>
	</view>

</template>

<script>
	import popupLayer from '@/components/sl-filter/popup-layer.vue';
	import slFilterView from '@/components/sl-filter/filter-view.vue';
	export default {
		components: {
			popupLayer,
			slFilterView
		},
		props: {
			menuList: Array,
			default () {
				return []
			},
			themeColor: {
				type: String,
				default () {
					return '#000000'
				}
			},
			color: {
				type: String,
				default () {
					return '#666666'
				}
			},
			independence: {
				type: Boolean,
				default: false
			},
			isTransNav: {
				type: Boolean,
				default: false
			},
			navHeight: {
				type: Number,
				default: 0
			}
		},

		computed: {
			itemWidth() {
				return 'calc(100%/2)'
			},
			menuListTemp() {
				let arr = this.menuList;
				for (let i = 0; i < arr.length; i++) {
					let item = arr[i];
					if (!item.isMore) {
						for (let j = 0; j < item.detailList.length; j++) {
							let d_item = item.detailList[j];
							if (j == 0) {
								d_item.isSelected = true
							} else {
								d_item.isSelected = false
							}
						}
					} else {
						let subMenuList = item.menuList
						for (let m = 0; m < subMenuList.length; m++) {
							for (let n = 0; n < subMenuList[m].detailList.length; n++) {
								let d_item = subMenuList[m].detailList[n];
								if (n == 0) {
									d_item.isSelected = true
								} else {
									d_item.isSelected = false
								}
							}
						}
					}
					
				}
				return arr;
			}
		},
		// #ifndef H5
		onReady: function() {
			let arr = [];
			for (let i = 0; i < this.menuList.length; i++) {
				arr.push({
					'isActive': false,
					'hasSelect': false
				});
			}
			this.statusList = arr;
		},
		// #endif
		
		// #ifdef H5
		created:function(){
			let arr = [];
			for (let i = 0; i < this.menuList.length; i++) {
				arr.push({
					'isActive': false,
					'hasSelect': false
				});
			}
			this.statusList = arr;
			
		},
		// #endif
		data() {
			return {
				down: 'sl-down',
				up: 'sl-up',
				tabHeight: 80,
				statusList: []
			};
		},
		methods: {
			showMenuClick(index, item) {
				if (item.key == 'IsSelected') {
					this.$refs.slFilterView.itemTap3(index, 1, item.isMutiple, item.key)
				} else {
					if (this.statusList[index].isActive == true) {
						this.$refs.popupRef.close();
						this.statusList[index].isActive = false
					} else {
						this.menuTabClick(index);
						this.$refs.popupRef.show()
					}
				}
				
			},
			menuTabClick(index) {
				this.$refs.slFilterView.menuTabClick(index);
				for (let i = 0; i < this.statusList.length; i++) {
					if (index == i) {
						this.statusList[i].isActive = true;
					} else {
						this.statusList[i].isActive = false;
					}
				}
			},
			filterResult(val) {
				// 处理当前筛选项是否选中
				for (let key in val) {
					for (let i = 0; i < this.menuList.length; i++) {
						if (key == this.menuList[i].key) { // 当前的key
							if(typeof val[key] == 'number' || val[key].length > 0) {
								// console.log('选中项：',key)
								this.statusList[i].hasSelect = true
							} else {
								this.statusList[i].hasSelect = false
							}
							
						}
					}
				}
				// 处理全部筛选是否选中
				let allSelect = false
				for (let key in val) {
					if(typeof val[key] == 'number' || val[key].length > 0) {
						allSelect = true
					}
				}
				this.statusList[this.menuList.length - 1].hasSelect = allSelect
				
				this.$refs.popupRef.close()
				this.$emit("result", val)
			},
			close() {
				for (let i = 0; i < this.statusList.length; i++) {
					this.statusList[i].isActive = false;
				}
			}
		}
	}
</script>

<style>
	@import 'iconfont/iconfont.css';

	.select-tab {
		border-top: #ccc 1rpx solid;
		border-bottom: #ccc 1rpx solid;
		display: flex;
		position:fixed;
		height:80rpx;
		top:0;
		width:100vw;
		z-index:99;
		/* background:#F6F7F8; */
		background:#EEEEEE;
	}

	.arrows {
		margin-left: 5rpx;
	}

	.select-tab .select-tab-item {
		display: flex;
		justify-content: center;
		align-items: center;
		border-radius: 40rpx;
		/* border: 1rpx solid #ccc; */
		margin: 10rpx 10rpx;
		background: #fff;
	}

	.select-tab .select-tab-item text {
		color: #333;
		font-weight: bold;
		font-size: 28rpx;
	}
	.select-tab .select-tab-item > text {
		
	}
</style>
