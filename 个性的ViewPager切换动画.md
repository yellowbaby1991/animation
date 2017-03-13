### 个性的ViewPager切换动画

#### 思路

 1. 想实现多张图片在同一个画面切换需要设置父控件的clipChildren属性设置成FALSE，代表是否限制子控件在容器范围内
 2. 使用PageTransformer来定义不同page切换时候展示的动画，如透明度，缩放等
 3. 拦截父控件的touch事件处理，交给我们自定义的PageView去处理，在ClipViewPager中重写dispatchTouchEvent拦截所有点击事件，判断点击坐标在哪个page就切换过去


#### 代码
##### ClipViewPager.java

``` java
package yellow.com.tubatu;

import android.content.Context;
import android.support.v4.view.ViewPager;
import android.util.AttributeSet;
import android.view.MotionEvent;
import android.view.View;

/**
 * 自定义PageView，处理点击事件
 */
public class ClipViewPager extends ViewPager {

    public ClipViewPager(Context context) {
        super(context);
    }

    public ClipViewPager(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    /**
     * 拦截所有点击事件，判断点击坐标在哪个page就切换过去
     */
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_UP) {
            View view = viewOfClickOnScreen(ev);
            if (view != null) {
                setCurrentItem(indexOfChild(view));
            }
        }
        return super.dispatchTouchEvent(ev);
    }

    /**
     * 判断点击坐标在哪个子page的范围
     */
    private View viewOfClickOnScreen(MotionEvent ev) {
        int childCount = getChildCount();
        int[] location = new int[2];
        for (int i = 0; i < childCount; i++) {
            View v = getChildAt(i);
            v.getLocationOnScreen(location);
            int minX = location[0];
            int minY = getTop();

            int maxX = location[0] + v.getWidth();
            int maxY = getBottom();

            float x = ev.getX();
            float y = ev.getY();

            if ((x > minX && x < maxX) && (y > minY && y < maxY)) {
                return v;
            }
        }
        return null;
    }
}

```

##### MyPagerAdpter.java

``` java
package yellow.com.tubatu;

import android.content.Context;
import android.support.v4.view.PagerAdapter;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;

import java.util.List;

/**
 *  自定义适配器
 */
public class MyPagerAdpter extends PagerAdapter {

    private Context context;
    private List<Integer> list;

    public MyPagerAdpter(Context context, List<Integer> list) {
        this.list = list;
        this.context = context;
    }

    @Override
    public Object instantiateItem(ViewGroup container, int position) {
        ImageView iv = new ImageView(context);
        iv.setImageResource(list.get(position));
        container.addView(iv);
        return iv;
    }

    @Override
    public int getCount() {
        return list.size();
    }

    @Override
    public boolean isViewFromObject(View view, Object object) {
        return view == object;
    }

    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        container.removeView((View) object);
    }
}

```

##### ScalePageTransformer.java

``` java
package yellow.com.tubatu;

import android.support.v4.view.ViewPager;
import android.view.View;

/**
 * 滑动动画
 */
public class ScalePageTransformer implements ViewPager.PageTransformer {

    public static final float MAX_SCALE = 1.2f;
    public static final float MIN_SCALE = 0.6f;

    @Override
    public void transformPage(View page, float position) {

        if (position < -1) {
            position = -1;
        } else if (position > 1) {
            position = 1;
        }

        float tempScale = position < 0 ? 1 + position : 1 - position;

        float slope = (MAX_SCALE - MIN_SCALE) / 1;
        //一个公式
        float scaleValue = MIN_SCALE + tempScale * slope;
        page.setScaleX(scaleValue);
        page.setScaleY(scaleValue);
    }
}


```

##### MainActivity.java

``` java
package yellow.com.tubatu;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.MotionEvent;
import android.view.View;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    private ClipViewPager mViewPager;
    private List<Integer> mList;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mViewPager = (ClipViewPager) findViewById(R.id.viewpager);

        initData();

        mViewPager.setAdapter(new MyPagerAdpter(this,mList));
        mViewPager.setPageTransformer(true,new ScalePageTransformer());//设置切换动画
        mViewPager.setPageMargin(80);//设置页边距

        /**
         * 将父容器的事件交给子控件处理
         */
        findViewById(R.id.page_container).setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                return mViewPager.dispatchTouchEvent(event);
            }
        });
    }

    private void initData() {
        mList = new ArrayList<>();
        mList.add(R.drawable.p001);
        mList.add(R.drawable.p002);
        mList.add(R.drawable.p003);
        mList.add(R.drawable.p004);
        mList.add(R.drawable.p005);
        mViewPager.setOffscreenPageLimit(Math.min(mList.size(), 5));//设置页面数量
    }
}

```

##### activity_main.xml

``` xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/page_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/white"
    android:clipChildren="false"
    tools:context=".MainActivity"
    tools:showIn="@layout/activity_main">

    <yellow.com.tubatu.ClipViewPager
        android:id="@+id/viewpager"
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:layout_centerInParent="true"
        android:overScrollMode="never" />

</RelativeLayout>

```

#### 效果

![enter description here][1]


  [1]: ./images/PageView%E5%88%87%E6%8D%A2%E6%95%88%E6%9E%9C.gif "PageView切换效果"
