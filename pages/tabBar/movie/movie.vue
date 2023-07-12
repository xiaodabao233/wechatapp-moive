<template>
	<view>
		<view class="topbar">
			<navigator class="city-entry" url="../../subPages/city-select/city-select">
				<text class="city-name">{{city}}</text>
				<text class="city-entry-arrow"></text>
			</navigator>
			<view class="switch-hot">
				<view :class="'hot-item '+(switchItem===0 ? 'active' :'')" @tap="selectItem" :data-item="0">正在热映</view>
				<view :class="'hot-item '+(switchItem===1 ? 'active' :'')" @tap="selectItem" :data-item="1">即将上映</view>
			</view>
			<navigator class="search-entry" url="../../subPages/search-page/search-page?stype=-1">
				<text class="iconfont icon-sousuo"></text>
			</navigator>
		</view>
		<view class="switch-content">
			<view v-if="switchItem !== 1">
				<movieSection compName="movieSection" :data="{movie:movie}" v-for="(movie,index) in (movieList0)"
					:key="index">
				</movieSection>
				<view v-if="(!loadComplete0 && movieList0.length)">
					<loadingMore compName="loadingMore"></loadingMore>
				</view>
				<view v-if="movieList0.length===0">
					<nothing compName="nothing" :data="{message:" 暂无影片"}"></nothing>
				</view>
			</view>
			<view v-if="switchItem !== 0">
				<view class="most-expected" v-if="mostExpectedList.length">
					<view class="title">近期最受期待</view>
					<scroll-view class="scroll-view_H" scroll-x @scrolltolower="lower">
						<navigator :url="'/pages/subPages/movie-detail/movie-detail?movieId='+(movie.id)"
							class="expected-item" v-for="(movie,index) in (mostExpectedList)" :key="index">
							<image :src="movie.img" class="poster"></image>
							<view class="name line-ellipsis">{{movie.nm}}</view>
							<view class="data line-ellipsis">{{movie.wish}}人想看</view>
							<view class="data">{{movie.comingTitle}}</view>
						</navigator>
					</scroll-view>
				</view>
				<block v-for="(movie,index) in (movieList1)" :key="index">
					<block v-if="index===0||movieList1[index-1].comingTitle!==movie.comingTitle">
						<view class="title">{{movie.comingTitle}}</view>
						<movieSection compName="movieSection" :data="{movie:movie,rt:true}"></movieSection>
					</block>
					<movieSection compName="movieSection" :data="{movie:movie,rt:true}" v-else></movieSection>
				</block>
				<view v-if="(!loadComplete1 && movieList1.length)">
					<loadingMore compName="loadingMore"></loadingMore>
				</view>
				<view v-if="movieList1.length===0">
					<!-- <template is='nothing' data='{{message:"暂无影片"}}' /> -->
					暂无影片
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import movieSection from "../../../templates/movieSection/movieSection.vue";
	import nothing from "../../../templates/nothing/nothing.vue";
	import loadingMore from "../../../templates/loadingMore/loadingMore.vue";
	const request = require('../../../utils/request');
	const app = getApp();
	export default {
		components: {
			movieSection,
			nothing,
			loadingMore
		},
		data() {
			return {
				city: '正在定位...',
				// city: '成都',
				switchItem: 0,

				//默认选择‘正在热映’
				//‘正在热映’数据
				movieList0: [],

				movieIds0: [],
				loadComplete0: false,

				//‘正在上映’数据是否加载到最后一条
				//‘即将上映’数据
				mostExpectedList: [],

				movieList1: [],
				movieIds1: [],
				loadComplete1: false,

				//水平滚动加载的数据是否加载完毕
				loadComplete2: false,

				movie: {
					id: "",
					img: "",
					nm: "",
					wish: "",
					comingTitle: ""
				},

				comingTitle: ""
			};
		},
		onLoad() {
			this.initPage();
		},
		onShow() {
			if (app.globalData.selectCity) {
				this.setData({
					city: app.globalData.selectCity.cityName
				});
			}
		},
		//上拉触底刷新
		onReachBottom() {
			const {
				switchItem,
				movieList0,
				movieIds0,
				loadComplete0,
				movieList1,
				movieIds1,
				loadComplete1
			} = this;
			if (this.switchItem === 0) {
				this.ReachBottom(movieList0, movieIds0, loadComplete0, 0);
			} else {
				this.ReachBottom(movieList1, movieIds1, loadComplete1, 1);
			}
		},
		//转发
		onShareAppMessage(res) {
			return {
				title: '快来看看附近的电影院',
				path: 'pages/tabBar/movie/movie'
			};
		},
		methods: {
			initPage() {
				//https://www.jianshu.com/p/aaf65625fc9d   解释的很好
				if (app.globalData.userLocation) {
					this.setData({
						city: app.globalData.selectCity ? app.globalData.selectCity.cityName : '定位失败'
					});
				} else {
					app.globalData.userLocationReadyCallback = () => {
						this.setData({
							city: app.globalData.selectCity ? app.globalData.selectCity.cityName : '定位失败'
						});
					};
				}
				this.firstLoad();
			},
			//第一次加载页面时请求‘正在热映的数据’
			async firstLoad() {
				uni.showLoading({
					title: '正在加载...'
				});
				const [res, err] = await request({
					api: '/ajax/movieOnInfoList'
				});
				if (!err) {
					const {
						movieList = [],
							movieIds = []
					} = res;
					const movieList0 = this.formatImgUrl(movieList);
					this.setData({
						movieIds0: movieIds,
						movieList0
					});
					if (movieList.length >= movieIds.length) {
						this.setData({
							loadComplete0: true
						});
					}
				}
				uni.hideLoading();
			},
			//切换swtch
			selectItem(e) {
				const item = e.currentTarget.dataset.item;
				this.setData({
					switchItem: item
				});
				if (item === 1 && !this.mostExpectedList.length) {
					uni.showLoading({
						title: '正在加载...'
					});
					request({
						api: '/ajax/mostExpected?limit=10&offset=0&token='
					}).then(([res]) => {
						this.setData({
							mostExpectedList: this.formatImgUrl(res.coming || [], true)
						});
					}).finally(() => {
						uni.hideLoading();
					});
					request({
						api: '/ajax/comingList',
						data: {
							limit: 10,
							optimus_uuid: 'B52C96001E4B11EA928853AC6CFDBC0221750D6FF9374A35B50A070B3195ED15',
							optimus_risk_level: 71,
							optimus_code: 10,
							token: ''
						}
					}).then(([res]) => {
						this.setData({
							movieIds1: res.movieIds || [],
							movieList1: this.formatImgUrl(res.coming || [])
						});
					});
				}
			},

			//上拉触底刷新的加载函数
			async ReachBottom(list, ids, complete, item) {
				if (complete) {
					return;
				}
				const length = list.length;
				if (length + 10 >= ids.length) {
					this.setData({
						[`loadComplete${item}`]: true
					});
				}
				let query = ids.slice(length, length + 10).join('%2C');
				const [res, err] = await request({
					api: `/ajax/moreComingList?token=&movieIds=${query}`
				});
				if (!err) {
					const arr = list.concat(this.formatImgUrl(res.coming || []));
					this.setData({
						[`movieList${item}`]: arr
					});
				}
			},

			//滚动到最右边时的事件处理函数
			async lower() {
				const {
					mostExpectedList,
					loadComplete2
				} = this;
				const length = mostExpectedList.length;
				if (loadComplete2) {
					return;
				}
				const [res, err] = await request({
					api: "/ajax/mostExpected",
					data: {
						limit: 10,
						offset: length,
						token: ''
					}
				});
				if (!err) {
					this.setData({
						mostExpectedList: mostExpectedList.concat(this.formatImgUrl(res.coming || [], true)),
						loadComplete2: !res.paging.hasMore || !res.coming.length //当返回的数组长度为0时也认为数据请求完毕
					});
				}
			},

			//处理图片url
			formatImgUrl(arr, cutTitle = false) {
				//小程序不能在{{}}调用函数，所以我们只能在获取API的数据时处理，而不能在wx:for的每一项中处理
				if (!Array.isArray(arr)) {
					return;
				}
				let newArr = [];
				arr.forEach(item => {
					let title = item.comingTitle;
					if (cutTitle) {
						title = item.comingTitle.split(' ')[0];
					}
					let imgUrl = item.img.replace('w.h', '128.180');
					newArr.push({
						...item,
						comingTitle: title,
						img: imgUrl
					});
				});
				return newArr;
			}
		}
	};
