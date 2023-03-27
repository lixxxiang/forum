# 基于Arcgis SDK的Android端地图应用的设计与开发（一）
移动端测绘软件是一种利用智能手机或平板电脑等移动设备进行地理信息采集、处理和展示的软件，它可以实现外业勘察、地图制作、空间分析等功能，为各行各业提供便捷的地理信息服务。其发展受到了移动互联网技术、云计算技术、大数据技术等多方面的影响和推动，它具有高效、灵活、智能等特点，能够满足不同用户的个性化需求。移动端测绘软件的应用领域非常广泛，包括城市规划、土地管理、环境保护、农林牧渔、交通运输、旅游导航等，它为社会经济发展和公共服务提供了有力的支撑。本文将从设计和开发两方面对基于ArcGIS runtime SDK的Android端测绘应用程序进行描述。
## 1 地图显示
### 样式设计
整体使用Google Material Design第三版的设计方案，页面主体为加载了遥感影像的地图，功能入口按照重要性分别以浮动操作按钮FloatingActionButton和侧边栏NavigationDrawer的形式进行设置。

<p float="left">
<img src="https://user-images.githubusercontent.com/20908880/227881021-aaf524f8-150f-426a-aeb3-6a27cae173a3.jpeg"  width="360" height="740"/> 
&nbsp; &nbsp; &nbsp; &nbsp; 
<img src="https://user-images.githubusercontent.com/20908880/227904981-9762a7b0-c501-4a01-94fa-126ae7f94b56.png"  width="360" height="740"/>
</p>

与Material Design 2设计风格相比，NavigationDrawer组件采用圆角的方式，新式的颜色映射与动态颜色的兼容性，更符合新的视觉风格和交互规范。

<p float="left">
<img src="https://user-images.githubusercontent.com/20908880/227885289-34205262-1b88-46f2-ab43-466dec69215d.png"  width="360"> &nbsp; &nbsp; &nbsp; &nbsp;  <img src="https://user-images.githubusercontent.com/20908880/227885630-5a89e679-d3ed-4634-8651-b0cea4a330a4.png"  width="360">
</p>

FloatingActionButton则使用了角半径更小的圆角风格，其阴影可随具体场景的变化而显示或隐藏，同时也兼容动态颜色，这些特点都是Material Design 2中所不具备的。

<p float="left">
<img src="https://user-images.githubusercontent.com/20908880/227886083-5443b232-3bfd-4260-9504-16f73ddc6bc0.png"  width="360"> &nbsp; &nbsp; &nbsp; &nbsp;  <img src="https://user-images.githubusercontent.com/20908880/227886112-4923244a-791d-45f8-a2c4-6de6e87b63b0.png"  width="360">
</p>

### 功能开发
在功能开发方面，地图引擎的选型为整个应用程序的核心点，对比国内外各大地图引擎后选择ArcGIS Runtime SDK，对比国内的高德地图和百度地图SDK来说，高德地图和百度地图SDK更符合国内使用习惯，但缺点在于对于遥感影像加载，遥感影像处理等功能方面与ArcGIS相差较多；同国外地图Mapbox相比，Mapbox的优点在于比Arcgis更轻量和更易配置，但同样具有特性和功能较少的缺点。而Leaflet.js则基于Web开发，需要使用WebView方式进行加载，在性能上和调用方式上与ArcGIS均有差距。

在图斑信息的展示上有多种方式，包括Point类型，Polygon类型以及使用WMS加载图层方式。

*  Point类型：若点位信息为单一坐标点，则使用Point类型显示点位信息，可以自定义点的样式，包括颜色，大小等：

```
createPoint(){
  val graphicsOverlay = GraphicsOverlay() 
  val point = Point(LATITUDE, LONGITUDE, SpatialReferences.getWgs84())
  val simpleMarkerSymbol = SimpleMarkerSymbol(SimpleMarkerSymbol.Style.CIRCLE, -0xa8cd, 10f)
  val blueOutlineSymbol = SimpleLineSymbol(SimpleLineSymbol.Style.SOLID, -0xff9c01, 2f)
  simpleMarkerSymbol.outline = blueOutlineSymbol

  val pointGraphic = Graphic(point, simpleMarkerSymbol)
  mapView.graphicsOverlay.graphics.add(pointGraphic)
}
```

*  Polygon类型：若点位为地标轮廓，如田地轮廓，建筑物轮廓时，使用Polygon类型加载点位信息：

```
val graphicsOverlay = GraphicsOverlay() 
val polygonPoints = PointCollection(SpatialReferences.getWgs84()).apply {
            // Point(latitude, longitude)
            add(Point(LATITUDE_1, LONGITUDE_1))
            add(Point(LATITUDE_2, LONGITUDE_2))
            add(Point(LATITUDE_3, LONGITUDE_3))
            ...
        }
val polygon = Polygon(polygonPoints)
val polygonFillSymbol = SimpleFillSymbol(SimpleFillSymbol.Style.SOLID, -0x7f00a8cd, blueOutlineSymbol)
val polygonGraphic = Graphic(polygon, polygonFillSymbol)
graphicsOverlay.graphics.add(polygonGraphic)
```
* WMS图层方式：当数据量过大且参考点位多为Polygon类型时，会导致应用程序的假死现象，通过阅读源码可知，graphicsOverlay.graphics.add方式仅支持主线程的操作，此时需使用加载WMS图层方式加载数据信息，可以流畅加载数以万计的点位：

