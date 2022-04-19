<template>
  
  <view>
    <view class="search-box">
      <!-- 使用 uni-ui 提供的搜索组件 -->
      <uni-search-bar @input="input" :radius="100" cancelButton="none"></uni-search-bar>
    </view>
    
    <view class="sugg-list" v-if="searchResults.length !== 0">
      <view class="sugg-item" v-for="(item, i) in searchResults" :key="i" @click="gotoDetail(item.goods_id)">
        <view class="goods-name">{{item.goods_name}}</view>
        <uni-icons type="arrowright" size="16"></uni-icons>
      </view>
    </view>
    
    <view class="history-box" v-else >
      <view class="history-title">
        <text>搜索历史</text>
        <uni-icons type="trash" size="17" @click="clearHistory"></uni-icons>
      </view>
      <view class="history-list">
        <uni-tag :text="item" v-for="(item,index) in history" :key="index" @click="gotoGoodsList(item)"></uni-tag>
      </view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        timer:null,
        kw:'',
        searchResults:[],
        historyList:[]
      };
    },
    computed:{
      history(){
        return [...this.historyList].reverse()
      }
    },
    onLoad() {
      this.historyList = JSON.parse(uni.getStorageSync("kw") || '[]')
    },
    methods:{
      input(e){
        // 历史记录添加处
        if(e === '') this.saveSearchHistory()
        clearTimeout(this.timer)
        this.timer = setTimeout(()=>{
          this.kw = e.trim()
          // console.log(this.kw)
          this.getSearchList()
        },500)
      },
      async getSearchList() {
        if(this.kw === '') {
          this.searchResults = []
          return
        }
        const {data:res} = await uni.$http.get('/api/public/v1/goods/qsearch',{query:this.kw})
        if (res.meta.status !== 200) uni.$showMsg()
        this.searchResults = res.message
        // this.saveSearchHistory()  // 教程建议历史搜索添加处
      },
      saveSearchHistory(){
        if(this.kw !== '') {
          // this.historyList.push(this.kw)
          // 通过Set对象去掉重复搜索历史记录
          const set = new Set(this.historyList)
          set.add(this.kw)
          this.historyList = Array.from(set)
          // 将搜索历史记录持久化存储到本地
          uni.setStorageSync("kw",JSON.stringify(this.historyList))
        }
      },
      gotoDetail(goods_id){
        uni.navigateTo({
          url: '/subpkg/goods_details/goods_details?goods_id=' + goods_id
        })
      },
      clearHistory(){
        this.historyList = []
        uni.setStorageSync("kw", '[]')
      },
      gotoGoodsList(item){
        uni.navigateTo({
          url: '/subpkg/goods_list/goods_list?query='+item
        })
      },
    }
  }
</script>

<style lang="scss">
  .search-box {
    position: sticky;
    top: 0;
    z-index: 999;
  }
  .sugg-list {
    padding: 0 5px;
  
    .sugg-item {
      font-size: 12px;
      padding: 13px 0;
      border-bottom: 1px solid #efefef;
      display: flex;
      align-items: center;
      justify-content: space-between;
  
      .goods-name {
        // 文字不允许换行（单行文本）
        white-space: nowrap;
        // 溢出部分隐藏
        overflow: hidden;
        // 文本溢出后，使用 ... 代替
        text-overflow: ellipsis;
        margin-right: 3px;
      }
    }
  }
  .history-box {
    padding: 0 5px;
  
    .history-title {
      display: flex;
      justify-content: space-between;
      align-items: center;
      height: 40px;
      font-size: 13px;
      border-bottom: 1px solid #efefef;
    }
  
    .history-list {
      display: flex;
      flex-wrap: wrap;
  
      .uni-tag {
        margin-top: 5px;
        margin-right: 5px;
      }
    }
  }
</style>
