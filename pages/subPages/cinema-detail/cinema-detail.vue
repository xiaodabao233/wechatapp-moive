<template>
    <view v-if="cinemaDetail">
        <view>
            <cinemaMap compName="cinemaMap" :data="{ cinemaData: cinemaDetail.cinemaData }"></cinemaMap>
        </view>
        <view>
            <view>
                <select-movie :movies="movies" @selectMovie="selectMovie" :defaultSelectID="movieId" />
            </view>
            <view class="movie-info" v-if="movie">
                <view class="movie-title line-ellipsis">
                    <text class="title">{{ movie.nm }}</text>
                    <text class="grade">
                        <text v-if="!movie.globalReleased">
                            {{ movie.wish }}
                            <text class="small">人想看</text>
                        </text>
                        <text v-else-if="movie.sc !== '0.0'">
                            {{ movie.sc }}
                            <text class="small">分</text>
                        </text>
                        <text v-else><text class="small">暂无评分</text></text>
                    </text>
                </view>
                <view class="movie-desc line-ellipsis">{{ movie.desc }}</view>
            </view>
        </view>
        <view>
            <select-time :days="days" @selectDayEvent="selectDay" :defaultSelect="day"></select-time>
        </view>
        <view>
            <view v-if="timeList.length">
                <view class="time item" @tap="buyTicket" :data-info="item" v-for="(item, index) in timeList" :key="item.tm">
                    <view class="time-block box">
                        <view class="begin">{{ item.tm }}</view>
                        <view class="end">{{ item.endTime }} 散场</view>
                    </view>

                    <view class="info-block box">
                        <view class="lan line-ellipsis">{{ item.lang }} {{ item.tp }}</view>
                        <view class="hall line-ellipsis">{{ item.th }}</view>
                    </view>

                    <view class="price box">
                        <view class="price-box line-ellipsis">
                            <view class="sellPr">
                                <text class="stonefont">{{ (item.vipPrice && item.vipPrice * 1 + 10) || 37 }}</text>
                                <text class="d">元</text>
                            </view>
                            <view class="vipPrice" v-if="item.vipPriceName || item.vipPriceNameNew">
                                <text class="icon">{{ item.vipPriceName || item.vipPriceNameNew }}</text>
                                <text class="num">{{ item.vipPrice }}</text>
                            </view>
                        </view>
                        <view class="discount line-ellipsis" v-if="item.extraDesc">{{ item.extraDesc }}</view>
                    </view>

                    <view class="button-block">
                        <view class="button">购票</view>
                    </view>
                </view>
            </view>
            <view v-else class="no-seat">
                <image src="/static/assets/images/cinema2.png"></image>
                <text>{{ movie.globalReleased ? '今日场次已映完' : '影片未上映' }}</text>
            </view>
        </view>
        <view class="tuan-list" v-if="divideDealList.length">
            <view class="title">影院超值套餐</view>
            <view class="tuan-item" v-for="(item, index) in divideDealList" :key="item.title">
                <view class="snack-item" @tap="goSnackPage" :data-info="i" v-for="(i, index1) in item.dealList" :key="index1">
                    <view>
                        <image :src="i.imageUrl"></image>
                    </view>

                    <view class="snack-info">
                        <view class="first-title">{{ i.firstTitle }}</view>
                        <view class="second-title line-ellipsis">{{ i.secondTitle }}</view>
                        <view class="snack-price">
                            <view>
                                <text class="num">{{ i.price }}</text>
                                <text class="rmb">元</text>
                                <text class="font">影院价:{{ i.value }}元</text>
                            </view>
                            <view class="font">{{ i.curNumberDesc }}</view>
                        </view>
                    </view>
                </view>
            </view>
        </view>
    </view>
</template>

