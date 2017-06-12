

使用高德地图SDK实现微信、手机QQ发送位置定位功能。

具体功能：

- 定位当前位置，并显示周围地标
- 拖动地图获取地图中间位置，并显示周围地标
- 搜索指定位置，并显示周围地标

功能实现：
配置高德，监听定位
```java
   // 自定义定位蓝点图标
        MyLocationStyle myLocationStyle = new MyLocationStyle();
        myLocationStyle.myLocationIcon(BitmapDescriptorFactory.fromResource(R.drawable.gps_point));
        // 自定义精度范围的圆形边框颜色
        myLocationStyle.strokeColor(Color.TRANSPARENT);
        //自定义精度范围的圆形边框宽度
        myLocationStyle.strokeWidth(5);
        myLocationStyle.anchor(0.5f, 0.5f);
        // 设置圆形的填充颜色
        myLocationStyle.radiusFillColor(Color.TRANSPARENT);
        // 将自定义的 myLocationStyle 对象添加到地图上
        mMap.setMyLocationStyle(myLocationStyle);
        //定位监听
        mMap.setLocationSource(new MyLocationSource());
        // 设置默认定位按钮是否显示
        mMap.getUiSettings().setMyLocationButtonEnabled(false);
        // 隐藏缩放按钮
        mMap.getUiSettings().setZoomControlsEnabled(false);
        mMap.setMyLocationEnabled(true);
        mMap.moveCamera(CameraUpdateFactory.zoomTo(zoom));
```
mMap.setLocationSource()
```java
    private class MyLocationSource implements LocationSource {

        @Override
        public void activate(OnLocationChangedListener onLocationChangedListener) {
            LocationActivity.this.mListener = onLocationChangedListener;
            if (mlocationClient == null) {
                mlocationClient = new AMapLocationClient(this);
                AMapLocationClientOption mLocationOption = new AMapLocationClientOption();
                // 设置定位监听
                mlocationClient.setLocationListener(new MyAMapLocationListener());
                // 设置为高精度定位模式
                mLocationOption.setLocationMode(AMapLocationClientOption.AMapLocationMode.Hight_Accuracy);
                //设置定位请求超时时间,单位是毫秒，默认30000毫秒，建议超时时间不要低于8000毫秒。
                mLocationOption.setHttpTimeOut(5000);
                //获取最近3s内精度最高的一次定位结果：
                // 设置定位参数
                mlocationClient.setLocationOption(mLocationOption);
                //启动定位
                mlocationClient.startLocation();
            }
        }

        @Override
        public void deactivate() {
            mListener = null;
            if (mlocationClient != null) {
                mlocationClient.stopLocation();
                mlocationClient.onDestroy();
            }
            mlocationClient = null;
        }
    }
```
	
定位成功，显示当前位置

```java
    private class MyAMapLocationListener implements AMapLocationListener {
        @Override
        public void onLocationChanged(AMapLocation aMapLocation) {
            if (aMapLocation != null) {
                if (aMapLocation.getErrorCode() == 0) {
                    //定位成功
                    mListener.onLocationChanged(aMapLocation);// 显示系统小蓝点
                    mlocationClient.stopLocation()
                } 
            }
        }
    }
```
显示周围地标
```java
     //高德逆地址转换
      mGeocoderSearch = new GeocodeSearch(this);
      mGeocoderSearch.setOnGeocodeSearchListener(new MyGeocodeSearchListener());
```     
setOnGeocodeSearchListener监听下onRegeocodeSearched里使用高德Poi搜索，得到周围坐标
```java
   @Override
        public void onRegeocodeSearched(RegeocodeResult result, int rCode) {
            if (rCode == 1000) {

                if (result != null && result.getRegeocodeAddress() != null &&
                        result.getRegeocodeAddress().getFormatAddress() != null) {
                    setAddress(result.getRegeocodeAddress());
                    String mType = "地名地址信息|餐饮服务|购物服务|生活服务|医疗保健服务|住宿服务|风景名胜|商务住宅|政府机构及社会团体|科教文化服务|交通设施服务|金融保险服务|公司企业|道路附属设施|公共设施";
                    // 第一个参数表示搜索字符串，第二个参数表示poi搜索类型，第三个参数表示poi搜索区域（空字符串代表全国）
                    PoiSearch.Query query = new PoiSearch.Query("", mType, result.getRegeocodeAddress().getCityCode());
                    query.setPageSize(100);// 设置每页最多返回多少条poiitem
                    query.setPageNum(0);//设置第几页
                    PoiSearch poiSearch = new PoiSearch(LocationActivity.this, query);
                    poiSearch.setOnPoiSearchListener(new MyOnPoiSearchListener());//设置数据返回的监听器
                    poiSearch.setBound(new PoiSearch.SearchBound(mCurrentPoint, 1000, true));//
                    poiSearch.searchPOIAsyn();
                }
            }
        }
````
地图Marker
```java
        MarkerOptions mMarkerOptions = new MarkerOptions();
        mMarkerOptions.draggable(true);//可拖放性
        mMarkerOptions.icon(BitmapDescriptorFactory.fromResource(R.drawable.ic_location_marker));
        mCenterMarker = mMap.addMarker(mMarkerOptions);
        ViewTreeObserver vto = mMapView.getViewTreeObserver();
        vto.addOnGlobalLayoutListener(new MyGlobalLayoutListener());
```
拖动地图监听
```java
          mMap.setOnCameraChangeListener(new MyCameraChangeListener());
```
搜索位置
```java
        // 第一个参数表示搜索字符串，第二个参数表示poi搜索类型，第三个参数表示poi搜索区域（空字符串代表全国）
        PoiSearch.Query mPoiQuery;// Poi查询条件类
        PoiSearch mPoiSearch;
        mPoiQuery = new PoiSearch.Query(key, "", city);
        mPoiSearch = new PoiSearch(this, mPoiQuery);
        mPoiQuery.setPageSize(50);// 设置每页最多返回多少条poiitem
        mPoiQuery.setPageNum(0);//设置查第一页
        mPoiSearch.setOnPoiSearchListener(new MyOnPoiSearchListener());
        mPoiSearch.searchPOIAsyn();//开始搜索
```

