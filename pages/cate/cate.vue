<template>
  <view class="cart-container">
    <my-search @click="gotoSearch"></my-search>
    <view class="scroll-view-container">
      <!-- 左侧的滚动视图区域 -->
      <scroll-view class="left-scroll-view" scroll-y :style="{height: wh + 'px'}">
        <block v-for="(item,index) in cateList" :key="index">
          <view :class="['left-scroll-view-item',index === active ? 'active' : '']" @click="activeChange(index)">
            {{item.cat_name}}
          </view>
        </block>
      </scroll-view>
      <!-- 右侧的滚动视图区域 -->
      <scroll-view class="right-scroll-view" scroll-y :style="{height: wh + 'px'}" :scroll-top="scrollTop">
        <view class="cate-lv2" v-for="(item, index) in cateLevel2" :key="index">
          <view class="cate-lv2-title">{{item.cat_name}}</view>
          <view class="cate-lv3-list">
            <view class="cate-lv3-item" v-for="(item2, index2) in item.children" :key="index2" @click="gotoGoodsList(item2.cat_id)">
              <image :src="item2.cat_icon"></image>
              <text>{{item2.cat_name}}</text>
            </view>
          </view>
        </view>
      </scroll-view>
    </view>
  </view>
</template>

<script>
  import badgeMix from '@/mixins/tabbar-badge.js'
  export default {
    data() {
      return {
        wh: 0,
        active: 0,
        scrollTop: 0,
        cateList: [],
        cateLevel2: []
      };
    },
    mixins:[badgeMix],
    onLoad() {
      const sysInfo = uni.getSystemInfoSync()
      this.wh = sysInfo.windowHeight - 50
      this.getCateList()
    },
    methods: {
      async getCateList() {
        const {
          data: res
        } = await uni.$http.get('/api/public/v1/categories')
        if (res.meta.status !== 200) uni.$showMsg()
        this.cateList = res.message
        this.cateLevel2 = res.message[0].children
      },
      activeChange(index) {
        this.active = index
        this.cateLevel2 = this.cateList[index].children
        this.scrollTop = this.scrollTop ? 1 : 0
      },
      gotoGoodsList(cat_id) {
        // console.log(cat_id)
        uni.navigateTo({
          url: '/subpkg/goods_list/goods_list?cid=' + cat_id
        })
      },
      gotoSearch() {
        uni.navigateTo({
          url: '/subpkg/search/search'
        })
      },
    }
  }
</script>

<style lang="scss">
  .scroll-view-container {
    display: flex;

    .left-scroll-view {
      width: 120px;

      .left-scroll-view-item {
        line-height: 60px;
        background-color: #f7f7f7;
        text-align: center;
        font-size: 12px;

        // 激活项的样式
        &.active {
          background-color: #ffffff;
          position: relative;

          // 渲染激活项左侧的红色指示边线
          &::before {
            content: ' ';
            display: block;
            width: 3px;
            height: 30px;
            background-color: #c00000;
            position: absolute;
            left: 0;
            top: 50%;
            transform: translateY(-50%);
          }
        }
      }
    }
  }

  .cate-lv2-title {
    font-size: 12px;
    font-weight: bold;
    text-align: center;
    padding: 15px 0;
  }

  .cate-lv3-list {
    display: flex;
    flex-wrap: wrap;

    .cate-lv3-item {
      width: 33.33%;
      margin-bottom: 10px;
      display: flex;
      flex-direction: column;
      align-items: center;

      image {
        width: 60px;
        height: 60px;
      }

      text {
        font-size: 12px;
      }
    }
  }

  .right-scroll-view {
    background-color: #ffffff
  }
  .cart-container{
    padding-bottom: 50px;
  }
</style>
