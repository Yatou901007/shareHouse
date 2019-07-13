<template>
	<view>
		<view class="home-section" style="padding: 0rpx 0rpx;">
			<view class="filter-content" v-for="(item, index) in menuList" :key="index" v-if="menuIndex == index">
				<view v-if="item.isSort">
					<view class="filter-content-list">
						<view v-for="(detailItem,idx) in selectDetailList" :key="idx" :class="detailItem.isSelected?'filter-content-list-item-active':'filter-content-list-item-default'"
						 :style="{'color': detailItem.isSelected?themeColor:'#666666'}" @tap="sortTap(idx,selectDetailList,item.key)">
							<text>{{detailItem.title}}</text>
						</view>
					</view>
				</view>
				<view v-else-if="item.isMore">
					<view class="filter-content-more">
						<view v-for="(item2, index2) in item.menuList" :key="index2" v-if="item.menuList">
							<view class="filter-content-title" v-if="item2.detailTitle && item2.detailTitle.length">
								<text>{{item2.detailTitle}}</text>
							</view>
							<view class="filter-content-detail">
								<text v-for="(detailItem,idx) in selectDetailList2[index2]" :key="idx" class='filter-content-detail-item-default' :style="{'background-color':detailItem.isSelected?themeColor:'#FFFFFF','color':detailItem.isSelected?'#FFFFFF':'#666666'}"
								 @tap="itemTap2(idx,index2,item2.isMutiple,item2.key)">
									{{detailItem.title}}
								</text>
							</view>
						</view>
					</view>
					<view class="filter-content-footer">
						<view class="filter-content-footer-item" style="color: #777777; background-color: #FFFFFF;" @tap="resetClickMore(selectDetailList2, item.menuList)">
							<text>重置</text>
						</view>
						<view class="filter-content-footer-item" :style="{'color': '#FFFFFF', 'background-color': themeColor}" @tap="sureClick">
							<text>确定</text>
						</view>
					</view>
				</view>	
				<view v-else>
					<view class="filter-content-title" v-if="item.detailTitle && item.detailTitle.length">
						<text>{{item.detailTitle}}</text>
					</view>
					<view class="filter-content-detail">
						<text v-for="(detailItem,idx) in selectDetailList" :key="idx" class='filter-content-detail-item-default' :style="{'background-color':detailItem.isSelected?themeColor:'#FFFFFF','color':detailItem.isSelected?'#FFFFFF':'#666666'}"
						 @tap="itemTap(idx,selectDetailList,item.isMutiple,item.key)">
							{{detailItem.title}}
						</text>
					</view>
					<view class="filter-content-footer">
						<view class="filter-content-footer-item" style="color: #777777; background-color: #FFFFFF;" @tap="resetClick(selectDetailList,item.key)">
							<text>重置</text>
						</view>
						<view class="filter-content-footer-item" :style="{'color': '#FFFFFF', 'background-color': themeColor}" @tap="sureClick">
							<text>确定</text>
						</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				selectArr: [],
				result: {},
				menuIndex: 0,
				selectDetailList: [],
				selectDetailList2: [],
				independenceObj: {},
				selectedKey: '',
				cacheSelectedObj: {}
			};
		},
		props: {
			themeColor: {
				type: String,
				default () {
					return '#D1372C'
				}
			},
			menuList: {
				type: Array,
				default () {
					return []
				}
			},
			independence: {
				type: Boolean,
				default: false
			}
		},
		computed: {
			selectedObj() {
				let obj = {}
				for (let i = 0; i < this.menuList.length; i++) {
					let item = this.menuList[i];
					if (!item.isMore) {
						this.dealItem(item, obj)
					} else {
						for (let j = 0; j < item.menuList.length; j++) {
							this.dealItem(item.menuList[j], obj)
						}
					}
				}
				this.result = obj;
				return obj
			}
		},
		methods: {
			dealItem (item, obj) {
				if (!this.independence && item.defaultSelectedIndex != null && item.defaultSelectedIndex.toString().length > 0) { // 处理并列菜单默认值

					if (item.isMutiple) {
						obj[item.key] = [];
						item.detailList[0].isSelected = false;
						if (!Array.isArray(item.defaultSelectedIndex)) { // 如果默认值不是数组
							item.defaultSelectedIndex = [item.defaultSelectedIndex];
						}
						for (let j = 0; j < item.defaultSelectedIndex.length; j++) { // 将默认选中的值放入selectedObj
							item.detailList[item.defaultSelectedIndex[j]].isSelected = true;
							obj[item.key].push(item.detailList[item.defaultSelectedIndex[j]].value)
						}

					} else {
						obj[item.key] = item.detailList[item.defaultSelectedIndex].value;
						item.detailList[0].isSelected = false;
						item.detailList[item.defaultSelectedIndex].isSelected = true;
					}
				} else {
					if (item.isMutiple) {
						obj[item.key] = [];
					} else {
						obj[item.key] = '';
					}
				}
			},
			dealTabClick (menuList, index) {
				this.selectDetailList = menuList[index].detailList;
				this.selectedKey = menuList[index].key;
				// 如果是独立菜单
				if (this.independence && !menuList[index].isSort) {
					if (JSON.stringify(this.independenceObj) == '{}') {
						this.initIndependenceObj(index);
					} else {
						
							for (let key in this.independenceObj) {
								if (key != this.selectedKey) {
									this.initIndependenceObj(index);
									this.resetSelected(menuList[index].detailList, this.selectedKey);
								}
							}
						
					}
					
				}
				if (this.independence && menuList[index].isSort) {
					
						this.independenceObj = {};
					

				}
				if (this.independence) {
					let idx = menuList[index].defaultSelectedIndex;
					if (idx != null && idx.toString().length > 0) { // 处理独立菜单默认值
						if (menuList[index].isMutiple) {
							for (let i = 0; i < idx.length; i++) {
								if (menuList[index].detailList[idx[i]].isSelected == false) {
									this.itemTap(idx[i], menuList[index].detailList, true, this.selectedKey);
								}

							}
						} else {
							if (menuList[index].detailList[idx].isSelected == false) {
								
								this.itemTap(idx, menuList[index].detailList, false, this.selectedKey);
								
							}
						}

					}
				}


				// #ifdef H5
				this.$forceUpdate();
				// #endif
			},
			menuTabClick(index) {
				this.menuIndex = index;
				if (!this.menuList[index].isMore) {
					this.dealTabClick(this.menuList, index)
				} else {
					this.selectDetailList2 = []
					// 前面已有的筛选条件
					const SHOW_LEN = this.menuList.length - 1
					for (let i = 0; i < SHOW_LEN; i++) {
						this.selectDetailList2.push(this.menuList[i].detailList)
					}
					// 后面的筛选条件
					for (let i = SHOW_LEN; i < this.menuList[index].menuList.length; i++) {
						this.selectDetailList2.push(this.menuList[index].menuList[i].detailList)
					}
					// console.log(this.selectDetailList2)
				}
			},
			initIndependenceObj(index) {
				this.independenceObj = {};
				if (this.menuList[index].isMutiple) {
					this.independenceObj[this.selectedKey] = [];
				} else {
					this.independenceObj[this.selectedKey] = '';
				}
			},
			itemTap2(index, index2, isMutiple, key) { // 全部筛选
				this.itemTap(index, this.selectDetailList2[index2], isMutiple, key)
			},
			itemTap3(index, index2, isMutiple, key) { // 仅看可选
				if (this.selectDetailList.length == 0) {
					this.selectDetailList = this.menuList[index].detailList
					for (let i = 0; i < this.selectDetailList.length; i++) {
						if (0 == i) {
							this.selectDetailList[i].isSelected = true
						} else {
							this.selectDetailList[i].isSelected = false
						}
					}
				}
				// console.log('---', this.selectDetailList[index2].isSelected, this.$parent.$parent.statusList[index].hasSelect)
				// 如果全部筛选中存在，赋值
				this.selectDetailList[index2].isSelected = this.$parent.$parent.statusList[index].hasSelect
				if (!this.selectDetailList[index2].isSelected) {
					this.selectedObj[key] = this.selectDetailList[index2].value;
				} else {
					this.selectedObj[key] = '';
				}
				
				this.result = this.selectedObj;

				for (let i = 0; i < this.selectDetailList.length; i++) {
					if (index2 == i) {
						this.selectDetailList[i].isSelected = !this.selectDetailList[i].isSelected
						this.selectDetailList[0].isSelected = !this.selectDetailList[i].isSelected
					} else {
						this.selectDetailList[i].isSelected = false
					}
				}
				// console.log('????' , this.selectDetailList[1].isSelected, key, this.selectedObj[key])
				// this.itemTap(index, this.selectDetailList, isMutiple, key)
				this.$emit("confirm", this.result);
			},
			itemTap(index, list, isMutiple, key) {

				if (isMutiple == true) {
					list[index].isSelected = !list[index].isSelected;
					if (index == 0) {
						this.resetSelected(list, key)
					} else {
						list[0].isSelected = false
						if (list[index].isSelected) {
							if (this.independence) {
								this.independenceObj[this.selectedKey].push(list[index].value);
							} else {
								this.selectedObj[key].push(list[index].value);
							}
						} else {
							list[index].isSelected = false;
							if (this.independence) {
								var idx = this.independenceObj[this.selectedKey].indexOf(list[index].value);
								this.independenceObj[this.selectedKey].splice(idx, 1);
							} else {
								var idx = this.selectedObj[key].indexOf(list[index].value);
								this.selectedObj[key].splice(idx, 1);
							}

						}
						if (this.independence) {
							this.result = this.independenceObj;
						} else {
							this.result = this.selectedObj;
						}
						// console.log('selectedObj new----', this.selectedObj)

					}
				} else {
					if (index == 0) {
						this.resetSelected(list, key)
					} else {
						list[0].isSelected = false
						if (this.independence) {
							this.independenceObj[this.selectedKey] = list[index].value;
							this.result = this.independenceObj;
						} else {
							this.selectedObj[key] = list[index].value;
							this.result = this.selectedObj;
						}

						for (let i = 0; i < list.length; i++) {
							if (index == i) {
								list[i].isSelected = true
							} else {
								list[i].isSelected = false
							}
						}
					}
				}
				// #ifdef H5
				this.$forceUpdate();
				// #endif
			},
			resetSelected(list, key) {
				if (typeof this.result[key] == 'object') {
					this.result[key] = [];
				} else {
					this.result[key] = '';
				}
				for (let i = 0; i < list.length; i++) {
					if (i == 0) {
						list[i].isSelected = true;
					} else {
						list[i].isSelected = false;
					}
				}
				// #ifdef H5
				this.$forceUpdate();
				// #endif
			},
			sortTap(index, list, key) {
				if (this.independence) {
					this.independenceObj[this.selectedKey] = list[index].value;
					this.result = this.independenceObj;
				} else {
					this.selectedObj[key] = list[index].value;
					this.result = this.selectedObj;
				}

				for (let i = 0; i < list.length; i++) {
					if (index == i) {
						list[i].isSelected = true;
					} else {
						list[i].isSelected = false;
					}
				}
				this.$emit("confirm", this.result);
			},
			sureClick() {
				this.$emit("confirm", this.result);
			},
			resetClick(list, key) {
				this.resetSelected(list, key)
			},
			resetClickMore(lists, menuList) {
				for (var i = 0; i < lists.length; i++) {
					this.resetSelected(lists[i], menuList[i].key)
				}
			}
		}
	}