</script>

<style>
	/* @import "../../../templates/movieSection/movieSection.css"; */
	.switch-hot {
		display: flex;
		height: 88rpx;
		line-height: 88rpx;
		position: relative;
	}

	.hot-item {
		font-size: 30rpx;
		color: #666;
		width: 21.33333vw;
		text-align: center;
		margin: 0 24rpx;
		font-weight: 700;
	}

	.hot-item.active {
		color: #ef4238;
		border-bottom: 2px solid #ef4238;
	}
	.search-entry {
		color: #ef4238;
		font-weight: 700;
		height: 100%;
		line-height: 90rpx;
		padding: 0 30rpx;
	}
	.search-entry .iconfont {
		font-size: 36rpx;
	}
	/* switch-content */
	.switch-content {
		padding-bottom: 30rpx;
	}
	.title {
		padding: 0 30rpx;
		padding-top: 20rpx;
		font-size: 28rpx;
		color: #333;
	}
	.most-expected {
		position: relative;
		padding: 0 30rpx;
		padding-bottom: 20rpx;
		border-bottom: 20rpx solid #f5f5f5;
	}
	.most-expected .title {
		padding-left: 0;
		margin-bottom: 20rpx;
	}
	.expected-item {
		display: inline-block;
		width: 170rpx;
		overflow: hidden;
		margin-right: 20rpx;
	}
	.poster {
		position: relative;
		width: 170rpx;
		height: 230rpx;
		margin-bottom: 12rpx;
	}
	.name {
		margin-bottom: 3px;
		font-size: 24rpx;
		color: #333;
	}
	.data {
		font-size: 24rpx;
		color: #999;
	}
</style>