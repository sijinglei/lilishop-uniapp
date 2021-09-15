<template>
  <view class="category-wrap">
    <u-navbar class="navbar" :is-back="false">
      <div class="title">商品分类</div>
      <u-search
        class="nav-search"
        disabled
        @click.native="search"
        placeholder="搜索商品"
        :show-action="false"
      ></u-search>
    </u-navbar>
    <view class="content">
      <scroll-view scroll-y scroll-with-animation class="left-aside">
        <view
          v-for="(item, index) in tabList"
          :key="item.id"
          class="f-item b-b"
          :class="{ active: item.id === currentId }"
          @click="tabtap(item, index)"
        >
          {{ item.name }}
        </view>
      </scroll-view>
      <scroll-view
        scroll-with-animation
        scroll-y
        class="right-aside"
        :upper-threshold="-100"
        :lower-threshold="-100"
      >
        <!-- 头部图片 -->
        <!--    <view class="top-img" id="main-top">
          <u-image width="500rpx" height="230rpx" @click="navigateToList(topImg.id,topImg.id)" :src="topImg.image" mode="">
          </u-image>
        </view> -->
        <!--  <view v-for="item in categoryList" :key="item.id" class="s-list" :id="'main-' + item.id">
       
          <text class="s-item">{{ item.name }}</text>
          
          <view class="t-list">
            <view @click="navigateToList(item.id, children.id)" v-if="children.parentId === item.id" class="t-item" v-for="(children, cIndex) in item.children" :key="children.id"
              :class="{ 'margin-right': (cIndex + 1) % 3 == 0 }">
              <u-image width="70px" height="70px" :src="children.image" :lazy-load="true">
              </u-image>
              <text>{{ children.name }}</text>
            </view>
          </view>
        </view> -->
        <view class="r-list">
          <view class="c-item-wrap">
            <view class="c-item" v-for="item in categoryList" :key="item.id">
              {{ item.name }}
            </view>
          </view>
          <view class="c-more" @click="isShowMore = !isShowMore">
            <uni-icons color="#BABABA" type="arrowdown"></uni-icons>
          </view>
        </view>
        <view class="top-list" v-show="isShowMore">
          <view class="c-item" v-for="item in categoryList" :key="item.id">
            {{ item.name }}
          </view>
        </view>
      </scroll-view>
    </view>
  </view>
</template>

<script>
import { getCategoryList } from '@/api/goods.js'
import Views from '../home/views.vue'
export default {
  data() {
    return {
      currentId: 0,
      ViewsrrentId: 0,
      tabList: [], //左侧标题列表
      categoryList: [], //右侧分类数据列表
      topImg: '', //顶部图片
      isShowMore: false,
    }
  },
  onLoad() {
    this.loadData()
    // #ifdef MP-WEIXIN
    // 小程序默认分享
    uni.showShareMenu({ withShareTicket: true })
    // #endif
  },
  methods: {
    /**
     * 查询
     */
    search() {
      uni.navigateTo({
        url: '/pages/navigation/search/searchPage',
      })
    },

    /**
     * 加载图片
     */
    async loadData() {
      let list = await getCategoryList(0)
      this.tabList = list.data.result
      this.currentId = list.data.result[0].id
      this.loadListContent(0)
    },

    /**
     * 加载列表内容
     */
    loadListContent(index) {
      this.topImg = this.tabList[index]
      this.categoryList = this.tabList[index].children
      console.log('categoryList==', this.categoryList)
    },
    /**
     * 一级分类点击
     */
    tabtap(item, i) {
      this.isShowMore = false
      if (item.id != this.currentId) {
        this.currentId = item.id
        this.loadListContent(i)
      }
    },

    navigateToList(sid, tid) {
      uni.navigateTo({
        url: `/pages/navigation/search/searchPage?category=${tid}`,
      })
    },
  },
}
</script>