<script>
import selectTime from '@/components/select-time/select-time';
import selectMovie from '@/components/select-movie/select-movie';
import cinemaMap from '../../../templates/cinemaMap/cinemaMap.vue';
const util = require('../../../utils/util.js');
const formatNumber = util.formatNumber;
const getRandom = util.getRandom;
export default {
    components: {
        selectTime,
        selectMovie,
        cinemaMap
    },
    data() {
        return {
            cinemaId: '',
            movieId: '',
            cinemaDetail: null,

            //影院详情
            movie: null,

            //选中的电影
            movies: null,

            //电影列表
            days: [],

            //该电影的排片日期列表
            timeList: [],

            //当天播放电影的时间段
            divideDealList: [],

            //影院分类零食列表
            //只在第一次提示
            first: true,

            day: '',

            i: {
                imageUrl: '',
                firstTitle: '',
                secondTitle: '',
                price: '',
                value: '',
                curNumberDesc: ''
            }
        };
    },
    onLoad(query) {
        this.initPage(query);
    },
    methods: {
        //初始化页面
        initPage(query) {
            const { cinemaId = '', movieId = '', day = '' } = query;
            this.setData({
                cinemaId,
                movieId,
                day
            });
            const _this = this;
            uni.showLoading({
                title: '正在加载...'
            });
            uni.request({
                url: `https://m.maoyan.com/ajax/cinemaDetail?cinemaId=${cinemaId}&movieId=${movieId}`,
                success(res) {
                    uni.hideLoading();
                    _this.setData({
                        cinemaDetail: res.data,
                        movies: _this.formatMovie(res.data.showData.movies),
                        divideDealList: _this.formatUrl(res.data.dealList.divideDealList)
                    });
                }
            });
        },

        //选择电影
        selectMovie(e) {
            const movie = e.detail.movie;
            let days = [];
            movie.shows.forEach((item) => {
                days.push({
                    title: item.dateShow,
                    day: item.showDate
                });
            });
            this.setData({
                movie,
                days,
                timeList: this.createEndTime(movie.shows[0].plist, movie.dur)
            });
        },

        //选择时间
        selectDay(e) {
            const day = e.detail.day;
            const movie = this.movie;
            const index = movie.shows.findIndex((item) => item.showDate === day);
            this.setData({
                timeList: this.createEndTime(movie.shows[index].plist, movie.dur)
            });
        },

        //跳转到“套餐详情”页面
        goSnackPage(e) {
            const info = e.currentTarget.dataset.info;
            //将参数转化为JSON通过页面跳转时传递
            const paramsStr = JSON.stringify({
                cinemaName: this.cinemaDetail.cinemaData.nm,
                cinemaId: this.cinemaId,
                dealId: info.dealId,
                cinemaData: this.cinemaDetail.cinemaData //影院信息
            });

            uni.navigateTo({
                url: `/pages/subPages/snack-page/snack-page?paramsStr=${encodeURIComponent(paramsStr)}`
            });
        },

        //购票
        buyTicket(e) {
            const info = e.currentTarget.dataset.info;
            const { movie, cinemaId, cinemaDetail, first } = this;
            //添加订单信息
            const paramsStr = JSON.stringify({
                cinemaName: cinemaDetail.cinemaData.nm,
                //电影院名
                cinemaId: cinemaId,
                //电影院ID
                hall: info.th,
                //大厅
                movieName: movie.nm,
                //电影名
                movieImg: movie.img,
                //海报
                lang: info.lang + info.tp,
                //语言
                time: `${info.dt} ${info.tm}`,
                //时间
                price: (info.vipPrice && info.vipPrice * 1 + 10) || 37,
                //票价
                seat: `${getRandom(1, 21, true)}排${getRandom(1, 21, true)}座`,
                //座位
                Vcode: getRandom(100000, 999999),
                //模拟6位数的验证码
                flowNumber: getRandom(100000000, 999999999),
                //模拟9位数的流水号,
                orderId: getRandom(1000000000, 9999999999),
                //模拟10位数的订单号,
                cinemaData: cinemaDetail.cinemaData //影院信息
            });
            // 只提示一次
            if (first) {
                uni.showModal({
                    title: '提示',
                    content: '此小程序仅为学习，不会产生任何支付',
                    success: (res) => {
                        this.setData({
                            first: false
                        });
                        if (res.confirm) {
                            uni.navigateTo({
                                url: `/pages/subPages/buy-ticket/buy-ticket?paramsStr=${encodeURIComponent(paramsStr)}`
                            });
                        }
                    }
                });
            } else {
                uni.navigateTo({
                    url: `/pages/subPages/buy-ticket/buy-ticket?paramsStr=${encodeURIComponent(paramsStr)}`
                });
            }
        },

        //处理散场时间
        createEndTime(arr, dur) {
            let timeList = [];
            if (Array.isArray(arr)) {
                timeList = arr.map((item) => {
                    let temp = {
                        ...item
                    };
                    let time = new Date(`${item.dt} ${item.tm}`);
                    time = time.setMinutes(time.getMinutes() + dur);
                    const endTime = new Date(time);
                    temp.endTime = `${formatNumber(endTime.getHours())}:${formatNumber(endTime.getMinutes())}`;
                    return temp;
                });
            }
            return timeList;
        },

        //处理电影图片的url
        formatMovie(arr) {
            let list = [];
            if (Array.isArray(arr)) {
                arr.forEach((item) => {
                    list.push({
                        ...item,
                        img: item.img.replace('w.h', '148.208')
                    });
                });
            }
            return list;
        },

        //处理零食图片的url
        formatUrl(arr) {
            let divideDealList = [];
            if (Array.isArray(arr)) {
                arr.forEach((item) => {
                    let temp = {
                        ...item
                    };
                    temp.dealList = temp.dealList.map((i) => ({
                        ...i,
                        imageUrl: i.imageUrl.replace('w.h', '440.0')
                    }));
                    divideDealList.push(temp);
                });
            }
            return divideDealList;
        }
    }
};
</script>

