<template>
    <view style="height: 100%">
        <template v-if="compName === 'nothing'">
            <view class="nothing">
                <image src="/static/assets/images/nothing.png" class="nothing-img"></image>
                <view class="nothing-text">暂无订单！</view>
            </view>
        </template>
    </view>
</template>

<script>
export default {
    props: ['data', 'compName'],
    computed: {
        message() {
            return this.data.message;
        }
    }
};
</script>
<style>
.nothing {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    text-align: center;
}

.nothing-img {
    width: 174rpx;
    height: 200rpx;
    margin-bottom: 16px;
}

.nothing-text {
    font-size: 32rpx;
    color: #999;
}
</style>
