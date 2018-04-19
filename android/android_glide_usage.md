### Android开源框架Glide的使用-示例应用
  ![来自 www.gratisography.com](http://7xplrz.com1.z0.glb.clouddn.com/jianshu/android/291H_1.jpg)

  * 加载网络图片
  * 加载本地图片-简易图库

  [工程源码](https://github.com/coderminer/Demo_Public/tree/master/gank)


#### 加载网络图片

  引入对应的库

  ```
  compile 'com.android.support:recyclerview-v7:25.0.0'
  compile 'com.github.bumptech.glide:glide:3.7.0'
  ```

  使用Glide加载网络图片, api接口:  http://gank.io/api/data/福利/10/1
  使用的网络资源是 [干货集中营](http://gank.io/) 提供的福利图片资源，具体的api的接口请访问 [API](http://gank.io/api)
  使用`Glide`开源框架加载图片，`Glide`框架的具体使用简介请参考 [Android开源框架Glide的使用](http://www.jianshu.com/p/299c8332aca6)
  使用`RecyclerView`实现瀑布流模式来显示图片

  创建`RecyclerView`的布局`res/layout/fragment_list.xml`

  ```
  <?xml version="1.0" encoding="utf-8"?>
  <android.support.v7.widget.RecyclerView
      xmlns:android="http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:id="@+id/recycler_view"></android.support.v7.widget.RecyclerView>
  ```

  创建`RecyclerView`的每个item的布局 `res/layout/list_item.xml`
  > 需要注意不要都写成`match_parent`或`wrap_content`,不然就显示不出来瀑布流的效果

  ```
  <?xml version="1.0" encoding="utf-8"?>
  <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:orientation="vertical" android:layout_width="match_parent"
      android:layout_height="wrap_content">
      <ImageView
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:adjustViewBounds="true"
          android:id="@+id/image"/>
  </LinearLayout>
  ```

  创建`RecyclerView`的适配器，`GankAdapter.java`，主要的代码逻辑如下

  ```
  @Override
  public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
      View v = LayoutInflater.from(mContext).inflate(R.layout.list_item,parent,false);
      return new ViewHolder(v);
  }

  @Override
  public void onBindViewHolder(ViewHolder holder, int position) {
      final String url = mItems.get(position);
      Log.e("tag","============onBindViewHolder url: "+url);
      Glide.with(mContext)
              .load(url)
              .placeholder(R.mipmap.ic_launcher)
              .diskCacheStrategy(DiskCacheStrategy.RESULT)
              .into(holder.image);
      holder.image.setOnClickListener(new View.OnClickListener(){
          @Override
          public void onClick(View v) {
              Intent intent = new Intent();
              intent.setClass(mContext,PreviewImageActivity.class);
              intent.putExtra("url",url);
              mContext.startActivity(intent);
          }
      });
  }

  @Override
  public int getItemCount() {
      return mItems.size();
  }

  public class ViewHolder extends RecyclerView.ViewHolder{
       ImageView image;
      public ViewHolder(View itemView) {
          super(itemView);
          image = (ImageView)itemView.findViewById(R.id.image);
      }
  }
  ```

  显示`RecyclerView`，创建一个`Fragment`来显示 `GankFragment.java`
  主要的显示逻辑如下：

  ```
  @Nullable
  @Override
  public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
      View v = inflater.inflate(R.layout.fragment_list,container,false);
      mClient = new OkHttpClient();
      mReyclerView = (RecyclerView) v.findViewById(R.id.recycler_view);
      mReyclerView.setLayoutManager(new StaggeredGridLayoutManager(2,StaggeredGridLayoutManager.VERTICAL));
      mAdapter = new GankAdapter(getActivity(),mUrls);
      mReyclerView.setAdapter(mAdapter);
      loadApi(index);

      mReyclerView.addOnScrollListener(new RecyclerView.OnScrollListener() {
          @Override
          public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
              super.onScrollStateChanged(recyclerView, newState);
          }

          @Override
          public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
              super.onScrolled(recyclerView, dx, dy);
              if(isScrollToEnd(mReyclerView)){
                  Log.e("tag","============scroll to end");
                  index += 1;
                  loadApi(index);
              }
          }
      });

      return v;
  }
  ```

  为了加载网络图片，引入了`OkHttpClient`的第三方库

  ```
  compile 'com.squareup.okhttp3:okhttp:3.4.1'
  ```

  加载网络的图片的逻辑

  ```
  private void loadApi(int page){
    Request request = new Request.Builder().url("http://gank.io/api/data/%E7%A6%8F%E5%88%A9/10/"+page).build();
    mClient.newCall(request).enqueue(new Callback() {
        @Override
        public void onFailure(Call call, IOException e) {
            Log.e("tag","loading failure ");
            e.printStackTrace();
        }

        @Override
        public void onResponse(Call call, Response response) throws IOException {
            if(response.isSuccessful()){
                String result = response.body().string();
                try {
                    JSONObject json = new JSONObject(result);
                    JSONArray array = new JSONArray(json.getString("results"));
                    for(int i = 0;i<array.length();i++){
                        JSONObject ob = array.getJSONObject(i);
                        mUrls.add(ob.getString("url"));
                        Log.e("tag","========== url: "+ob.getString("url"));
                    }

                    mHandler.sendEmptyMessage(2);
                }catch (JSONException e){
                    e.printStackTrace();
                }

            }
        }
    });
  }
  ```

  ![效果图]()

#### 加载本地图片
  使用Glide加载本地图片，和网络图片使用的是同一个适配器的代码`GankAdapter.java`
  显示逻辑代码`LocalAlbumFragment.java`,主要是从本地图像数据库中加载数据


  ```
  private void loadAlbum(){
    AsyncTask<Void, Void, Void> asyncTask = new AsyncTask<Void, Void, Void>() {
        @Override
        protected Void doInBackground(Void... params) {
            Cursor  c = getContext().getContentResolver().query(MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
                    new String[]{MediaStore.Images.ImageColumns.DATA},null,null, MediaStore.Images.ImageColumns.DATE_TAKEN+" desc ");
            if(null != c && c.getCount() > 0 && c.moveToFirst()){
                while (c.moveToNext()){
                    mData.add(c.getString(c.getColumnIndex(MediaStore.Images.ImageColumns.DATA)));
                }
            }
            return null;
        }

        @Override
        protected void onPostExecute(Void aVoid) {
            mHandler.sendEmptyMessage(2);

        }
    };

    asyncTask.execute();

  }
  ```

  ![效果图]()

#### 添加图像变换
  使用`Glide`库时，可以对图像做一些变换处理，如：圆角，模糊等处理，使用`Glide`的`.bitmapTransform()`方法，
  自己需要写对应的变换的方法，但是现在有很好的第三方库已经对一些常用的变换做了封装，可以直接使用，不要重复造轮子
  引入第三方图像变换库 [glide-transformations](https://github.com/wasabeef/glide-transformations)

  ```
  compile 'jp.wasabeef:glide-transformations:2.0.1'
  ```

  这个库提供很多的变换，如 剪裁相关的，颜色变化相关的，模糊相关的等，具体的请参考 [源码](https://github.com/wasabeef/glide-transformations)
  试用了一个圆形的效果

  ```
  Glide.with(mContext)
                .load(url)
                .placeholder(R.mipmap.ic_launcher)
                .diskCacheStrategy(DiskCacheStrategy.RESULT)
                .bitmapTransform(new CropCircleTransformation(mContext)) //使用圆形变换，还可以使用其他的变换
                .into(holder.image);
  ```

  当然，如果对这些效果都不满意，可以自己写对应的变换效果

  ![效果图]()
