
#### [Dali](https://github.com/patrickfav/Dali)

Dali 是一个Android端提供图像模糊的开源库，内部是使用RenderScript实现，内置了多种图像过滤器，而且很容易扩展。  

* 安装使用

```
compile 'at.favre.lib:dali:0.3.4'
```

还需要在build.gradle中添加下面的内容，使RenderScript工作
```
android {
    ...
    defaultConfig {
        ...
        renderscriptTargetApi 20
        renderscriptSupportModeEnabled true
    }
}
```

* 静态图片模糊(背景图片等)

```
Dali.create(context).load(R.drawable.test_img1).blurRadius(12).into(imageView);
```

Dali还提供了一些图片处理的算法，如(亮度调节、对比度、颜色等)

```
Dali.create(context).load(R.drawable.test_img1).placeholder(R.drawable.test_img1).blurRadius(12)
        .downScale(2).colorFilter(Color.parseColor("#ffccdceb")).concurrent().reScale().into(iv3)
```

* 一切view都可以模糊

使用Dali，所有的view都可以模糊并且在imageview中显示。  

```
Dali.create(context).load(rootView.findViewById(R.id.blurTemplateView)).blurRadius(20)
   		    .downScale(2).concurrent().reScale().skipCache().into(imageView);
```

* 跳过模糊

如果只想使用Dali的其他功能，儿不想使用模糊效果，可以使用下面的代码  

```
Dali.create(context).load(R.drawable.test_img1).algorithm(EBlurAlgorithm.NONE).brightness(70).concurrent().into(iv);
```

* 动态图像模糊

这个特性可以在 `ViewPager` `Scrollview` `RecyclerView`等view中使用  

```
blurWorker = Dali.create(getActivity()).liveBlur(rootViewPagerWrapperView,topBlurView,bottomBlurView).downScale(8).assemble(true);

mViewPager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {
    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
        blurWorker.updateBlurView();
    }
    @Override public void onPageSelected(int position) {}
@Override public void onPageScrollStateChanged(int state) {}
});
```

![](http://7xplrz.com1.z0.glb.clouddn.com/viewpager_anim.gif)

#### 抽屉导航的背景模糊

Dali可以结合DrawerLayout使用，模糊导航控件的背景  

```
protected void onCreate(Bundle savedInstanceState) {
    ...
mDrawerToggle = new DaliBlurDrawerToggle(this, mDrawerLayout,
            toolbar, R.string.drawer_open, R.string.drawer_close) {

  public void onDrawerClosed(View view) {
    super.onDrawerClosed(view);
    invalidateOptionsMenu(); // creates call to onPrepareOptionsMenu()
  }

  public void onDrawerOpened(View drawerView) {
    super.onDrawerOpened(drawerView);
    invalidateOptionsMenu(); // creates call to onPrepareOptionsMenu()
  }
};
mDrawerToggle.setDrawerIndicatorEnabled(true);
// Set the drawer toggle as the DrawerListener
mDrawerLayout.addDrawerListener(mDrawerToggle);
    ...
}

@Override
protected void onPostCreate(Bundle savedInstanceState) {
super.onPostCreate(savedInstanceState);
mDrawerToggle.syncState();
}

@Override
public void onConfigurationChanged(Configuration newConfig) {
super.onConfigurationChanged(newConfig);
mDrawerToggle.onConfigurationChanged(newConfig);
}

@Override
public boolean onPrepareOptionsMenu(Menu menu) {
boolean drawerOpen = mDrawerLayout.isDrawerOpen(mDrawerList);
return super.onPrepareOptionsMenu(menu);
}
```

![](http://7xplrz.com1.z0.glb.clouddn.com/blur_nav.gif)

#### 模糊的过场动画

在使用Dali的过程中，在使图像模糊的过程中可以添加对应的动画  

```
BlurKeyFrameManager man = new BlurKeyFrameManager();
    man.addKeyFrame(new BlurKeyFrame(2,4,0,300));
    man.addKeyFrame(new BlurKeyFrame(2,8,0,300));
```

![](http://7xplrz.com1.z0.glb.clouddn.com/blur_anim.gif)

#### [PanoramaImageView]()

#### [ExpectAnim]()

#### [Chuck]()

#### [BlockCanaryEx]()

#### [BubblePicker]()

#### [StickySwitch]()
