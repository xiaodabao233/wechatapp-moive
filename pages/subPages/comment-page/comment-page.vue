<template>
    <view>
        <view class="hot" v-if="hcmts.length">
            <view class="comment-title">热门评论</view>
            <view>
                <commentSection compName="commentSection" :data="{ comment }" v-for="(comment, index) in hcmts" :key="comment.id"></commentSection>
            </view>
        </view>
        <view class="hot" v-if="cmts.length">
            <view class="comment-title">最新评论</view>
            <view>
                <commentSection compName="commentSection" :data="{ comment }" v-for="(comment, index) in cmts" :key="comment.id"></commentSection>
            </view>
        </view>
        <view class="loadingMore" v-if="!loadComplete && cmts.length && hcmts.length">
            <view class="loading-text">加载更多...</view>
        </view>
    </view>
</template>

<script>
import commentSection from '../../../templates/commentSection/commentSection.vue';
const util = require('../../../utils/util.js');
export default {
    components: {
        commentSection
    },
    data() {
        return {
            movieId: '',
            cmts: [],

            //最新评论
            hcmts: [],

            //热门评论
            //是否加载完
            loadComplete: false,

            comment: {
                id: ''
            }
        };
    },
    onLoad(options) {
        this.initPage(options);
    },
    onReachBottom() {
        this.getComment(this.movieId);
    },
    methods: {
        //初始化页面
        initPage(options) {
            const movieId = options.movieId;
            const movieName = options.movieName;
            uni.setNavigationBarTitle({
                title: `观众评论-${movieName}`
            });
            uni.showLoading({
                title: '正在加载...'
            });
            this.getComment(movieId);
            this.setData({
                movieId
            });
        },

        //获取观众评论
        getComment(movieId) {
            if (this.loadComplete) {
                return;
            }
            const cmts = this.cmts;
            const _this = this;
            uni.request({
                url: `https://m.maoyan.com/mmdb/comments/movie/${movieId}.json?_v_=yes&offset=${cmts.length}`,
                success(res) {
                    let comments = {
                        ...res.data
                    };
                    const newCmts = cmts.concat(_this.formatData(comments.cmts));
                    uni.hideLoading();
                    _this.setData({
                        hcmts: _this.formatData(comments.hcmts),
                        cmts: newCmts,
                        loadComplete: newCmts.length >= comments.total
                    });
                }
            });
        },

        //处理数据
        formatData(arr) {
            let list = [];
            if (arr.length) {
                list = arr.map((item) => {
                    let temp = {
                        ...item
                    };
                    temp.avatarurl = temp.avatarurl || '/static/assets/images/avatar.png';
                    temp.purchase = !!(temp.tagList && temp.tagList.fixed.some((item) => item.id === 4));
                    temp.stars = this.formatStar(temp.score);
                    temp.calcTime = util.calcTime(temp.startTime);
                    return temp;
                });
            }
            return list;
        },

        //处理评分星星
        formatStar(sc) {
            //1分对应满星，0.5对应半星
            let stars = new Array(5).fill('star-empty');
            const fullStars = Math.floor(sc); //满星的个数
            if (sc % 1) {
            } else {
            } //半星的个数，半星最多1个
            stars.fill('star-full', 0, fullStars); //填充满星
            if (fullStars < 5) {
                stars[fullStars] = halfStar; //填充半星
            }

            return stars;
        }
    }
};
</script>

<style>
/* @import '../../../templates/commentSection/commentSection.css'; */
.comment-title {
    height: 60rpx;
    font-size: 28rpx;
    background: #f5f5f5;
    color: #666;
    line-height: 60rpx;
    padding-left: 30rpx;
}
.loading-text {
    margin-top: 0;
    margin-bottom: 8rpx;
    padding: 10rpx 0;
    background: #f5f5f5;
}
</style>
