<template>
    <view class="swiper-container">
        <view class="post-bg" :style="'background-image:url(' + movie.img + ')'"></view>
        <view class="post-bg-mask"></view>
        <scroll-view class="swiper-wrapper scroll-view_H" scroll-x scroll-with-animation :scroll-left="scrollLeft">
            <view class="movie-item" :data-movie="item" @tap="selectMovie" :id="'item' + index" v-for="(item, index) in moviesClone" :key="item.id">
                <view :class="'post ' + (movie.id === item.id ? 'select' : '')">
                    <image :src="item.img"></image>
                </view>
            </view>
        </scroll-view>
    </view>
</template>

<script>
export default {
    data() {
        return {
            moviesClone: [],
            movie: null,

            //选中的电影
            scrollLeft: 0,

            //设置滚动条位置
            size: 0,

            //电影item的大小（包括margin）
            //当前电影的索引,下面的函数传递了事件对象后就不能传递其他参数了，所以只能放在data中传递
            i: 0,

            moviesClone: []
        };
    },
    props: {
        movies: {
            type: Array,
            default: () => []
        },
        defaultSelectID: {
            type: String,
            default: ''
        }
    },
    methods: {
        //选中电影
        selectMovie(e) {
            const { moviesClone: movies } = this;
            const movie = (e && e.currentTarget.dataset.movie) || movies.find((item) => item.id == this.defaultSelectID) || movies[this.i];
            if (movies.length && this.movie && movie.id === this.movie.id) {
                return;
            }
            const index = movies.findIndex((item) => item.id === movie.id);
            if (this.size) {
                this.setData({
                    movie,
                    scrollLeft: this.size * index
                });
            } else {
                this.calcSize().then((size) => {
                    this.setData({
                        movie,
                        size,
                        scrollLeft: size * index
                    });
                });
            }
            this.$emit('selectMovie', {
                detail: {
                    movie
                }
            });
        },
        //计算节点大小
        calcSize() {
            return new Promise((resolve, reject) => {
                const query = uni.createSelectorQuery().in(this);
                query
                    .select(`#item1`)
                    .fields(
                        {
                            size: true,
                            computedStyle: ['margin-left']
                        },
                        function (res) {
                            let size = 0;
                            if (res) {
                                size = res.width + parseFloat(res['margin-left']);
                            }
                            resolve(size);
                        }
                    )
                    .exec();
            });
        }
        /**
   * 本来想用touchend事件来做滚动结束后“选择电影”功能，但是获取scrollOffset有两三秒的延迟，所以就放弃了
  handleTouchend(e) {
    const size = this.data.size
    const query = wx.createSelectorQuery().in(this)
    query.select('.swiper-wrapper').scrollOffset((res) => {
      const scrollLeft = res.scrollLeft
      let index = Math.ceil(scrollLeft / size)
      if ((index * size - size / 2) < scrollLeft && (index * size + size / 2)) {
        } else {
        index--
      }
      this.setData({
        i: index
      }, () => {
        this.selectMovie()
      })
    }).exec()
  }
   */
    },
    created: function () {},
    watch: {
        movies: {
            handler: function (movies) {
                this.moviesClone = this.clone(this.movies);
                if (Array.isArray(movies) && movies.length) {
                    this.setData(
                        {
                            moviesClone: movies
                        },
                        () => {
                            this.selectMovie();
                        }
                    );
                }
            },

            immediate: true,
            deep: true
        },

        defaultSelectID: {
            handler: function (movie) {
                // movie && this.selectMovie()
            },

            immediate: true
        }
    }
};
</script>
<style>
.swiper-container {
    position: relative;
    width: 100%;
    overflow: hidden;
}

.post-bg {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: -1;
    overflow: hidden;
    -webkit-filter: blur(30px);
    filter: blur(30px);
    background-position-y: 40%;
}

.post-bg-mask {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: #40454d;
    opacity: 0.55;
}

.scroll-view_H {
    white-space: nowrap;
    width: 100%;
}

.movie-item {
    display: inline-block;
    width: 156rpx;
    height: 228rpx;
    margin: 50rpx 0 40rpx 40rpx;
}
.movie-item:first-child {
    margin-left: calc(50vw - 78rpx);
}
.movie-item:last-child {
    margin-right: calc(50vw - 78rpx);
}

.movie-item .post {
    width: 156rpx;
    height: 228rpx;
    transition: transform 0.3s;
}

.movie-item .post.select {
    position: relative;
    transform: scale(1.15);
    border: 2px solid #fff;
}

.movie-item .post.select::after {
    content: '';
    position: absolute;
    bottom: -6px;
    left: 50%;
    transform: translateX(-50%);
    width: 20rpx;
    height: 10rpx;
    background-image: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABQAAAAKCAYAAAC0VX7mAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyNpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuNS1jMDE0IDc5LjE1MTQ4MSwgMjAxMy8wMy8xMy0xMjowOToxNSAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENDIChNYWNpbnRvc2gpIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjMwODIyNEEwNTkwRDExRTZBNkMwOTE1NDA0RjA5MDA3IiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjMwODIyNEExNTkwRDExRTZBNkMwOTE1NDA0RjA5MDA3Ij4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6MzA4MjI0OUU1OTBEMTFFNkE2QzA5MTU0MDRGMDkwMDciIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6MzA4MjI0OUY1OTBEMTFFNkE2QzA5MTU0MDRGMDkwMDciLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7kjYk3AAAAVUlEQVR42qzMMQ7AIAwEwSvy8P25EwokZNkYCFcgCu/IzLB7Q9+jS2gz1MG/KN0ZwVOU0fDgLorvI3AVJWozsELJuhmYocyaCvQo1f2jeiT/cK8AAwBzvH2t7aaajQAAAABJRU5ErkJggg==');
    background-repeat: no-repeat;
    background-size: contain;
}

.movie-item image {
    width: 100%;
    height: 100%;
}
</style>