<style lang="scss" scoped>
page {
  height: 100%;
  background-color: #fdfaff;
}
/* 解决小程序和app滚动条的问题 */
/* #ifdef MP-WEIXIN || APP-PLUS */
::-webkit-scrollbar {
  display: none;
}
/* #endif */
/* 解决H5 的问题 */
/* #ifdef H5 */
uni-scroll-view .uni-scroll-view::-webkit-scrollbar {
  /* 隐藏滚动条，但依旧具备可以滚动的功能 */
  display: none;
  width: 0 !important;
  height: 0 !important;
  -webkit-appearance: none;
  background: transparent;
  color: transparent;
}
/* #endif */

.s-list {
  box-shadow: 0 4rpx 12rpx 0 rgba(0, 0, 0, 0.05);
}
.nav-search {
  padding-left: 30rpx !important;
  padding-right: 20rpx !important;
}
.title {
  display: block;
  width: 200rpx;
  text-align: center;
  font-size: 34rpx;
  letter-spacing: 2rpx;
  // #ifdef MP-WEIXIN
  margin-left: 26rpx;
  // #endif
}
.category-wrap {
  height: 100%;
  .content {
    height: calc(100vh - 94px);
    display: flex;
    color: #333;
    font-size: 28rpx;
    background: #fff;
  }
  .left-aside {
    flex-shrink: 0;
    width: 200rpx;
    height: 100%;
    background-color: #f7f7f7;
  }
  .f-item {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 97rpx;
    position: relative;
    &.active {
      font-weight: bold;
      color: $light-color;
      background: #fff;
    }
  }
  .right-aside {
    flex: 1;
    overflow: hidden;
    padding: 20rpx 0 0 30rpx;
  }

  .top-img {
    width: 500rpx;
    height: 230rpx;
    border-radius: 8px;
    overflow: hidden;
    image {
      width: 100%;
      height: 100%;
    }
  }
  .s-item {
    display: flex;
    align-items: center;
    height: 70rpx;
    padding-top: 16rpx;
    font-weight: 500;
  }
  .t-list {
    display: flex;
    flex-wrap: wrap;
    width: 100%;
    padding-top: 12rpx;
  }
  .margin-right {
    margin-right: 0 !important;
  }
  .t-item {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    width: 150rpx;
    margin-right: 25rpx;
    font-size: 24rpx;
    padding-bottom: 20rpx;
    image {
      width: 140rpx;
      height: 140rpx;
      border-radius: 8px;
      margin-bottom: 20rpx;
    }
    /deep/ .u-image {
      width: 140rpx !important;
      height: 140rpx !important;
      border-radius: 8px !important;
      margin-bottom: 20rpx !important;
    }
  }
  // 新分类
  .r-list {
    position: relative;
    height: 60rpx;
    overflow-x: auto;
    width: 100%;
    &::after {
      content: '';
      position: absolute;
      bottom: 0;
      left: 0;
      width: 100%;
      height: 1px;
      background: #eeeded;
    }
    .c-item-wrap {
      display: flex;
      height: 100%;
      width: 100%;
      justify-content: flex-start;
      align-items: center;
      overflow-x: auto;
      -webkit-overflow-scrolling: touch;
    }
    .c-more {
      position: absolute;
      width: 60rpx;
      height: 58rpx;
      right: 0;
      top: 0;
      background: #fff;
      display: flex;
      justify-content: center;
      align-items: center;
      z-index: 3;
    }
  }
  .c-item {
    flex: none;
    height: 40rpx;
    line-height: 40rpx;
    font-size: 20rpx;
    padding: 0 10rpx;
    border-radius: 2rpx;
    color: #000;
    background-color: #f1f5f9;
    margin-right: 10rpx;
    flex-basis: 44rpx;
    flex-shrink: 0;
    text-align: center;
    white-space: nowrap;
    &.active {
      background-color: #bcf4b5;
      color: #ff6b35;
    }
  }
  .top-list {
    display: flex;
    flex-wrap: wrap;
    background: #fff;
    z-index: 4;
    .c-item {
      margin: 5rpx;
    }
  }
}
</style>
