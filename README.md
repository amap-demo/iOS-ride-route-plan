# iOS-ride-route-plan
iOS骑行出行路线规划demo

## 前述 ##

- 工程是基于iOS 3D地图SDK实现的
- [高德官方网站申请key](http://lbs.amap.com/api/ios-sdk/guide/create-project/get-key/#t1).
- 阅读[骑行出行路线规划](http://lbs.amap.com/api/ios-sdk/guide/route-plan/ride/).
- 查阅[参考手册](http://a.amap.com/lbs/static/unzip/iOS_Map_Doc/AMap_iOS_API_Doc_3D/index.html).

## 使用方法 ##

- 运行demo请先执行pod install --repo-update 安装依赖库，完成后打开.xcworkspace 文件
- 如有疑问请参阅[自动部署](http://lbs.amap.com/api/ios-sdk/guide/create-project/cocoapods/).

## demo运行效果图 ##

![Screenshot](./ScreenShots/1.jpeg)
![Screenshot](./ScreenShots/2.jpeg)

## 核心类／接口 ##

| 类    | 接口  | 说明   |
| -----|:-----:|:-----:|
| AMapSearchDelegate | 	- (void)onRouteSearchDone:(AMapRouteSearchBaseRequest *)request response:(AMapRouteSearchResponse *)response | 路径规划查询完成回调 |
| MAMapViewDelegate | - (MAOverlayRenderer *)mapView:(MAMapView *)mapView rendererForOverlay:(id<MAOverlay>)overlay | 地图上覆盖物的渲染的回调，可以设置路径线路的宽度，颜色等 |
| MAMapViewDelegate | - (MAAnnotationView *)mapView:(MAMapView *)mapView viewForAnnotation:(id<MAAnnotation>)annotation | 地图上的起始点，终点，拐点的标注的回调，可以自定义图标展示等 |

## 核心难点 ##

`Objective-C`

```
//在地图上显示当前选择的路径
- (void)presentCurrentRouteCourse {

    if (self.totalRouteNums <= 0) {
        return;
    }

    [self.naviRoute removeFromMapView];  //清空地图上已有的路线

    self.infoLabel.text = [NSString stringWithFormat:@"共%u条路线，当前显示第%u条",(unsigned)self.totalRouteNums,(unsigned)self.currentRouteIndex + 1];  //提示信息

    MANaviAnnotationType type = MANaviAnnotationTypeRiding; //骑行类型

    AMapGeoPoint *startPoint = [AMapGeoPoint locationWithLatitude:self.startAnnotation.coordinate.latitude longitude:self.startAnnotation.coordinate.longitude]; //起点

    AMapGeoPoint *endPoint = [AMapGeoPoint locationWithLatitude:self.destinationAnnotation.coordinate.latitude longitude:self.destinationAnnotation.coordinate.longitude];  //终点

    //根据已经规划的路径，起点，终点，规划类型，是否显示实时路况，生成显示方案
    self.naviRoute = [MANaviRoute naviRouteForPath:self.route.paths[self.currentRouteIndex] withNaviType:type showTraffic:NO startPoint:startPoint endPoint:endPoint];

    [self.naviRoute addToMapView:self.mapView];  //显示到地图上

    UIEdgeInsets edgePaddingRect = UIEdgeInsetsMake(RoutePlanningPaddingEdge, RoutePlanningPaddingEdge, RoutePlanningPaddingEdge, RoutePlanningPaddingEdge);

    //缩放地图使其适应polylines的展示
    [self.mapView setVisibleMapRect:[CommonUtility mapRectForOverlays:self.naviRoute.routePolylines]
                                                          edgePadding:edgePaddingRect
                                                             animated:YES];

}

```


`Swift`

````
//在地图上显示当前选择的路径
func presentCurrentRouteCourse() {

    if self.totalRouteNums <= 0 {
        return
    }
    self.naviRoute?.removeFromMapView() //清空地图上已有的路线

    self.infoLabel.text = "共\(UInt(self.totalRouteNums))条路线，当前显示第\(UInt(self.currentRouteIndex) + 1)条"  //提示信息

    let type = MANaviAnnotationType.riding //骑行类型

    let startPoint = AMapGeoPoint.location(withLatitude: CGFloat(self.startAnnotation.coordinate.latitude), longitude: CGFloat(self.startAnnotation.coordinate.longitude)) //起点

    let endPoint = AMapGeoPoint.location(withLatitude: CGFloat(self.destinationAnnotation.coordinate.latitude), longitude: CGFloat(self.destinationAnnotation.coordinate.longitude))  //终点

    //根据已经规划的路径，起点，终点，规划类型，是否显示实时路况，生成显示方案
    self.naviRoute = MANaviRoute(for: self.route.paths[self.currentRouteIndex], withNaviType: type, showTraffic: false, start: startPoint, end: endPoint)
    self.naviRoute?.add(to: self.mapView)

    //显示到地图上
    let edgePaddingRect = UIEdgeInsetsMake(RoutePlanningPaddingEdge, RoutePlanningPaddingEdge, RoutePlanningPaddingEdge, RoutePlanningPaddingEdge)
    //缩放地图使其适应polylines的展示
    self.mapView.setVisibleMapRect(CommonUtility.mapRect(forOverlays: self.naviRoute?.routePolylines), edgePadding: edgePaddingRect, animated: true)

}

````

