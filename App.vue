<script>
//app.js
const QQMapWX = require('./assets/libs/qqmap-wx-jssdk.min.js');
let qqmapsdk;
qqmapsdk = new QQMapWX({
    key: 'MH2BZ-4WTK3-2D63K-YZRHP-HM537-HHBD3'
});
export default {
    data() {
        return {};
    },
    onLaunch: function () {
        this.globalData.initPage();
    },
    globalData: {
        userLocation: null,

        //用户的位置信息
        //用户切换的城市
        selectCity: null,

        initPage() {
            // 获取用户授权信息信息,防止重复出现授权弹框
            uni.getSetting({
                success: (res) => {
                    //已有权限直接获得信息，否则出现授权弹框
                    if (res.authSetting['scope.userLocation']) {
                        this.getUserLocation();
                    } else {
                        this.getUserLocation();
                    }
                }
            });
        },

        //获取用户的位置信息
        getUserLocation() {
            uni.getLocation({
                //成功授权
                success: (res) => {
                    const latitude = res.latitude;
                    const longitude = res.longitude;
                    // 使用腾讯地图接口将位置坐标转出成名称（为什么弹框出出现两次？）
                    qqmapsdk.reverseGeocoder({
                        location: {
                            //文档说location默认为当前位置可以省略，但是还是要手动加上，否则弹框会出现两次，手机端则出现问题
                            latitude,
                            longitude
                        },
                        success: (res) => {
                            const cityFullname = res.result.address_component.city;
                            const cityInfo = {
                                latitude,
                                longitude,
                                cityName: cityFullname.substring(0, cityFullname.length - 1),
                                status: 1
                            };
                            this.userLocation = {
                                ...cityInfo
                            }; //浅拷贝对象
                            this.selectCity = {
                                ...cityInfo
                            }; //浅拷贝对象
                            // 由于 getUserInfo 是网络请求，可能会在 Page.onLoad 之后才返回，所以此处加入 callback 以防止这种情况
                            if (this.userLocationReadyCallback) {
                                this.userLocationReadyCallback();
                            }
                        }
                    });
                },
                fail: () => {
                    this.userLocation = {
                        status: 0
                    };
                    //防止当弹框出现后，用户长时间不选择，
                    if (this.userLocationReadyCallback) {
                        this.userLocationReadyCallback();
                    }
                }
            });
        }
    }
};
</script>
<style>
@import './assets/font/icon.css';
/* @import './templates/loadingMore/loadingMore.css'; */
/* @import './templates/nothing/nothing.css'; */
/* 因为将模板的样式直接引入到了app.wxss，所以注意上面的样式的污染 */

.line-ellipsis {
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
}
.scroll-view_H {
    white-space: nowrap;
    width: 100%;
}
/* topbar */
.topbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 88rpx;
    border-bottom: 1px solid #e6e6e6;
}
.city-entry {
    padding-left: 30rpx;
    font-size: 30rpx;
    color: #666;
    display: -webkit-box;
    display: flex;
    -webkit-box-align: center;
    align-items: center;
}
.city-name {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    max-width: 19.2vw;
}

.city-entry-arrow {
    width: 0;
    height: 0;
    border: 8rpx solid #b0b0b0;
    border-left-color: transparent;
    border-right-color: transparent;
    border-bottom-color: transparent;
    display: inline-block;
    margin-left: 8rpx;
    margin-top: 10rpx;
}
.phcolor {
    color: #999;
}
.buy-tickets .btn {
    width: 94rpx;
    height: 56rpx;
    line-height: 56rpx;
    text-align: center;
    box-sizing: border-box;
    background-color: #f03d37;
    color: #fff;
    border-radius: 8rpx;
    white-space: nowrap;
    font-size: 24rpx;
    border: none;
}

.buy-tickets .btn.pre-sale {
    background-color: #3c9fe6;
}
.buy-tickets .want-see {
    background-color: #faaf00;
}
</style>
