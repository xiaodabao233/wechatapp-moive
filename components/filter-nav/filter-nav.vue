<template>
    <view class="nav">
        <view class="tab">
            <view :class="'nav-item ' + (itemNum === 1 ? 'select-item' : '')" @tap="selectItemNum" :data-item-num="1">
                <text class="title line-ellipsis">{{ itemName1 }}</text>
                <text class="city-entry-arrow"></text>
            </view>
            <view :class="'nav-item have-border ' + (itemNum === 2 ? 'select-item' : '')" @tap="selectItemNum" :data-item-num="2">
                <text class="title line-ellipsis">{{ itemName2 }}</text>
                <text class="city-entry-arrow"></text>
            </view>
            <view :class="'nav-item ' + (itemNum === 3 ? 'select-item' : '')" @tap="selectItemNum" :data-item-num="3">
                <text class="title line-ellipsis">{{ itemName3 }}</text>
                <text class="city-entry-arrow"></text>
            </view>
        </view>
        <view class="nav-content">
            <view class="nav-content-item region" :style="'display:' + (itemNum === 1 ? 'block' : 'none')">
                <view class="tab">
                    <view :class="'nav-item ' + (selectRegion.item === 0 ? 'active' : '')" @tap="selectRegionItem" :data-index="0">商区</view>
                    <view :class="'nav-item ' + (selectRegion.item === 1 ? 'active' : '')" @tap="selectRegionItem" :data-index="1">地铁站</view>
                </view>
                <view class="region-list">
                    <scroll-view class="region-sidenav" scroll-y>
                        <view
                            :class="
                                'line-ellipsis side-item ' +
                                (selectRegion.item === 0 ? (item.id === selectRegion.selectDistrictId ? 'active' : '') : item.id === selectRegion.selectLineId ? 'active' : '')
                            "
                            @tap="regionSideClick"
                            :data-side="item"
                            v-for="(item, index) in selectRegion.sideList"
                            :key="item.id"
                        >
                            {{ item.name }}({{ item.count }})
                        </view>
                    </scroll-view>
                    <scroll-view class="region-list-item" scroll-y>
                        <view
                            :class="
                                'item ' + (selectRegion.item === 0 ? (item.id === selectRegion.selectAreaId ? 'red' : '') : item.id === selectRegion.selectStationId ? 'red' : '')
                            "
                            @tap="regionListClick"
                            :data-item="item"
                            v-for="(item, index) in selectRegion.list"
                            :key="item.id"
                        >
                            <view>
                                <text
                                    class="iconfont icon-hook"
                                    :style="
                                        'visibility:' +
                                        (selectRegion.item === 0
                                            ? item.id === selectRegion.selectAreaId
                                                ? ''
                                                : 'hidden'
                                            : item.id === selectRegion.selectStationId
                                            ? ''
                                            : 'hidden')
                                    "
                                ></text>
                                {{ item.name }}
                            </view>

                            <view>{{ item.count }}</view>
                        </view>
                    </scroll-view>
                </view>
            </view>
            <view class="nav-content-item brand" :style="'display:' + (itemNum === 2 ? 'block' : 'none')">
                <scroll-view class="brand-scroll-view" scroll-y>
                    <view
                        :class="'brand-item ' + (selectBrandId === item.id ? 'red' : '')"
                        @tap="selectBrand"
                        :data-brand="item"
                        v-for="(item, index) in cityCinemaInfo.brand.subItems"
                        :key="item.id"
                    >
                        <view>
                            <text class="iconfont icon-hook" :style="'visibility:' + (selectBrandId === item.id ? '' : 'hidden')"></text>
                            {{ item.name }}
                        </view>

                        <view class="brand-count">{{ item.count }}</view>
                    </view>
                </scroll-view>
            </view>
            <view class="nav-content-item special" :style="'display:' + (itemNum === 3 ? 'block' : 'none')">
                <scroll-view class="special-scroll-view" scroll-y>
                    <view class="item-title">特色功能</view>
                    <view class="item-list">
                        <view
                            :class="'btn line-ellipsis ' + (selectServiceId === item.id ? 'select' : '')"
                            @tap="specialSelectItem"
                            :data-type-id="item.id"
                            data-type="service"
                            v-for="(item, index) in cityCinemaInfo.service.subItems"
                            :key="item.id"
                        >
                            {{ item.name }}
                        </view>
                    </view>
                    <view class="item-title">特殊厅</view>
                    <view class="item-list">
                        <view
                            :class="'btn line-ellipsis ' + (selectHallTypeId === item.id ? 'select' : '')"
                            @tap="specialSelectItem"
                            :data-type-id="item.id"
                            data-type="hallType"
                            v-for="(item, index) in cityCinemaInfo.hallType.subItems"
                            :key="item.id"
                        >
                            {{ item.name }}
                        </view>
                    </view>
                </scroll-view>
                <view class="special-btn">
                    <view class="btn" @tap="specialReset">重置</view>
                    <view class="btn confirm-btn" @tap="specialConfirm">确定</view>
                </view>
            </view>
        </view>
        <view class="mask" @tap="cancal" :style="'display:' + (itemNum === -1 ? 'none' : 'block')" @tap.stop.prevent="cancal"></view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            itemNum: -1,
            itemName1: '全城',
            itemName2: '品牌',
            itemName3: '特色',
            selectBrandId: -1,
            //选择的品牌ID
            selectServiceId: -1,
            //选择的服务ID
            selectHallTypeId: -1,
            //选择的特殊厅ID
            selectRegion: {
                item: 0,
                sideList: [],
                //侧边导航的list
                list: [],
                //详情list
                selectDistrictId: -1,
                //选择的大区ID
                selectAreaId: -1,
                //选择的小区ID
                selectLineId: -1,
                //选择的地铁线ID
                selectStationId: -1 //选择的地铁站ID
            }
        };
    },
    /**
     * 组件的属性列表
     */
    props: {
        cityCinemaInfo: {
            type: Object,
            default: () => ({})
        },
        hidden: {
            type: Boolean,
            default: true
        }
    },
    created() {
        //自己实现的一个watch。因为不能在外面直接写watch，所以只能定义在这里
        const watch = {
            itemNum: (value) => {
                this.$emit('toggleShow', {
                    detail: {
                        item: value
                    }
                });
            }
        };
        this.setWatcherFun(this, watch);
    },
    /**
     * 组件的方法列表
     */
    methods: {
        attached() {},

        //导航栏的点击事件
        selectItemNum(e) {
            const itemNum = e.currentTarget.dataset.itemNum;
            let num = itemNum;
            if (this.itemNum !== -1) {
                num = itemNum === this.itemNum ? -1 : itemNum;
            }
            this.setData({
                itemNum: num
            });
        },

        //蒙板的点击事件
        cancal() {
            this.setData({
                itemNum: -1
            });
        },

        //选择品牌的点击事件
        selectBrand(e) {
            const brand = e.currentTarget.dataset.brand;
            let brandName = brand.name;
            if (brand.id === -1) {
                brandName = '品牌';
            }
            this.$emit('change', {
                detail: {
                    brandId: brand.id
                }
            });
            this.setData({
                selectBrandId: brand.id,
                itemName2: brandName,
                itemNum: -1
            });
        },

        //特色重置按钮
        specialReset() {
            this.setData({
                selectServiceId: -1,
                selectHallTypeId: -1
            });
        },

        //特色选择按钮
        specialSelectItem(e) {
            const { type, typeId } = e.currentTarget.dataset;
            if (type === 'service') {
                this.setData({
                    selectServiceId: typeId
                });
            } else {
                this.setData({
                    selectHallTypeId: typeId
                });
            }
        },

        //特色确定按钮
        specialConfirm() {
            const { selectServiceId, selectHallTypeId } = this;
            this.$emit('change', {
                detail: {
                    serviceId: selectServiceId,
                    hallType: selectHallTypeId
                }
            });
            this.setData({
                itemNum: -1
            });
        },

        //“全城”的item点击事件
        selectRegionItem(e) {
            const index = e.currentTarget.dataset.index;
            const cityCinemaInfo = this.cityCinemaInfo;
            let obj = {
                ...this.selectRegion
            };
            if (index === 0) {
                obj.item = 0;
                obj.sideList = cityCinemaInfo.district.subItems;
                const findItem = obj.sideList.find((item) => item.id === obj.selectDistrictId);
                obj.list = findItem.subItems ? findItem.subItems : [];
            } else {
                obj.item = 1;
                obj.sideList = cityCinemaInfo.subway.subItems;
                const findItem = obj.sideList.find((item) => item.id === obj.selectLineId);
                obj.list = findItem.subItems ? findItem.subItems : [];
            }
            this.setData({
                selectRegion: obj
            });
        },

        //“全城”的side的点击事件
        regionSideClick(e) {
            const { item, selectDistrictId, selectLineId, selectStationId, selectAreaId } = this.selectRegion;
            const side = e.currentTarget.dataset.side;
            let obj = {
                ...this.selectRegion,
                list: side.subItems ? side.subItems : []
            };
            if (item === 0) {
                //点击“全部”时关闭下拉框
                if (side.id === -1 && selectDistrictId !== -1) {
                    this.$emit('change', {
                        detail: {
                            districtId: -1,
                            lineId: selectLineId,
                            areaId: -1,
                            stationId: selectStationId
                        }
                    });
                    this.setData({
                        itemNum: -1,
                        itemName1: '全城',
                        selectRegion: {
                            ...this.selectRegion,
                            selectDistrictId: -1,
                            selectAreaId: -1,
                            list: []
                        }
                    });
                    return;
                }
                obj.selectDistrictId = side.id;
                obj.list = side.subItems ? side.subItems : [];
            } else {
                if (side.id === -1 && selectLineId !== -1) {
                    this.$emit('change', {
                        detail: {
                            districtId: selectDistrictId,
                            lineId: -1,
                            areaId: selectAreaId,
                            stationId: -1
                        }
                    });
                    this.setData({
                        itemNum: -1,
                        itemName1: '全城',
                        selectRegion: {
                            ...this.selectRegion,
                            selectLineId: -1,
                            selectStationId: -1,
                            list: []
                        }
                    });
                    return;
                }
                obj.selectLineId = side.id;
            }
            this.setData({
                selectRegion: obj
            });
        },

        //“全城”详细list的点击事件
        regionListClick(e) {
            const item = e.currentTarget.dataset.item;
            let obj = {
                ...this.selectRegion
            };
            if (this.selectRegion.item === 0) {
                obj.selectAreaId = item.id;
            } else {
                obj.selectStationId = item.id;
            }
            this.$emit('change', {
                detail: {
                    districtId: obj.selectDistrictId,
                    lineId: obj.selectLineId,
                    areaId: obj.selectAreaId,
                    stationId: obj.selectStationId
                }
            });
            this.setData({
                selectRegion: obj,
                itemNum: -1,
                itemName1: item.name
            });
        },

        //简单实现类似vue的watch
        setWatcherFun(data, watch) {
            Object.keys(watch).forEach((key) => {
                this.observeFun(data, key, watch[key]);
            });
        },

        observeFun(obj, key, func) {
            let val = obj[key];
            Object.defineProperty(obj, key, {
                configurable: true,
                enumerable: true,
                set: function (newVal) {
                    if (newVal === val) {
                        return;
                    }
                    val = newVal;
                    func(newVal); // 赋值(set)时，调用对应函数
                },

                get: function () {
                    return val;
                }
            });
        }
    },
    mounted() {
        // 处理小程序 attached 生命周期
        this.attached();
    },
    // watch: {
    //     cityCinemaInfo: {
    //         handler: function (newVal, oldVal, changedPath) {
    //             if (newVal.district) {
    //                 const sideList = newVal.district.subItems;
    //             } else {
    //                 const sideList = [];
    //             }
    //             this.setData({
    //                 selectRegion: {
    //                     ...this.selectRegion,
    //                     sideList
    //                 }
    //             });	
    //         },

    //         immediate: true,
    //         deep: true
    //     },

    //     hidden: {
    //         handler: function (newVal) {
    //             if (newVal) {
    //                 this.setData({
    //                     itemNum: -1
    //                 });
    //             }
    //         },

    //         immediate: true
    //     }
    // }
};
</script>
<style>
@import '../../assets/font/icon.css';

