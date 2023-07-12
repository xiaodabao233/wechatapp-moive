<template>
    <view class="container">
        <view class="banner">
            <image src="/static/assets/images/youcan.jpg" mode="aspectFill"></image>
        </view>
        <view class="section">
            <view v-for="(item, index) in info" :key="item.label">
                <text>{{ item.label }}：</text>

                <text>{{ item.value }}</text>
            </view>
            <view></view>
        </view>
        <view class="section">
            <text>基于Uni-app快速开发电影购票的小程序，取名为目微电影，代码托管在GitHub仓库。</text>
        </view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            info: [
                {
                    label: '项目：电影购票',
                    value: '基于Uni-app快速开发的小程序'
                },
                {
                    label: 'Coder-01',
                    value: '文烨'
                },
                {
                    label: 'Coder-02',
                    value: '王洪伟'
                },
                {
                    label: 'Coder-03',
                    value: '周成波'
                },
                {
                    label: 'github仓库',
                    value: 'https://github.com/xiaodabao233/wechatapp-moive'
                }
            ]
        };
    },
    methods: {}
};
</script>
<style>
.container {
    width: 100%;
    height: 100vh;
    background: #f7f7f7;
    overflow: auto;
}

.banner {
    height: 375rpx;
    margin: 20rpx;
    box-sizing: border-box;
    background-color: #eee;
    line-height: 0;
    border-radius: 10rpx;
    overflow: hidden;
    box-shadow: 2px 2px 2px 1px #ddd;
}

.banner image {
    width: 100%;
    height: 100%;
}

.section {
    background-color: #fff;
    margin: 0 20rpx 20rpx;
    border-radius: 10rpx;
    padding: 20rpx;
    box-shadow: 2px 2px 2px 1px #ddd;
}

.section text {
    font-size: 28rpx;
    line-height: 2em;
    color: #666;
}
</style>
