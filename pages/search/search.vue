<template>
  <view class="search">
    <view class="top-search">
      <van-search :value="keyword"
        shape="round"
        placeholder="关键词搜索"
        @change="changeEvent"
        @search="handleSearch">
      </van-search>
    </view>
    <scroll-view class="search-list"
      scroll-y>
      <view class="list-item van-hairline--bottom"
        v-for="(item) in list"
        :key="item.id"
        @tap="handleSelect(item)">
        <view class="info">
          <view class="title ellipsis">{{ item.title }}</view>
          <view class="address ellipsis">{{ item.address }}</view>
        </view>
      </view>
      <view v-if="loading"
        class="loading">
        <van-loading size="32rpx">
          <text class="loading-text">加载中...</text>
        </van-loading>
      </view>
      <view v-if="noResult"
        class="no-result">
        您搜索的地点不存在或者超出了范围
      </view>
    </scroll-view>
  </view>
</template>

<script>
import debounce from 'lodash/debounce'
import { mapSearch } from '@/utils'
import { mapMutations } from 'vuex'
export default {
  data () {
    return {
      keyword: '',
      loading: false,
      noResult: false,
      list: [],
      longitude: '',
      latitude: '',
      changeEvent: debounce(this.handleChange, 500)
    }
  },
  onLoad ({ longitude, latitude }) {
    this.longitude = longitude
    this.latitude = latitude
  },
  methods: {
    ...mapMutations(['SET_SELECTED_LOCATION','SET_SELECTED_SEARCH']),
    handleChange (e) {
      this.keyword = e.detail
      this.mapSearch()
    },
    handleSearch (e) {
      this.keyword = e.detail
      this.mapSearch()
    },
    handleSelect (item) {
      this.SET_SELECTED_SEARCH(item)
      this.SET_SELECTED_LOCATION(null)
      uni.navigateBack()
    },
    mapSearch () {
      if (!this.keyword) return
      this.list = []
      this.noResult = false
      this.loading = true
      mapSearch(this.keyword, {
        longitude: this.longitude,
        latitude: this.latitude,
      })
        .then(res => {
          this.list = res.data
          this.loading = false
          if (!this.list.length) {
            this.noResult = true
          }
        })
        .catch(() => {
          this.noResult = false
          this.loading = false
        })
    }
  }
}

</script>

<style lang="scss" scoped>
.search {
  height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  .top-search {
    box-shadow: 0px 1px 10px 1px rgba(153, 153, 153, 0.34);
  }
  .search-list {
    overflow: auto;
    position: relative;
    flex: 1;
    .list-item {
      padding: 30rpx;
      .title,
      .address {
        text-overflow: ellipsis;
        overflow: hidden;
        white-space: nowrap;
      }
      .title {
        font-size: 28rpx;
        line-height: 40rpx;
      }
      .address {
        font-size: 24rpx;
        height: 34rpx;
        color: $uni-text-color-grey;
      }
    }
  }
  .loading {
    text-align: center;
    margin-top: 30rpx;
  }
  .no-result {
    width: 100%;
    text-align: center;
    font-size: 24rpx;
    color: $uni-text-color-grey;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }
}
</style>
