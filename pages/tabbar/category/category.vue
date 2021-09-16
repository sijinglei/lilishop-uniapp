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

      <div class="r-list">
        <scroll-view scroll-x scroll-with-animation class="c-item-wrap">
          <div
            class="c-item"
            v-for="item in goodsCateList"
            :key="item.id"
            @click="getGoods(item.id)"
            :class="{ active: item.id == params.categoryId }"
          >
            {{ item.name }}
          </div>
        </scroll-view>
        <div class="c-more" @click="isShowMore = !isShowMore">
          <uni-icons color="#BABABA" type="arrowdown"></uni-icons>
        </div>
      </div>
      <div class="top-list" v-show="isShowMore">
        <div
          class="c-item"
          v-for="item in goodsCateList"
          :key="item.id"
          @click="getGoods(item.id)"
          :class="{ active: item.id == params.categoryId }"
        >
          {{ item.name }}
        </div>
      </div>
      <scroll-view
        scroll-with-animation
        scroll-y
        class="right-aside"
        :upper-threshold="-100"
        :lower-threshold="-100"
        enableBackToTop="true"
        lower-threshold="250"
        @scrolltolower="loadmore()"
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
        <div class="goodsClass">
          <u-row
            v-for="(item, index) in goodsList"
            :key="index"
            class="goodsRow"
            style="
              margin: 20rpx 0;
              padding: 10px 0;
              border-bottom: 1rpx solid #f5f3f3;
              border-radius: 0;
            "
          >
            <u-col
              :span="4"
              @click.native="navigateToDetailPage(item)"
              class="switchType1"
            >
              <u-image
                width="150rpx"
                height="150rpx"
                class="imgGoods"
                :src="item.content.thumbnail"
              >
                <u-loading slot="loading"></u-loading>
              </u-image>
            </u-col>
            <u-col
              :span="6"
              @click.native="navigateToDetailPage(item)"
              class="switchType2"
            >
              <div
                class="title clamp3"
                style="
                  font-size: 28rpx;
                  text-align: left;
                  letter-spacing: 0;
                  padding-top: 0;
                "
              >
                {{ item.content.goodsName }}
              </div>
              <view class="price-box">
                <div class="price" v-if="item.content.price != undefined">
                  ¥
                  <span>{{ formatPrice(item.content.price)[0] }}</span>
                  .{{ formatPrice(item.content.price)[1] }}
                </div>
              </view>
              <div class="promotion">
                <div
                  v-for="(promotionItem, promotionIndex) in getPromotion(item)"
                  :key="promotionIndex"
                >
                  <span v-if="promotionItem.indexOf('COUPON') != -1">劵</span>
                  <span v-if="promotionItem.indexOf('FULL_DISCOUNT') != -1">
                    满减
                  </span>
                  <span v-if="promotionItem.indexOf('SECKILL') != -1">
                    秒杀
                  </span>
                </div>
              </div>
              <!-- <div style="overflow: hidden" class="countConfig">
                <span style="float: left; font-size: 22rpx">
                  已售 {{ item.buyCount || '0' }}
                </span>
                <span style="float: right; font-size: 22rpx">
                   {{ item.commentNum || '0' }}条评论
                </span>
              </div> -->
            </u-col>
            <u-col
              :span="2"
              @click.native="navigateToDetailPage(item)"
              class="switchType3"
            >
              <view class="buy" @click.stop="addToCartOrBuy(item.id)">
                <uni-icons color="#ffffff" type="cart-filled"></uni-icons>
                <text class="num" v-if="item.goodNums && item.goodNums > 0">
                  {{ item.goodNums }}
                </text>
              </view>
            </u-col>
          </u-row>
        </div>
        <uni-load-more
          :status="loadingType"
          @loadmore="loadmore()"
        ></uni-load-more>
      </scroll-view>
    </view>
  </view>
</template>

