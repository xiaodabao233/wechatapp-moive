<template>
    <view style="height: 100%">
        <template v-if="compName === 'loadingMore'">
            <view class="loadingMore">
                <image src="/static/assets/images/loading.gif" class="loading-img"></image>
                <view class="loading-text">正在加载...</view>
            </view>
        </template>
    </view>
</template>

<script>
export default {
    props: ['data', 'compName'],
    computed: {}
};
</script>
<style>
.loadingMore {
    text-align: center;
    margin-top: 18rpx;
}
.loading-img {
    width: 54rpx;
    height: 54rpx;
}
.loading-text {
    margin-top: 8rpx;
    color: #666;
    font-size: 28rpx;
}
</style>
