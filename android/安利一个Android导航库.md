### SlidingRootNav
这是一个像`DrawerLayout`一样的抽屉式的导航库，这个库实现的抽屉在content view的下层，滑动之后，才能看到相应的导航页  

![](http://7xplrz.com1.z0.glb.clouddn.com/sample.gif)

### 使用

#### Gradle
添加依赖

```
compile 'com.yarolegovich:sliding-root-nav:1.0.2'
```
#### 使用说明
1. 创建一个 content_view.xml或通过编程方式创建
2. 在Activity中设置view(setContentView)
3. 创建menu.xml或通过编程方式创建
4. 在onCreate方法中注入菜单。

```
new SlidingRootNavBuilder(this)
  .withMenuLayout(R.layout.menu_left_drawer)
  .inject();
```

#### API

##### 过场动画
创建时可以添加一些过场动画，库本身提供一些默认的过场。

```
new SlidingRootNavBuilder(this)
  .withDragDistance(140) //水平动画. Default == 180dp
  .withRootViewScale(0.7f) //设置主view的缩放比例0~0.7. 默认值 == 0.65f;
  .withRootViewElevation(10) //主view垂直方向的值 0~10dp. 默认值 == 8.
  .withRootViewYTranslation(4) //主view y轴方向的过场0~4. 默认值 == 0
  .addRootTransformation(customTransformation) // 添加自定义过场
  .inject();
```
`customTransformation` 是自定义的，需要实现 `RootTransformation` 接口

##### 菜单的行为

```
new SlidingRootNavBuilder(this)
  .withMenuOpened(true) //初始化菜单的状态(打开/关闭) 默认值 == false
  .withMenuLocked(false) //锁定菜单，true时不能打开或关闭菜单 默认值 == false.
  .withGravity(SlideGravity.LEFT) //设置菜单从哪个方向出来，
  .withSavedState(savedInstanceState) //是否保存菜单的状态
  .inject();
```

##### 控制布局

```
public interface SlidingRootNav {
    boolean isMenuHidden();
    boolean isMenuLocked();
    void closeMenu();
    void closeMenu(boolean animated);
    void openMenu();
    void openMenu(boolean animated);
    void setMenuLocked(boolean locked);
    SlidingRootNavLayout getLayout(); //If for some reason you need to work directly with layout - you can
}
```

在调用`inject()`后返回的实例，可以控制布局

##### 回调

* 滑动过程的回调

```
builder.addDragListener(listener);

public interface DragListener {
  void onDrag(float progress); //Float between 0 and 1, where 1 is a fully visible menu
}
```

* 滑动状态的回调

```
builder.addDragStateListener(listener);

public interface DragStateListener {
  void onDragStart();
  void onDragEnd(boolean isMenuOpened);
}
```

##### 兼容 `DrawerLayout.DrawerListener` 回调

```
DrawerListenerAdapter adapter = new DrawerListenerAdapter(yourDrawerListener, viewToPassAsDrawer);
builder.addDragListener(listenerAdapter).addDragStateListener(listenerAdapter);
```

[库-github](https://github.com/yarolegovich/SlidingRootNav)  
[实例源码](https://github.com/yarolegovich/SlidingRootNav/blob/master/sample/src/main/java/com/yarolegovich/slidingrootnav/sample)