<style>
/* @import '../../../templates/cinemaMap/cinemaMap.css'; */
.movie-info {
    padding: 22rpx 30rpx;
    text-align: center;
    border-bottom: 1px solid #e6e6e6;
}

.grade {
    color: #ffb400;
    font-size: 32rpx;
}

.small {
    font-size: 20rpx;
}

.movie-desc {
    height: 37rpx;
    margin-top: 2px;
    line-height: 37rpx;
    font-size: 26rpx;
    color: #999;
}

.no-seat {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 460rpx;
}

.no-seat image {
    width: 155rpx;
    height: 143rpx;
}

.no-seat text {
    margin-top: 24rpx;
    line-height: 1;
    font-size: 32rpx;
    color: #acacac;
}

.item {
    display: flex;
    height: 84rpx;
    padding: 30rpx 30rpx 30rpx 0;
    margin: 0 0 0 30rpx;
    border-bottom: 1px solid #e6e6e6;
    color: #333;
}

.item:last-child {
    border: none;
}

.box {
    display: flex;
    height: 84rpx;
    flex-direction: column;
    justify-content: space-between;
}

.begin {
    line-height: 1;
    font-size: 40rpx;
    color: #333;
    white-space: nowrap;
}

.end {
    margin-left: 1px;
    color: #999;
    font-size: 22rpx;
    white-space: nowrap;
}

.info-block {
    margin-left: 34rpx;
    width: 180rpx;
    overflow-x: hidden;
}

.lan {
    margin-top: 1px;
    font-size: 26rpx;
    color: #333;
}

.hall {
    font-size: 22rpx;
    color: #999;
}

.price {
    /* flex: 0 1 auto; */
    width: 260rpx;
    margin-left: 20rpx;
}

.price-box {
    display: flex;
    /* flex-wrap: wrap; */
    height: 40rpx;
    align-items: flex-end;
}

.sellPr {
    line-height: 1;
    color: #f03d37;
    margin-top: 1px;
    font-size: 38rpx;
    margin-right: 10rpx;
}

.d {
    font-size: 22rpx;
}

.vipPrice {
    height: 25rpx;
    line-height: 25rpx;
    margin-top: 1px;
    border: 1px solid #ff9000;
    border-radius: 2px;
    font-size: 22rpx;
}

.vipPrice .icon {
    display: inline-block;
    font-size: 22rpx;
    padding: 0 2px;
    color: #fff;
    background-color: #f90;
}

.vipPrice .num {
    display: inline-block;
    font-size: 22rpx;
    padding: 0 2px;
    color: #f90;
    background-color: #fff;
}

.discount {
    color: #999;
    font-size: 22rpx;
}

.button-block {
    display: flex;
    align-items: center;
    margin-left: 34rpx;
}

.button {
    width: 90rpx;
    height: 54rpx;
    line-height: 54rpx;
    text-align: center;
    box-sizing: border-box;
    background-color: #f03d37;
    color: #fff;
    border-radius: 8rpx;
    white-space: nowrap;
    font-size: 24rpx;
    border: none;
}

.tuan-list {
    padding-left: 30rpx;
    border-top: 20rpx solid #f5f5f5;
}

.tuan-list .title {
    height: 90rpx;
    line-height: 90rpx;
    font-size: 30rpx;
    color: #999;
}

.snack-item {
    display: flex;
    width: 100%;
    box-sizing: border-box;
    border-top: 1px solid #e6e6e6;
    padding: 26rpx 30rpx 26rpx 0;
}

.snack-item image {
    width: 160rpx;
    height: 160rpx;
    margin-right: 20rpx;
}

.snack-info {
    display: flex;
    flex-direction: column;
    justify-content: space-around;
    flex-grow: 1;
    overflow: hidden;
}

.first-title {
    font-size: 30rpx;
    color: #666;
}

.second-title {
    width: 90%;
    font-size: 24rpx;
    color: #999;
}

.snack-price {
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    color: #999;
}

.num {
    font-size: 30rpx;
    color: #f03d37;
}

.rmb {
    color: #f03d37;
}

.font {
    font-size: 24rpx;
}
</style>
