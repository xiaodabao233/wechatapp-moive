<template>
    <view class="container">
        <view class="header">
            <open-data type="userAvatarUrl" class="avatar"></open-data>
            <open-data type="userNickName" class="name"></open-data>
        </view>
        <view>
            <view class="order">
                <view class="title-box">
                    <view class="title">我的订单</view>
                    <view class="title-line"></view>
                </view>
                <view class="order-list">
                    <navigator class="order-item" url="../../subPages/movie-order/movie-order" hover-class="none">
                        <image src="/static/assets/images/order_movie.png"></image>
                        <view>电影</view>
                    </navigator>
                    <navigator class="order-item" url="../../subPages/snack-order/snack-order" hover-class="none">
                        <image src="/static/assets/images/order_store.png"></image>
                        <view>小吃</view>
                    </navigator>
                </view>
            </view>
            <view class="other">
                <view class="share other-item">
                    <button open-type="share" class="btn" hover-class="none">
                        <text class="iconfont icon-fenxiang1"></text>
                        分享给朋友
                    </button>
                </view>
                <view class="customer other-item">
                    <button open-type="contact" class="btn" hover-class="none">
                        <text class="iconfont icon-lianxikefu"></text>
                        客服反馈
                    </button>
                </view>
                <view class="about other-item">
                    <navigator url="../../subPages/about-page/about-page" hover-class="none">
                        <text class="iconfont icon-guanyuwomen"></text>
                        关于
                    </navigator>
                </view>
            </view>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {};
    },
    onShareAppMessage(res) {
        return {
            title: '目微电影',
            path: 'pages/tabBar/movie/movie'
        };
    },
    methods: {}
};
</script>

<style>
.container {
    width: 100vw;
    height: 100vh;
    background: #f3f3f3;
}

.header {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background-image: url('https://s0.meituan.net/bs/file/?f=my-canary:deploy/dpmmweb/client/dpmmweb/component/mycenter/img/bg.png');
    background-repeat: no-repeat;
    background-position: 50%;
    background-size: 100% 300rpx;
    height: 300rpx;
    overflow: hidden;
    text-align: center;
}

.header .avatar {
    width: 120rpx;
    height: 120rpx;
    border-radius: 120rpx;
    border: 3px solid #fff;
    overflow: hidden;
}

.header .name {
    height: 45rpx;
    margin-top: 14rpx;
    font-size: 32rpx;
    line-height: 45rpx;
    color: #fff;
}

.order {
    margin-top: 20rpx;
    background: #fff;
    padding: 26rpx 30rpx 0;
}

.order .title-box {
    position: relative;
    width: 320rpx;
    height: 42rpx;
    margin: 0 auto;
}

.order .title {
    position: relative;
    height: 42rpx;
    width: 160rpx;
    margin: 0 auto;
    font-size: 32rpx;
    line-height: 42rpx;
    color: #333;
    text-align: center;
    background: #fff;
    z-index: 9;
}

.order .title-line {
    position: absolute;
    width: 320rpx;
    height: 1px;
    left: 0;
    top: 23rpx;
    background: #f0f0f0;
}

.order-list {
    display: flex;
    height: 230rpx;
    align-items: center;
    background: #fff;
}

.order-item {
    width: 50%;
    text-align: center;
}

.order-item image {
    width: 84rpx;
    height: 84rpx;
    margin-bottom: 6rpx;
}

.order-item view {
    color: #333;
    font-size: 28rpx;
}

.other {
    margin-top: 20rpx;
    background: #fff;
}

.other-item {
    position: relative;
    height: 100rpx;
    line-height: 100rpx;
    font-size: 30rpx;
    margin-left: 30rpx;
    padding-right: 30rpx;
    color: #333;
    border-bottom: 1px solid #f0f0f0;
}

.other-item::after {
    position: absolute;
    right: 30rpx;
    top: 50%;
    content: '';
    width: 16rpx;
    height: 16rpx;
    border-left: 1px solid #ccc;
    border-top: 1px solid #ccc;
    transform: translateY(-50%) rotate(135deg);
}
/* 覆盖按钮的默认样式 */
.other .btn {
    width: 100%;
    border: none;
    border-radius: 0;
    padding: 0;
    margin: 0;
    font-size: 1em;
    position: static;
    line-height: normal;
    color: #333;
    text-align: left;
    background: #fff;
}
.other .btn::after {
    content: none;
}
.other .iconfont {
    display: inline-block;
    line-height: 100rpx;
    margin-right: 8rpx;
    font-size: 32rpx;
    color: #333;
}
.customer .iconfont {
    font-size: 38rpx;
}
.about .iconfont {
    opacity: 0.8;
}
</style>
