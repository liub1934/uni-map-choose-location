<template>
  <div class="signin">
    <view class="location">
      <van-cell class="field-cell"
        required
        title-width="70px"
        title="当前定位：">
        <view v-if="formData.signAddress"
          class="location">
          {{ formData.signAddress }}
        </view>
        <view v-if="location.error"
          class="error">
          {{ location.errorInfo }}
        </view>
        <van-loading v-if="location.loading"
          size="26rpx">
          定位中...
        </van-loading>
        <van-button v-if="location.error"
          slot="right-icon"
          type="primary"
          size="mini"
          @click="getLocation">
          重试
        </van-button>
        <van-button v-if="location.curLocation"
          slot="right-icon"
          type="primary"
          size="mini"
          @click="handleSelect">
          位置微调
        </van-button>
      </van-cell>
    </view>
    <view class="signin-area">
      <van-button class="signin-btn"
        type="primary"
        round
        :disabled="signin.isSignin"
        @click="handleSignin">
        <view class="label">签到</view>
        <view class="time">{{ signin.time }}</view>
      </van-button>
      <view class="signin-result">
        <van-icon v-if="signin.count"
          name="checked"
          size="30rpx"
          color="#4cd964"
          style="margin-right:10rpx" />
        <text v-if="signin.count">今日您已签到<text>{{ signin.count }}</text>次</text>
        <text v-else>今日您还未签到</text>
      </view>
    </view>
  </div>
</template>

<script>
import { formatDate, reverseGeocoder, getLocation } from '@/utils'
import { mapGetters, mapMutations } from 'vuex'
export default {
  data () {
    return {
			signin: {
				time: '', // 签到时间
				count: 0, // 签到时间
				isSignin: false //是否签到
			},
      formData: {
        signAddress: '', // 签到地址
        longitude: '', // 经度
        latitude: '' // 维度
			},
			location: {
				loading: true,
				error: false,
				errorInfo: '定位失败',
				curLocation: null // 当前位置信息
			}
    }
  },
  computed: {
    ...mapGetters(['selectedLocation', 'selectedSearch'])
  },
  onLoad () {
    this.getLocation()
    this.getTime()
    if (this.timerInterval) {
      clearInterval(this.timerInterval)
    } else {
      this.timerInterval = setInterval(this.getTime, 1000)
    }
  },
  methods: {
    ...mapMutations(['SET_SELECTED_SEARCH']),
    // 签到
    handleSignin () {
      if (!this.location.curLocation) {
        this.getLocation()
      } else {
				clearInterval(this.timerInterval)
				this.signin.count++
				this.signin.isSignin = true
				this.getTime(new Date())
				uni.showToast({
					title: '签到成功',
					icon: 'success'
        })
        console.log('签到数据：', this.formData)
			}
    },
    // 获取当前时间时分秒
    getTime (time) {
      time = time ? new Date(time) : new Date()
      this.signin.time = formatDate(time, 'HH:mm:ss')
    },
    // 获取当前定位
    getLocation () {
			this.location.loading = true
			this.location.error = false
      getLocation()
        .then(res => {
					const { longitude, latitude } = res
					this.getLocationInfo({ longitude, latitude })
        })
        .catch(() => {
          this.location.loading = false
          this.location.error = true
        })
    },
    // 获取位置信息
    getLocationInfo (location) {
      reverseGeocoder(location)
        .then(res => {
          console.log('当前位置信息：', res)
          const address = res.result.pois[0].title
					this.formData.signAddress = address
					this.location.curLocation = res.result
          this.location.error = false
          this.location.loading = false
        })
        .catch(err => {
          this.location.loading = false
          this.location.error = true
        })
    },
    // 位置微调
    handleSelect () {
      if (this.location.curLocation) {
        const { lng, lat } = this.location.curLocation.location
        if (this.selectedSearch && (this.selectedSearch.location.lng !== lng || this.selectedSearch.location.lat !== lat)) {
          this.SET_SELECTED_SEARCH(null)
        }
        uni.navigateTo({
					url: `/pages/select/select?longitude=${lng}&latitude=${lat}`
				})
      } else {
        this.getLocation()
      }
    }
  },
  watch: {
    selectedLocation (newData) {
      if (newData) {
        const { title, location } = newData
        this.formData.signAddress = title
        this.formData.longitude = location.lng
        this.formData.latitude = location.lat
        this.location.curLocation = newData
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.signin {
  /deep/ .field-cell {
    .cell-field {
      .van-cell {
        padding: 0;
      }
    }
    .van-cell__value {
      display: flex;
      align-items: center;
      .location {
        text-align: left;
        text-overflow: ellipsis;
        overflow: hidden;
        white-space: nowrap;
        flex: 1;
      }
    }
  }
  .signin-area {
    padding-top: 100rpx;
    text-align: center;
    /deep/ .signin-btn {
      .van-button {
        width: 260rpx;
        height: 260rpx;
        box-shadow: 0px 12rpx 30rpx 0px rgba(25, 137, 250, 0.34);
      }
      .label {
        font-size: 30rpx;
        line-height: 42rpx;
        color: $uni-text-color-inverse;
      }
      .time {
        font-size: 40rpx;
        line-height: 56rpx;
      }
    }
    .signin-result {
      color: $uni-text-color-grey;
      font-size: 24rpx;
      line-height: 34rpx;
      margin-top: 20rpx;
      display: flex;
      align-items: center;
      justify-content: center;
    }
  }
}
</style>
