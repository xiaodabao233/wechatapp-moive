<template>
	<view class="container" :style="'position:'+(isShow?'fixed':'')">
		<view class="top">
			<view>
				<select-time :start-time="showTime" @selectDayEvent="changeTime"></select-time>
			</view>
			<view>
				<filter-nav :city-cinema-info="cityCinemaInfo" @change="changeCondition" @toggleShow="toggleShow"
					v-if="(isShow)"></filter-nav>
			</view>
		</view>
		<view class="main-content">
			<view class="cinema-list">
				<cinemaSection compName="cinemaSection" :data="{cinema,movieId:params.movieId,day:params.day}"
					v-for="(cinema,index) in (cinemas)" :key="index"></cinemaSection>
			</view>
			<view v-if="(!loadComplete && cinemas.length)">
				<loadingMore compName="loadingMore"></loadingMore>
			</view>
			<view v-if="(nothing)">
				<nothing compName="nothing" :data="{message:" 暂无符合条件的影院"}"></nothing>
			</view>
			<view v-if="(noSchedule)">
				<nothing compName="nothing" :data="{message:" 当天暂无场次"}"></nothing>
			</view>
		</view>
	</view>
</template>

<script>
	import filterNav from "@/components/filter-nav/filter-nav";
	import selectTime from "@/components/select-time/select-time";
	import cinemaSection from "../../../templates/cinemaSection/cinemaSection.vue";
	import nothing from "../../../templates/nothing/nothing.vue";
	import loadingMore from "../../../templates/loadingMore/loadingMore.vue";
	export default {
		components: {
			filterNav,
			selectTime,
			cinemaSection,
			nothing,
			loadingMore
		},
		data() {
			return {
				showTime: '',
				//影片上映日期
				isShow: false,
				//导航下拉框是否展开
				cityCinemaInfo: {},
				//影院过滤菜单
				params: {
					//影院搜索条件
					movieId: 0,
					day: '',
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
				cinemas: [],
				//影院列表 
				loadComplete: false,
				//数据是否加载完
				nothing: false,
				//是否有符合过滤的影院
				noSchedule: false //当天是否有场次，原本时间是由后台返回的，但是缺少城市ID就没有返回，导致当天可能没有播放场次
			};
		},
		onLoad(options) {
			this.initPage(options);
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
		methods: {
			initPage(options) {
				const movieId = options.movieId;
				const movieName = options.movieName;
				const showTime = options.showTime; //影片上映日期
				uni.setNavigationBarTitle({
					title: movieName
				});
				this.setData({
					showTime,
					params: {
						...this.params,
						movieId
					}
				});
				//select-time会触发事件来调用changeTime初始化数据
			},

			//获取影院列表
			getCinemas(params) {
				const _this = this;
				return new Promise((resolve, reject) => {
					uni.request({
						url: `https://m.maoyan.com/ajax/movie?forceUpdate=${Date.now()}`,
						method: 'POST',
						data: params,
						success(res) {
							// 缺少了城市ID所以返回值缺少showDays，只能自己模拟时间了
							resolve(res.data.cinemas);
							_this.setData({
								cinemas: _this.cinemas.concat(res.data.cinemas),
								loadComplete: !res.data.paging.hasMore
							});
						}
					});
				});
			},

			//获取过滤菜单数据
			getFilter() {
				const _this = this;
				const {
					params
				} = this;
				uni.request({
					url: `https://m.maoyan.com/ajax/filterCinemas?movieId=${params.movieId}&day=${params.day}`,
					success(res) {
						_this.setData({
							cityCinemaInfo: res.data
						});
					}
				});
			},

			//当选择的时间变化时触发
			changeTime(e) {
				const day = e.detail.day;
				this.setData({
					params: {
						...this.params,
						day
					},
					cinemas: [],
					isShow: false,
					//隐藏过滤下拉框
					noSchedule: false
				}, () => {
					uni.showLoading({
						title: '正在加载...'
					});
					this.getCinemas(this.params).then(list => {
						uni.hideLoading();
						if (!list.length) {
							this.setData({
								noSchedule: true
							});
						}
					});
					this.getFilter();
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

			//导航下拉框状态变化时的处理，在下拉框展开时禁止滚动穿透
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
	/* @import "../../../templates/cinemaSection/cinemaSection.vue"; */
	.top {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		color: #555;
		font-size: 28rpx;
		background: #fff;
		z-index: 10;
	}

	.main-content {
		padding-bottom: 30rpx;
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