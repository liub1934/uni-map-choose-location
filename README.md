从已有微信小程序项目中扣离出来的签到打卡及地图定位，不涉及其他业务。  
因为微信小程序的`wx.chooseLocation`无法满足业务需要，所以自己撸了一个，个性化更强一些。

## 使用方法

因为只是在微信小程序中使用，所以一些相关的 SDK 都是腾讯地图的，如果需要在其他端使用，请自行修改。

1、先去腾讯地图申请一个 KEY：[去申请](https://lbs.qq.com)  
2、将申请的 KEY 填入项目`config\index.js`中的`MAP_KEY`  
3、`manifest.json`中配置上自己的小程序的 appid，不然无法定位  
4、执行`npm install`安装一些依赖(必须)

## 项目内容

1、引用了部分[Vant](https://youzan.github.io/vant-weapp)组件，具体见`wxcomponents\vant`  
2、引用了[dayjs](https://github.com/iamkun/dayjs)处理日期格式  
3、引用了部分[lodash](https://lodash.com)的方法  
4、获取位置信息及搜索使用的`微信小程序JavaScript SDK`，详情见文档：[https://lbs.qq.com/qqmap_wx_jssdk/index.html](https://lbs.qq.com/qqmap_wx_jssdk/index.html)

## 截图预览

![image](https://image.liubing.me/2020/03/14/cd7ee4cd27b50.gif)

![image](https://image.liubing.me/2020/03/14/c6598849d265b.png)

![image](https://image.liubing.me/2020/03/14/5f814796dc35a.png)

![image](https://image.liubing.me/2020/03/14/ac6cbfb9247a3.png)

![image](https://image.liubing.me/2020/03/14/d88b49c34579e.png)
