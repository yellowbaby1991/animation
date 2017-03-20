### 自定义布局下拉刷新ListView
#### 思路
##### 布局

 1. 刷新头布局较简单，由图片+文本组成，图片根据状态切换箭头和进度条，文本用来显示一次刷新时间
 
``` xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/pull_to_refresh_head"
    android:layout_width="fill_parent"
    android:layout_height="60dip" >

    <LinearLayout
        android:layout_width="200dip"
        android:layout_height="60dip"
        android:layout_centerInParent="true"
        android:orientation="horizontal" >

        <RelativeLayout
            android:layout_width="0dip"
            android:layout_height="60dip"
            android:layout_weight="3"
            >
            <ImageView
                android:id="@+id/arrow"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_centerInParent="true"
                android:src="@drawable/arrow"
                />
            <ProgressBar
                android:id="@+id/progress_bar"
                android:layout_width="30dip"
                android:layout_height="30dip"
                android:layout_centerInParent="true"
                android:visibility="gone"
                />
        </RelativeLayout>

        <LinearLayout
            android:layout_width="0dip"
            android:layout_height="60dip"
            android:layout_weight="12"
            android:orientation="vertical" >

            <TextView
                android:id="@+id/description"
                android:layout_width="fill_parent"
                android:layout_height="0dip"
                android:layout_weight="1"
                android:gravity="center_horizontal|bottom"
                android:text="@string/pull_to_refresh" />

            <TextView
                android:id="@+id/updated_at"
                android:layout_width="fill_parent"
                android:layout_height="0dip"
                android:layout_weight="1"
                android:gravity="center_horizontal|top"
                android:text="@string/updated_at" />
        </LinearLayout>
    </LinearLayout>

</RelativeLayout>
```

 2. 自定义一个LinearLayout，命名RefreshableView，里面只包含了一个ListView，刷新头在构造方法中addView，所以整体布局就很简洁了
 
``` xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="yellow.com.pulltorefresh.MainActivity">

    <yellow.com.pulltorefresh.RefreshableView
        android:id="@+id/refreshable_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <ListView
            android:id="@+id/list_view"
            android:layout_width="fill_parent"
            android:layout_height="fill_parent">
        </ListView>
    </yellow.com.pulltorefresh.RefreshableView>
</RelativeLayout>

```


 