.line-ellipsis {
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
}
.nav {
    font-size: 28rpx;
}
.tab {
    display: flex;
    align-items: center;
    width: 100vw;
    height: 80rpx;
    border-bottom: 1px solid #e6e6e6;
    color: #555;
    background: #fff;
}

.nav-item {
    flex-grow: 1;
    text-align: center;
    font-size: 28rpx;
}
.tab .nav-item {
    width: 33.33%;
}
.nav-item .title {
    display: inline-block;
    max-width: 80%;
}

.have-border {
    border-left: 1px solid #e6e6e6;
    border-right: 1px solid #e6e6e6;
}

.city-entry-arrow {
    width: 0;
    height: 0;
    border: 8rpx solid #b0b0b0;
    border-left-color: transparent;
    border-right-color: transparent;
    border-bottom-color: transparent;
    display: inline-block;
    vertical-align: text-top;
    margin-left: 8rpx;
    margin-top: 10rpx;
}

.select-item {
    color: #e54847;
}

.select-item .city-entry-arrow {
    position: relative;
    transform: translateY(-8rpx);
    border: 8rpx solid #e54847;
    border-left-color: transparent;
    border-right-color: transparent;
    border-top-color: transparent;
}

.nav-content-item {
    position: relative;
    width: 100vw;
    height: calc(100vh * 0.6);
    background: #fff;
    color: #777;
}

