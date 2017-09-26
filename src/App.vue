<template>
  <div id="app">
    <div id="container"
         tabindex="0"></div>
    <div id="panel"></div>
    <!-- <img src="./assets/logo.png"> -->
    <!-- <router-view></router-view> -->
  </div>
</template>

<script>
  export default {
    name: 'app',
    data () {
      return {
        map: {},
        geolocation: {}
      }
    },
    methods: {
      async initMapApi ({ plugin, container }) {
        await this.loaderJSDK()
        this.createMap(container)
        await this.installPlugin(plugin)
        await this.installService()
      },
      loaderJSDK () {
        return new Promise((resolve) => {
          const script = document.createElement('script')
          script.async = true
          script.defer = true
          script.onload = () => {
            resolve()
          }
          script.src = 'http://webapi.amap.com/maps?v=1.4.0&key=e633773ca4137e8182ab13ebd1f9b647'
          document.body.append(script)
        })
      },
      installPlugin (plugin) {
        return new Promise((resolve) => {
          this.map.plugin(plugin, () => {
            resolve()
          })
        })
      },
      installService () {
        return new Promise((resolve) => {
          AMap.service(['AMap.CloudDataSearch'], () => {
            resolve()
          })
        })
      },
      createMap (container) {
        this.map = new AMap.Map(container, {
          resizeEnable: true
        })
      },
      searchPeripheralData (lng, lat) {
        const center = [lng, lat]
        const searchOptions = {
          map: this.map,
          panel: 'panel',
          pageSize: 5,
          orderBy: '_id:ASC'
        }
        // 加载CloudDataSearch服务
        this.search = new AMap.CloudDataSearch('59c22ed12376c11dabb62231', searchOptions)
        this.search.searchNearBy(center, 5000, (...args) => {
          console.log(args)
        })
        AMap.event.addListener(this.search, 'listElementClick', this.onlistElementClick)
      },
      addToolBar () {
        this.map.addControl(new AMap.ToolBar())
        if (location.href.indexOf('&guide=1') !== -1) {
          this.map.setStatus({ scrollWheel: false })
        }
      },
      hotSpotclickHandler () {
        const placeSearch = new AMap.PlaceSearch()
        this.map.on('hotspotclick', (result) => {
          placeSearch.getDetails(result.id, (status, result) => {
            if (status === 'complete' && result.info === 'OK') {
              this.placeSearchCallBack(result)
            }
          })
        })
      },
      placeSearchCallBack (data) { // infoWindow.open(map, result.lnglat);
        console.log(data)
        const infoWindow = new AMap.AdvancedInfoWindow({})
        const poiArr = data.poiList.pois
        const location = poiArr[0].location
        infoWindow.setContent(this.createContent(poiArr[0]))
        infoWindow.open(this.map, location)
      },
      createContent (poi) {  // 信息窗体内容
        const s = []
        s.push('<div class="info-title">' + poi.name + '</div><div class="info-content">' + '地址：' + poi.address)
        s.push('电话：' + poi.tel)
        s.push('类型：' + poi.type)
        s.push('<div>')
        return s.join('<br>')
      },
      launchGeolocation () {
        this.geolocation = new AMap.Geolocation({
          enableHighAccuracy: true, // 是否使用高精度定位，默认:true
          timeout: 10000,          // 超过10秒后停止定位，默认：无穷大
          buttonOffset: new AMap.Pixel(14, 130), // 定位按钮与设置的停靠位置的偏移量，默认：Pixel(10, 20)
          zoomToAccuracy: true,      // 定位成功后调整地图视野范围使定位位置及精度范围视野内可见，默认：false
          buttonPosition: 'RB',
          showCircle: false
        })
        this.map.addControl(this.geolocation)
        this.geolocation.getCurrentPosition()
        // this.geolocation.watchPosition()
      },
      onGeolocationComplete (data) {
        this.searchPeripheralData(data.position.getLng(), data.position.getLat())
      },
      onGeolocationError (data) {
        alert('定位失败')
      },
      initEventHandle () {
        AMap.event.addListener(this.geolocation, 'complete', this.onGeolocationComplete)// 返回定位信息
        AMap.event.addListener(this.geolocation, 'error', this.onGeolocationError)      // 返回定位出错信息
      },
      onlistElementClick (date) {
        console.log(date)
      }
    },
    async mounted () {
      await this.initMapApi({ container: 'container', plugin: ['AMap.Geolocation', 'AMap.PlaceSearch', 'AMap.AdvancedInfoWindow', 'AMap.ToolBar'] })
      this.addToolBar()
      this.launchGeolocation()
      // this.hotSpotclickHandler()
      this.initEventHandle()
    }
  }
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
  }

  #container {
    height: 500px;
    margin: 0px;
  }

  .amap-logo {
    display: none !important;
  }

  .amap-copyright {
    display: none !important;
  }

  .info-title {
    color: white;
    font-size: 14px;
    background-color: blue;
    line-height: 26px;
    padding: 0px 0 0 6px;
    font-weight: lighter;
    letter-spacing: 1px
  }

  .info-content {
    padding: 4px;
    color: #666666;
    line-height: 23px;
    font: 12px Helvetica, 'Hiragino Sans GB', 'Microsoft Yahei', '微软雅黑', Arial;
  }

  .info-content img {
    float: left;
    margin: 3px;
  }
</style>
