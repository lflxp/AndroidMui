<template>
	<view class="page">
		<scroll-view id="tab-bar" class="swiper-tab" scroll-x :scroll-left="scrollLeft">
			<block v-for="(tab,index) in tabs" :key="tab.id">
				<view :class="['swiper-tab-list',currentTab==index ? 'on' : '']" :id="tab.id" :data-current="index" @tap="swichNav">{{tab.name}}</view>
			</block>
		</scroll-view>
		<!-- <swiper :current="currentTab" class="swiper-box" duration="300" @change="bindChange">
			<block v-for="(tab,index1) in newsitems" :key="index1">
				<swiper-item>
					<scroll-view class="index-bd" scroll-y @scrolltolower="loadMore(index1)">
						<block v-for="(newsitem,index2) in tab" :key="index2">
							<view class="tab-list">{{newsitem.name}}-{{newsitem.label}}</view>
						</block>
					</scroll-view>
				</swiper-item>
			</block>
		</swiper> -->
		<swiper :current="currentTab" class="swiper-box" duration="300" @change="bindChange">
			<block>
				<swiper-item>
					<scroll-view class="index-bd" scroll-y @scrolltolower="loadMore(index1)">
						<block>
							<view class="uni-list">
								<view class="uni-list-cell" hover-class="uni-list-cell-hover" v-for="(value,key) in listData" :key="key" @click="goDetail(value)">
									<view class="uni-media-list">
										<image class="uni-media-list-logo" :src="value.cover"></image>
										<view class="uni-media-list-body">
											<view class="uni-media-list-text-top">{{value.title}}</view>
											<view class="uni-media-list-text-bottom">
												<text>{{value.author_name}}</text>
												<text>{{value.published_at}}</text>
											</view>
										</view>
									</view>
								</view>
							</view>
						</block>
					</scroll-view>
				</swiper-item>
			</block>
		</swiper>
	</view>