/* region */

.region .tab {
    border: none;
}

.region .nav-item {
    height: 100%;
    line-height: 80rpx;
    box-sizing: border-box;
}

.region .nav-item.active {
    color: #e54847;
    border-bottom: 2px solid #e54847;
}

.region-list {
    display: flex;
    width: 100%;
    height: calc(100% - 80rpx);
}

.region-sidenav {
    white-space: nowrap;
    /* 直接用百分比有点问题 */
    width: calc(100vw * 0.4);
    height: 100%;
}
.side-item {
    height: 80rpx;
    line-height: 80rpx;
    padding-left: 30rpx;
}
.side-item.active {
    color: #dd403b;
    background: #f5f5f5;
}

.region-list-item {
    white-space: nowrap;
    flex-grow: 1;
    height: 100%;
    background: #f5f5f5;
}
.region-list-item .item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-left: 30rpx;
    padding-right: 50rpx;
    height: 80rpx;
}

/* brand */

.brand-scroll-view {
    white-space: nowrap;
    width: 100%;
    height: 100%;
}

.brand-item {
    display: flex;
    justify-content: space-between;
    line-height: 88rpx;
    height: 88rpx;
    border-bottom: 1px solid #f0f0f0;
    padding-left: 45rpx;
    padding-right: 80rpx;
}