```
val url = "http://localhost:8080/geoserver/BASE_NAME/wms?service=WMS&version=1.1.0&request=GetMap&layers=BASE_NAME:LAYER_NAME&styles=”
mapView.map.operationalLayers.add(WmsLayer(url, mutableListOf(LAYER_NAME)))
```
对应的每个点位的独立信息，可以通过获取点击屏幕位置的经纬度，调用WFS接口服务进而获取到点位信息。参考[https://enterprise.arcgis.com/zh-cn/server/latest/publish-services/windows/wfs-services.htm]()：

```
val wfs = "http://localhost:8080/geoserver/wfs?service=WFS&request=GetFeature&version=1.1.0&typename=LAYER_NAME&outputFormat=json"
```
地图加载Polygon如下图所示：

<img src="https://user-images.githubusercontent.com/20908880/227893900-f3edd0a2-ef38-480b-9022-f5b9536e856d.png"  width="360" height="740">


## 2 点位列表
### 样式设计

点位检索以列表RecyclerView的形式显示，对于列表项来说，在Material Design 3版本中进行了颜色，布局，对齐方式和标准化的高度（56dp、72dp或88dp）的优化，如图所示：

<img src="https://user-images.githubusercontent.com/20908880/227895570-ae3fe788-cc8d-4a9b-92b7-28fd634d2f7f.png"  width="360" height="740">

多种类型的图斑以选项卡TabLayout的方式展示，可以通过滑动和点击切换图斑类型。主体内容为RecyclerView与下拉刷新组件SwipeRefreshLayout，图斑检索的入口以ExtendedFloatingActionButton形式展示，在文字区域显示列表内容数量，点击搜索页在对话框AlertDialog中输入搜索内容进行检索。

### 功能开发

在功能开发方面，布局代码为：

```
<?xml version="1.0" encoding="utf-8”?>
<androidx.constraintlayout.widget.ConstraintLayout ...>
    <androidx.constraintlayout.widget.ConstraintLayout ...>
        <com.google.android.material.appbar.MaterialToolbar .../>
        <com.google.android.material.tabs.TabLayout .../>
        ...
    </androidx.constraintlayout.widget.ConstraintLayout/>
    <androidx.viewpager.widget.ViewPager .../>
</androidx.constraintlayout.widget.ConstraintLayout>
```
其中，ViewPager中包含不同状态的列表项：

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout ...>
    <androidx.swiperefreshlayout.widget.SwipeRefreshLayout ...>
        <androidx.recyclerview.widget.RecyclerView .../>
    </androidx.swiperefreshlayout.widget.SwipeRefreshLayout>
    ...
    <com.google.android.material.floatingactionbutton.ExtendedFloatingActionButton .../>
    ...
</androidx.constraintlayout.widget.ConstraintLayout>
```

由于系统整体使用单一地图模式，即所有的地图相关操作都在同一个MapView上完成，因此在点位列表功能中涉及到一个点位信息传递的功能，通过点击列表项，将对应的点位信息传递至地图页面，该功能的实现方式为Fragment页面调用其父Activity的方法，进行参数的传递。

在列表页中RecyclerView的Adapter中设置点击事件:

```
class CheckPagePagerAdapter(private val activity: MainActivity, private val fragment: CheckPageFragment) :
    PagingDataAdapter<Record_listApp, CheckPagePagerAdapter.PageViewHolder>(Comparator) {
     override fun onBindViewHolder(holder: PageViewHolder, position: Int) {
        holder.view.mConstraintLayout.setOnClickListener {
            activity.navigateHome(record)
        }
    }
}
```
在MainAcitivity中：

```
fun navigateHome(record: MutableMap<String, Any>) {
    HomeFragment.dataFromActivity(record)
}
```
在HomeFragment中即可获得点位信息：

```
fun dataFromActivity(record: MutableMap<String, Any>) {
    Logger.d(record)
}
```

Fragment之间传值有多种方式，也可以通过接口等方式进行实现。


## 3 点位详情
### 样式设计

在点位详情页面使用三段式菜单样式，可以使用户方便的切换关注点为文字信息或地图信息，菜单组件为standard BottomSheet，可以垂直拖动或向下划动来完全关闭，如下图所示：

<p float="left">
<img src="https://user-images.githubusercontent.com/20908880/227899035-173e982d-d427-4ed0-ac6d-a0db01d27db8.png" width="30%"/> 
&nbsp; &nbsp; &nbsp; &nbsp; 
<img src="https://user-images.githubusercontent.com/20908880/227898936-7ff5db8d-07b7-41c0-af02-5ea246befcac.png" width="30%"/>
&nbsp; &nbsp; &nbsp; &nbsp; 
<img src="https://user-images.githubusercontent.com/20908880/227899013-4a3a88dc-09b0-47c1-99c2-cf0c688769b0.png" width="30%"/>
</p>


在菜单顶部右侧设置导航按键作为浮动操作按钮，该按钮的位置随菜单栏的高度而改变，在菜单栏完全打开时处于隐藏状态，而提交核查结果按钮处于显示状态，此外，在完全关闭状态，以小型浮动操作按钮的形式提示上拉打开菜单按钮。

在菜单页中，分别以ChipGroup显示核查内容，使用TextInputLayout填写文字信息，使用RecyclerView+ImageView显示核查图片。

与传统的固定布局的地图+文字形式的页面相比，BottomSheet增强了应用程序的用户体验和功能，允许页面显示更多操作和内容而无需占用太多屏幕空间或需要单独的屏幕和活动。
同时它可以通过提供对相关信息或功能的快速访问来提高用户参与度和保留率。

### 功能开发

点位详情页面的核心功能即为standard bottomSheet的设置与状态管理，初始化standard bottomSheet如代码所示：

```
<androidx.coordinatorlayout.widget.CoordinatorLayout>
	<FrameLayout
		android:id="@+id/standardBottomSheet"
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		style="?attr/bottomSheetStyle"
		app:layout_behavior ="com.google.android.material.bottomsheet.BottomSheetBehavior">
	<!-- Bottom Sheet 内容 -->
	 </FrameLayout >
	 ...
</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

使用BottomSheetBehavior设置滑动状态，如代码所示：

```
val bottomSheetCallback = object : BottomSheetBehavior.BottomSheetCallback() {
    override fun onStateChanged(bottomSheet: View, newState: Int) {
        // Do something for new state.
    }

    override fun onSlide(bottomSheet: View, slideOffset: Float) {
        // Do something for slide offset.
    }
}

// To add the callback:
bottomSheetBehavior.addBottomSheetCallback(bottomSheetCallback)

// To remove the callback:
bottomSheetBehavior.removeBottomSheetCallback(bottomSheetCallback)
```

在onSlide状态中，设置导航按键在菜单展开时渐变隐藏：

```
override fun onStateChanged(bottomSheet: View, newState: Int) {
                when (newState) {
                    BottomSheetBehavior.STATE_EXPANDED -> {
                        mUpFAB.visibility = View.GONE
                        mSubmitEFAB.visibility = View.VISIBLE
                    }
                    BottomSheetBehavior.STATE_HIDDEN -> {
                        mUpFAB.visibility = View.VISIBLE
                        mSubmitEFAB.visibility = View.GONE
                        centerMap()
                    }
                    BottomSheetBehavior.STATE_COLLAPSED -> {
                        resetPlaceHolder()
                        mUpFAB.visibility = View.GONE
                        mSubmitEFAB.visibility = View.GONE
                        topCenterMap()
                    }
                    else -> {
                    }
                }
            }
```

* 当菜单完全打开时（STATE_EXPANDED），设置提交按钮显示，提示上拉按钮隐藏；

* 当菜单完全隐藏时（STATE_HIDDEN），设置提交按钮隐藏，提示上拉按钮显示，地图整体居中；

* 当菜单处于半折叠的中间态时（STATE_COLLAPSED），设置提交按钮隐藏，提示上拉按钮隐藏，地图向上居中。

其中设置导航按钮随动于菜单的方式为设置anchor参数：


```
<com.google.android.material.floatingactionbutton.FloatingActionButton
    android:id="@+id/mNavigationFAB”
    ...
    app:layout_anchor="@+id/mPlaceholder"
    app:layout_anchorGravity="top|end”/>
<TextView
    android:id="@+id/mPlaceholder"
    android:layout_width="5dp"
    android:layout_height="88dp"
    android:visibility="invisible"
    app:layout_anchor="@+id/detail_view2"
    app:layout_anchorGravity="top|end" />
<include
    android:id="@+id/detail_view2"
    layout="@layout/detail_bottom_sheet_2" />
```

设置导航按钮的anchor为一个高度为88的占位textview，高度为88的原因是默认导航按钮高度为56dp，默认导航按钮距离菜单栏高度应为16dp，导航按钮中心至菜单栏的总高度为56/2+16=44dp，为占位Textview顶部与中心点的距离，因此设置占位textview总高度为88dp，如图所示：

<img src="https://user-images.githubusercontent.com/20908880/227907178-abee2a8c-7ab8-4946-952d-af392e4c29fe.png"  width="433" height="740">

通过上述三个部分，基本可以完成基于Arcgis SDK的地图应用程序整体流程的搭建，将在后续文章中对每一部分进行更详细的描述，以及对其他多种功能的介绍。