</template>
<script>
	var dateUtils = require('../../../common/util.js').dateUtils;

	export default {
		data() {
			return {
				title: 'tabbar',
				scrollLeft: 0,
				isClickChange: false,
				currentTab: 0,
				listData: [],
				last_id: "",
				reload: false,
				tabs: [{
					name: '关注',
					id: 'guanzhu'
				}, {
					name: '推荐',
					id: 'tuijian'
				}, {
					name: '体育',
					id: 'tiyu'
				}, {
					name: '热点',
					id: 'redian'
				}, {
					name: '财经',
					id: 'caijing'
				}, {
					name: '娱乐',
					id: 'yule'
				}, {
					name: '军事',
					id: 'junshi'
				}, {
					name: '历史',
					id: 'lishi'
				}, {
					name: '本地',
					id: 'bendi'
				}],
				newsitems: []
			}
		},
		onLoad: function () {
			this.newsitems = this.randomfn()
			this.getList()
		},
		onPullDownRefresh() {
			this.reload = true;
			this.last_id = "";
			this.getList();
		},
		onUnload:function(){
			this.scrollLeft = 0,
				this.isClickChange = false,
				this.listData = [],
				this.last_id = "";
				this.currentTab = 0;
		},
		onReachBottom() {
			this.getList();
		},
		methods: {
			getList() {
				var data = {
					column: "id,post_id,title,author_name,cover,published_at" //需要的字段名
				};
				console.log(this.last_id);
				if (this.last_id) { //说明已有数据，目前处于上拉加载
					data.minId = this.last_id;
					data.time = new Date().getTime() + "";
					data.pageSize = 10;
				}
				uni.request({
					url: 'https://spider.dcloud.net.cn/api/news',
					data: data,
					success: (data) => {
						if (data.statusCode == 200) {
							let list = this.setTime(data.data);
							this.listData = this.reload ? list : this.listData.concat(list);
							this.last_id = list[list.length - 1].id;
							this.reload = false;
						}
					},
					fail: (data, code) => {
						console.log('fail'+JSON.stringify(data));
					}
				})
			},
			goDetail: function (e) {
				console.log(e);
				if (!/前|刚刚/.test(e.published_at)) {
					e.published_at = dateUtils.format(e.published_at);
				}
				let detail = {
					author_name: e.author_name,
					cover: e.cover,
					id: e.id,
					post_id: e.post_id,
					published_at: e.published_at,
					title: e.title
				}
				uni.navigateTo({
					url: "../list2detail-detail/list2detail-detail?detailDate=" + JSON.stringify(detail)
				})
			},
			setTime: function (items) {
				var newItems = [];
				items.forEach((e) => {
					newItems.push({
						author_name: e.author_name,
						cover: e.cover,
						id: e.id,
						post_id: e.post_id,
						published_at: dateUtils.format(e.published_at),
						title: e.title
					});
				});
				return newItems;
			},
			bindChange: async function (e) {
				let index = e.target.current;
				if (this.isClickChange) {
					this.currentTab = index;
					this.isClickChange = false;
					return;
				}
				let tabBar = await this.getWidth("tab-bar"),
					tabBarScrollLeft = tabBar.scrollLeft;

				let width = 0;

				for (let i = 0; i < index; i++) {
					let result = await this.getWidth(this.tabs[i].id);
					width += result.width;
				}

				let winWidth = uni.getSystemInfoSync().windowWidth,
					nowElement = await this.getWidth(this.tabs[index].id),
					nowWidth = nowElement.width;

				if (width + nowWidth - tabBarScrollLeft > winWidth) {
					this.scrollLeft = width + nowWidth - winWidth;
				}
				if (width < tabBarScrollLeft) {
					this.scrollLeft = width;
				}
				this.isClickChange = false;
				this.currentTab = index; //一旦访问data就会出问题
			},
			getWidth: function (id) { //得到元素的宽高
				return new Promise((res, rej) => {
					uni.createSelectorQuery().select("#" + id).fields({
						size: true,
						scrollOffset: true
					}, (data) => {
						if (id === 'tab-bar') {
							console.log("id=", id, "数据:", data)
						}
						res(data);
					}).exec();
				})
			},
			swichNav: async function (e) { //点击tab-bar
				if (this.currentTab === e.target.dataset.current) {
					return false;
				} else {
					let tabBar = await this.getWidth("tab-bar"),
						tabBarScrollLeft = tabBar.scrollLeft; //点击的时候记录并设置scrollLeft
					this.scrollLeft = tabBarScrollLeft;
					this.isClickChange = true;
					this.currentTab = e.target.dataset.current
				}
			},
			loadMore: function (e) {
				let last = this.newsitems[e][this.newsitems[e].length - 1].label,
					name = this.newsitems[e][this.newsitems[e].length - 1].name;
				for (let i = 1; i <= 10; i++) {
					this.newsitems[e].push({
						name: name,
						label: i + last
					});
				}
			},
			randomfn() {
				let ary = [];
				for (let i = 0, length = this.tabs.length; i < length; i++) {
					let aryItem = [];
					for (let j = 1; j <= 20; j++) {
						aryItem.push({
							name: this.tabs[i].name,
							label: j
						});
					}
					ary.push(aryItem);
				}
				return ary;
			}
		}
	}
</script>

<style>
	@import "../../../common/uni.css";

	page {
		display: flex;
	}

	.index {
		display: flex;
		flex: 1;
		flex-direction: column;
		overflow: hidden;
		height: 100%;
	}

	.index-bd {
		width: 750px;
		height: 100%;
	}

	.swiper-tab {
		width: 100%;
		white-space: nowrap;
		line-height: 64px;
		height: 64px;
	}


	.swiper-tab-list {
		font-size: 30px;
		width: 150px;
		display: inline-block;
		text-align: center;
		color: #777777;
	}

	.on {
		color: #FF0000;
		border-bottom: 5px solid #FF0000;
	}

	.swiper-box {
		flex: 1;
		width: 100%;
		height: 100%;
	}

	.swiper-box view {
		text-align: center;
	}

	.tab-list {
		width: 100%;
		height: 90px;
		line-height: 90px;
		text-align: left;
		border-bottom: 2px solid #EFEFF4;
	}

	uni-media-list-logo {
		width: 180px;
		height: 140px;
	}

	.uni-media-list-body {
		height: auto;
		justify-content: space-around;
	}

	.uni-media-list-text-top {
		height: 74px;
		font-size: 28px;
		overflow: hidden;
	}

	.uni-media-list-text-bottom {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
	}
</style>
