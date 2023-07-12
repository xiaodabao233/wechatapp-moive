<template>
	<view class="container" :style="'position:'+(isShow?'fixed':'')">
		<view class="topbar">
			<navigator class="city-entry" url="../../subPages/city-select/city-select">
				<text class="city-name">{{city}}</text>
				<text class="city-entry-arrow"></text>
			</navigator>
			<navigator url="../../subPages/search-page/search-page?stype=2" class="search-input">
				<text class="iconfont icon-sousuo"></text>搜影院
			</navigator>
		</view>
		<view class="nav-wrapper">
			<filter-nav :city-cinema-info="cityCinemaInfo" @change="changeCondition" @toggleShow="toggleShow"></filter-nav>
		</view>
		<view class="cinema-list">
			<cinemaSection compName="cinemaSection" :data="{cinema}" v-for="(cinema,index) in (cinemas)" :key="index">
			</cinemaSection>
		</view>
		<view v-if="(!loadComplete && cinemas.length)">
			<loadingMore compName="loadingMore"></loadingMore>
		</view>
		<view v-if="(nothing)">
			<nothing compName="nothing" :data="{message:" 暂无符合条件的影院"}"></nothing>
		</view>
	</view>
</template>

<script>
	import filterNav from "@/components/filter-nav/filter-nav";
	import cinemaSection from "../../../templates/cinemaSection/cinemaSection.vue";
	import nothing from "../../../templates/nothing/nothing.vue";
	import loadingMore from "../../../templates/loadingMore/loadingMore.vue";
	const util = require('../../../utils/util.js');
	const app = getApp();
	export default {
		components: {
			filterNav,
			cinemaSection,
			nothing,
			loadingMore
		},
		data() {
			return {
				city: '正在定位...',
				params: {
					//url请求参数对象
					day: util.getToday(),
					offset: 0,
					limit: 20,
					districtId: -1,
					lineId: -1,
					hallType: -1,
					brandId: -1,
					serviceId: -1,
					areaId: -1,
					stationId: -1,
					item: '',
					updateShowDay: false
				},
				nothing: false,
				//结果是否为空
				cinemas: [],
				//影院列表
				cityCinemaInfo: {},
				//城市影院信息
				loadComplete: false,
				//数据是否加载完
				isShow: false //导航下拉框是否展开
			};
		},
		onLoad() {
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
			this.initPage();
		},
		onShow() {
			if (app.globalData.selectCity) {
				this.setData({
					city: app.globalData.selectCity.cityName
				});
			}
		},
		//上拉触底加载更多
		onReachBottom() {
			if (this.loadComplete) {
				return;
			}
			const params = {
				...this.params,
				offset: this.cinemas.length
			};
			this.getCinemas(params);
		},
		//转发
		onShareAppMessage(res) {
			return {
				title: '快来看看附近的电影院',
				path: 'pages/tabBar/movie/movie'
			};
		},
		methods: {
			//初始化页面
			initPage() {
				uni.showLoading({
					title: '正在加载...'
				});
				const _this = this;
				this.getCinemas(this.params).then(() => {
					uni.hideLoading();
				});
				uni.request({
					url: 'https://m.maoyan.com/ajax/filterCinemas',
					success(res) {
						_this.setData({
							cityCinemaInfo: res.data
						});
					}
				});
			},

			//获取影院列表
			getCinemas(params) {
				const _this = this;
				return new Promise((resolve, reject) => {
					uni.request({
						url: 'https://m.maoyan.com/ajax/cinemaList',
						data: params,
						success(res) {
							resolve(res.data.cinemas);
							_this.setData({
								cinemas: _this.cinemas.concat(res.data.cinemas),
								loadComplete: !res.data.paging.hasMore
							});
						}
					});
				});
			},

			//当过滤条件变化时调用的函数
			changeCondition(e) {
				const obj = e.detail;
				uni.showLoading({
					title: '正在加载...'
				});
				this.setData({
					params: {
						...this.params,
						...obj
					},
					cinemas: [],
					nothing: false
				}, () => {
					this.getCinemas(this.params).then(list => {
						if (!list.length) {
							this.setData({
								nothing: true
							});
						}
						uni.hideLoading();
					});
				});
			},

			//导航下拉框状态变化时的处理
			toggleShow(e) {
				const item = e.detail.item;
				this.setData({
					isShow: item !== -1
				});
			}
		}
	};
</script>

<style>
	/* @import "../../../templates/cinemaSection/cinemaSection.css"; */

	.topbar {
		position: fixed;
		width: 100vw;
		top: 0;
		height: 90rpx;
		border: none;
		background-color: #f5f5f5;
	}

	.search-input {
		flex-grow: 1;
		height: 56rpx;
		font-size: 26rpx;
		color: #999;
		border: 1rpx solid #e6e6e6;
		border-radius: 10rpx;
		margin: 0 30rpx;
		background: #fff;
		line-height: 56rpx;
		text-align: center;
	}

	.icon-sousuo {
		display: inline-block;
		margin-right: 10rpx;
		font-size: 24rpx;
		color: #999;
	}

	.nav-wrapper {
		position: fixed;
		z-index: 10;
		top: 90rpx;
		left: 0;
		width: 100vw;
		height: 80rpx;
		background: #fff;
	}

	.cinema-list {
		margin-top: 170rpx;
	}

	::-webkit-scrollbar {
		width: 0;
		height: 0;
		color: transparent;
	}
</style>