<script>
import { getCategoryList, getGoodsList, getGoodsRelated } from '@/api/goods.js'
import * as API_trade from '@/api/trade.js'
// import Views from '../home/views.vue'
export default {
  data() {
    return {
      currentId: 0,
      ViewsrrentId: 0,
      tabList: [], //左侧标题列表
      categoryList: [], //右侧分类数据列表
      topImg: '', //顶部图片
      isShowMore: false,
      loadingType: 'more', //加载更多状态
      params: {
        pageNumber: 1,
        pageSize: 10,
        // sort: 'grade_asc',
        sort: 'releaseTime',
        order: 'desc',
        keyword: '',
      },
      goodsList: [],
      goodsCateList: [],
      goodIdNum: [],
    }
  },
  onLoad() {
    this.loadData()
    // #ifdef MP-WEIXIN
    // 小程序默认分享
    uni.showShareMenu({ withShareTicket: true })
    // #endif
  },
  onShow() {
    this.getGoodCart()
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
      this.categoryList.forEach(item => {
        this.goodsCateList.push(...item.children)
      })
      console.log('categoryList==', this.categoryList)
      this.getGoods(this.categoryList[0].children[0].id)
    },
    /**
     * 一级分类点击
     */
    tabtap(item, i) {
      this.isShowMore = false
      this.goodsList = []
      this.goodsCateList = []
      if (item.id != this.currentId) {
        this.currentId = item.id
        this.loadListContent(i)
      }
    },
    getGoods(cateId) {
      this.params.pageNumber = 1
      this.params.pageSize = 10
      this.params.categoryId = cateId
      this.isShowMore = false
      uni.pageScrollTo({
        duration: 300,
        scrollTop: 0,
      })
      this.loadGoodData('refresh', 1)
      uni.showLoading({
        title: '正在加载',
      })
    },
    loadmore() {
      this.params.pageNumber++
      this.loadGoodData()
    },
    async loadGoodData(type, loading) {
      this.loadingType = 'loading'
      if (type == 'refresh') {
        this.goodsList = []
      }
      //没有更多直接返回 #TODO
      let goodsList = await getGoodsList(this.params)

      if (goodsList.data.result.content.length < 10) {
        this.loadingType = 'noMore'
      }
      this.goodsList.push(...goodsList.data.result.content)
      this.getGoodCart()
      uni.hideLoading()
    },
    navigateToDetailPage(item) {
      uni.navigateTo({
        url: `/pages/product/goods?id=${item.content.id}&goodsId=${item.content.goodsId}`,
      })
    },
    // 数据去重一下 只显示一次 减免 劵 什么的
    getPromotion(item) {
      if (item.promotionMap) {
        let array = []
        Object.keys(item.promotionMap).forEach(child => {
          if (!array.includes(child.split('-')[0])) {
            array.push(child.split('-')[0])
          }
        })

        return array
      }
    },

    // 格式化金钱  1999 --> [1999,00]
    formatPrice(val) {
      if (typeof val == 'undefined') {
        return val
      }
      return val.toFixed(2).split('.')
    },
    addToCartOrBuy(goodId) {
      let data = {
        skuId: goodId,
        num: 1,
      }
      API_trade.addToCart(data).then(res => {
        if (res.data.code == 200) {
          uni.showToast({
            title: '商品已添加到购物车',
            icon: 'none',
          })
          this.getGoodCart()
        }
      })
    },
    getGoodCart() {
      API_trade.getCarts().then(result => {
        if (result.data.success) {
          let cartDatail = result.data.result
          console.log('this.cartDetail=', cartDatail)
          console.log('this.goodsList=', this.goodsList)
          let skuList = cartDatail.skuList
          this.goodsList.forEach(item => {
            let obj = skuList.find(d => d.goodsSku.id == item.id)
            let num = 0
            if (obj) {
              num = obj.num
            }
            this.$set(item, 'goodNums', num)
          })
        }
      })
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
@import '../../navigation/search/search.scss';
page {
  height: 100%;
  background-color: #fdfaff;
}
.buy {
  width: 50rpx;
  height: 50rpx;
  border-radius: 50%;
  background-color: rgb(88, 240, 133);
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  .num {
    font-size: 22rpx;
    color: #fff;
    background: red;
    position: absolute;
    top: -5px;
    right: -5px;
    width: 16px;
    height: 16px;
    border-radius: 50%;
    text-align: center;
    line-height: 16px;
  }
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
    padding: 60rpx 0 0 0;
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
    position: fixed;
    height: 60rpx;
    z-index: 5;
    top: 82rpx;
    right: 0;
    width: calc(100% - 230rpx);
    background-color: #fff;
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
      justify-content: flex-start;
      align-items: center;
      overflow-x: auto;
      white-space: nowrap;
      -webkit-overflow-scrolling: touch;
      /deep/.uni-scroll-view-content {
        display: flex;
        align-items: center;
      }
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
    height: 40rpx;
    line-height: 40rpx;
    font-size: 20rpx;
    padding: 0 10rpx;
    border-radius: 2rpx;
    color: #000;
    background-color: #f1f5f9;
    margin-right: 10rpx;
    flex-shrink: 0;
    flex-grow: 0;
    flex-basis: auto;
    display: inline-block;
    text-align: center;
    &.active {
      background-color: #e9ece9;
      color: #ff6b35;
    }
  }
  .top-list {
    display: flex;
    flex-wrap: wrap;
    background: #fff;
    z-index: 4;
    position: fixed;
    top: 140rpx;
    right: 0;
    width: calc(100% - 230rpx);
    .c-item {
      margin: 5rpx;
    }
  }
}
</style>
