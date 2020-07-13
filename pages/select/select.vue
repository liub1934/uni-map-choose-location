<template>
  <view class="select-location">
		<!-- 顶部搜索框 -->
		<view class="top-search">
		  <van-search :value="keyword"
		    background="rgba(255, 255, 255, 0)"
		    shape="round"
		    readonly
		    placeholder="关键词搜索"
		    @click.native="goSearch">
		  </van-search>
		</view>
    <map id="map"
      :latitude="latitude"
      :longitude="longitude"
      @regionchange="changeEvent">
      <!-- 定位标志 -->
      <cover-image class="location-icon"
        src="/static/images/location.png">
      </cover-image>
      <!-- tip -->
      <cover-view class="tip">您可以拖动地图，标记当前精准位置</cover-view>
    </map>
    <view class="search-area">
      <view class="search-list">
        <scroll-view class="list"
          scroll-y>
          <view :class="['list-item', 'flex', 'van-hairline--bottom', active === item.id ? 'active' : '']"
            v-for="(item) in list"
            :key="item.id"
            @tap="selectedAddress(item)">
            <van-icon v-if="active === item.id"
              class="icon"
              name="checked"
              size="34rpx"
              :color="active === item.id ? '#007aff' : ''">
            </van-icon>
            <view class="info">
              <view class="title ellipsis">{{ item.title }}</view>
              <view class="address ellipsis">{{ item.address }}</view>
            </view>
          </view>
        </scroll-view>
        <van-button type="primary"
          block
          @click="handleConfirm">
          确认位置
        </van-button>
      </view>
    </view>
  </view>
</template>

<script>
import debounce from 'lodash/debounce'
import { reverseGeocoder } from '@/utils'
import { mapGetters, mapMutations } from 'vuex'
export default {
  data () {
    return {
      keyword: '',
      active: '',
      longitude: '',
      latitude: '',
      list: [],
      mapContext: null,
      changeEvent: debounce(this.handleRegionChange, 300)
    }
  },
  computed: {
    ...mapGetters(['selectedLocation', 'selectedSearch'])
  },
  onLoad ({ longitude, latitude }) {
    this.mapContext = uni.createMapContext('map')
    this.longitude = longitude
    this.latitude = latitude
    this.getLocationInfo({ longitude, latitude })
  },
  methods: {
    ...mapMutations(['SET_SELECTED_LOCATION', 'SET_SELECTED_SEARCH']),
    goSearch () {
      uni.navigateTo({
        url: `/pages/search/search?longitude=${this.longitude}&latitude=${this.latitude}`
      })
    }, 
    getLocationInfo (location) {
      reverseGeocoder(location)
        .then(res => {
          let pois = res.result.pois
          if (this.selectedLocation) {
            const index = pois.findIndex(item => item.id === this.selectedLocation.id)
            if (index > -1) pois.splice(index, 1)
            pois.unshift(this.selectedLocation)
          } else if (this.selectedSearch) {
            const index = pois.findIndex(item => item.id === this.selectedSearch.id)
            if (index > -1) pois.splice(index, 1)
            pois.unshift(this.selectedSearch)
          }
          this.list = pois
          if (pois.length) {
            this.active = pois[0].id
            
          }
        })
    },
    handleRegionChange (e) {
      if (e.type === 'end' && e.causedBy === 'drag') {
        this.SET_SELECTED_LOCATION(null)
        this.SET_SELECTED_SEARCH(null)
        this.mapContext.getCenterLocation({
          success: res => {
            const { longitude, latitude } = res
            this.getLocationInfo({ longitude, latitude })
          }
        })
      }
    },
    selectedAddress (item) {
      this.active = item.id
      this.mapContext.moveToLocation({
        latitude: item.location.lat,
        longitude: item.location.lng
      })
    },
    handleConfirm () {
      const index = this.list.findIndex(item => item.id === this.active)
      if (index > -1) {
        this.SET_SELECTED_LOCATION(this.list[index])
        uni.navigateBack()
      }
    }
  },
  watch: {
    selectedSearch (newData) {
      if (newData) {
        const { lng, lat } = newData.location
        this.mapContext.moveToLocation({
          longitude: lng,
          latitude: lat
        })
        this.getLocationInfo({
          longitude: lng,
          latitude: lat
        })
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.select-location {
  height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  #map {
    width: 100%;
    height: 600rpx;
    position: relative;
  }
  /deep/ .top-search {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
		z-index: 1;
    .van-search__content {
      background: $uni-bg-color;
    }
  }
  .location-icon {
    width: 64rpx;
    height: 64rpx;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }
  .tip {
    font-size: 20rpx;
    color: $uni-text-color-grey;
    line-height: 42rpx;
    padding: 0 20rpx;
    position: absolute;
    left: 50%;
    bottom: 30rpx;
    box-shadow: 0px 1px 10px 1px rgba(153, 153, 153, 0.34);
    background-color: $uni-bg-color;
    border-radius: 4px;
    transform: translateX(-50%);
  }
  .search-area {
    width: 100%;
    padding: 0 30rpx;
    box-sizing: border-box;
    position: absolute;
    top: 600rpx;
    bottom: 0;
    .search-list {
      height: 100%;
      background-color: $uni-bg-color;
      border-radius: 4px;
      padding: 20rpx 0;
      box-sizing: border-box;
      display: flex;
      flex-direction: column;
      .list {
        flex: 1;
        overflow: auto;
        .list-item {
          padding: 15rpx 0;
          position: relative;
          &.active {
            .title,
            .address {
              color: $uni-color-primary;
            }
          }
          .info {
            padding-left: 50rpx;
          }
          .icon {
            position: absolute;
            left: 0;
            top: 50%;
            transform: translateY(-50%);
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
    }
  }
}
</style>
