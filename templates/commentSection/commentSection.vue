<template>
    <view style="height: 100%">
        <template v-if="compName === 'commentSection'">
            <view class="commentSection">
                <view><image :src="comment.avatarurl" class="avatar"></image></view>
                <view class="left">
                    <view class="nickName">
                        <text>{{ comment.nick || comment.nickName }}</text>
                        <text v-if="comment.purchase" class="buy">购</text>
                    </view>
                    <view class="stars">
                        <image
                            class="movie-star"
                            :src="'/assets/images/' + item + '/static/templates/commentSection/.png'"
                            mode="aspectFill"
                            v-for="(item, index) in comment.stars"
                            :key="index"
                        ></image>
                    </view>
                    <view class="content">{{ comment.content }}</view>
                    <view class="footer">
                        <view class="comment-time">{{ comment.calcTime }}</view>
                        <view class="approve">
                            <text class="iconfont icon-zan1"></text>
                            <text>{{ comment.approve ? comment.approve : '赞' }}</text>
                        </view>
                    </view>
                </view>
            </view>
        </template>
    </view>
</template>

<script>
export default {
    props: ['data', 'compName'],
    computed: {
        comment() {
            return this.data.comment;
        },
        item() {
            return this.data.item;
        },
        index() {
            return this.data.index;
        }
    }
};
</script>
<style>
.commentSection {
    display: flex;
    padding: 30rpx 0 0 30rpx;
    color: #555;
}

.avatar {
    height: 70rpx;
    width: 70rpx;
    border-radius: 50%;
}

.left {
    flex-grow: 1;
    margin-left: 40rpx;
    padding-right: 30rpx;
    border-bottom: 1px solid #e6e6e6;
    padding-bottom: 30rpx;
}
.commentSection:last-child .left {
    border: none;
}

.nickName {
    font-size: 24rpx;
    color: #444;
}

.buy {
    display: inline-block;
    border-radius: 2px;
    color: #fff;
    background: #f90;
    font-size: 22rpx;
    margin-left: 5px;
    padding: 1px 2px;
}

.commentSection .movie-star {
    width: 22rpx;
    height: 22rpx;
    margin-right: 3px;
}

.content {
    font-size: 30rpx;
    margin: 20rpx 0;
}

.footer {
    display: flex;
    justify-content: space-between;
    font-size: 24rpx;
    color: #999;
}
.comment-time {
    font-size: 24rpx;
}
.approve {
    font-size: 24rpx;
}
</style>