.red {
    color: #dd403b;
}

.brand-count {
    font-size: 24rpx;
}

/* special */

.special-scroll-view {
    white-space: nowrap;
    width: 100%;
    height: calc(100% - 120rpx);
}

.special .item-title {
    padding: 20rpx 30rpx;
}

.special .item-list {
    display: flex;
    flex-wrap: wrap;
    padding: 0 30rpx;
}

.special .item-list .btn {
    width: 21.3%;
    height: 60rpx;
    margin-right: 3%;
    margin-bottom: 20rpx;
    line-height: 60rpx;
    text-align: center;
    border-radius: 10rpx;
    font-size: 24rpx;
    box-sizing: border-box;
    border: 1px solid #e5e5e5;
    padding: 0 6rpx;
}

.special .item-list .btn.select {
    background: #fcf0f0;
    color: #f03d37;
    border: 1px solid #f03d37;
}

.special-btn {
    position: absolute;
    bottom: 0;
    left: 0;
    width: 100%;
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 120rpx;
    background: #fafafa;
    border-top: 1px solid #e5e5e5;
    padding: 0 30rpx;
    box-sizing: border-box;
}

.special-btn .btn {
    background: #fff;
    border: 1px solid #e5e5e5;
    height: 68rpx;
    width: 20%;
    text-align: center;
    border-radius: 12rpx;
    font-size: 28rpx;
    line-height: 68rpx;
}

.special-btn .btn.confirm-btn {
    background: #f03d37;
    border: 1px solid #f03d37;
    color: #fff;
}

/* 蒙板 */

.mask {
    position: fixed;
    z-index: -1;
    bottom: 0;
    left: 0;
    height: 50%;
    width: 100%;
    background: rgba(0, 0, 0, 0.3);
}
</style>