</script>

<style>
	.filter-content {
		border-top: #eee 1rpx solid;
		background-color: #F6F7F8;
	}

	.filter-content-title {
		border-bottom: #EEEEEE 1rpx solid;
		padding: 20rpx 30rpx;
		font-size: 28rpx;
		color: #999999;
	}

	.filter-content-detail {
		padding: 10rpx 30rpx;
	}

	.filter-content-detail-item-active {
		background-color: #D1372C;
		color: #FFFFFF;
		padding: 10rpx 30rpx;
		border-radius: 40rpx;
		margin-right: 20rpx;
		margin-top: 20rpx;
		display: inline-block;
		font-size: 28rpx;
	}

	.filter-content-detail-item-default {
		background-color: #FFFFFF;
		color: #666666;
		padding: 10rpx 30rpx;
		border-radius: 40rpx;
		margin-right: 20rpx;
		margin-top: 20rpx;
		display: inline-block;
		font-size: 28rpx;
	}

	.filter-content-footer {
		display: flex;
		justify-content: space-between;
		width: 100%;
		height: 80rpx;
		margin-top: 20rpx;
	}

	.filter-content-footer-item {
		width: 50%;
		display: flex;
		justify-content: center;
		align-items: center;
		font-size: 32rpx;
	}

	.filter-content-list {

		padding: 10rpx 30rpx;
	}

	.filter-content-list-item-default {
		color: #666666;
		width: 100%;
		padding: 20rpx 0rpx;
	}

	.filter-content-list-item-default text {
		width: 90%;
		font-size: 28rpx;
		display: inline-block;
	}

	.filter-content-list-item-active {
		color: #D1372C;
		width: 100%;
		padding: 20rpx 0rpx;
	}

	.filter-content-list-item-active text {
		font-size: 28rpx;
		width: 90%;
		display: inline-block;
	}

	.filter-content-list-item-active:after {
		content: '✓';
	}
	
	.filter-content-more {
		/* max-height: calc(100vh - 115px); */
		max-height: 60vh;
		overflow: scroll;
	}
</style>
