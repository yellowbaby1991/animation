### HorizontalScrollView仿QQ侧滑
#### 思路

 1. HorizontalScrollView会将无法全屏显示的子控件滑动显示，所以可以用来做我们的侧滑效果
 2. 由于HorizontalScrollView默认会把所有子控件全屏展示，所以需要在onMeasure中计算出子控件的长度，这里指的是菜单栏和内容栏
 3. 重写onTouchEvent处理滑动后的效果，根据滑动距离判断显示左边还是右边
 4. 重写onScrollChanged显示滑动效果，导入nineoldandroid.jar兼容动画效果


#### 代码
##### activity_main.xml

``` xml
<yellow.com.horizontalscrollview.SlidingMenu
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="wrap_content"
    android:layout_height="fill_parent"
    android:scrollbars="none">

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="fill_parent"
        android:orientation="horizontal">

        <include layout="@layout/layout_menu" />

        <LinearLayout
            android:layout_width="fill_parent"
            android:layout_height="fill_parent"
            android:background="@drawable/qq"></LinearLayout>
    </LinearLayout>

</yellow.com.horizontalscrollview.SlidingMenu>
```

##### layout_menu.xml

``` xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/img_frame_background" >

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_centerVertical="true"
        android:orientation="vertical" >

        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content" >

            <ImageView
                android:id="@+id/one"
                android:layout_width="50dp"
                android:layout_height="50dp"
                android:layout_centerVertical="true"
                android:layout_marginLeft="20dp"
                android:layout_marginTop="20dp"
                android:src="@drawable/img_1" />

            <TextView
                android:layout_width="fill_parent"
                android:layout_height="wrap_content"
                android:layout_centerVertical="true"
                android:layout_marginLeft="20dp"
                android:layout_toRightOf="@id/one"
                android:text="第1个Item"
                android:textColor="#f0f0f0"
                android:textSize="20sp" />
        </RelativeLayout>

        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content" >

            <ImageView
                android:id="@+id/two"
                android:layout_width="50dp"
                android:layout_height="50dp"
                android:layout_centerVertical="true"
                android:layout_marginLeft="20dp"
                android:layout_marginTop="20dp"
                android:src="@drawable/img_2" />

            <TextView
                android:layout_width="fill_parent"
                android:layout_height="wrap_content"
                android:layout_centerVertical="true"
                android:layout_marginLeft="20dp"
                android:layout_toRightOf="@id/two"
                android:text="第2个Item"
                android:textColor="#f0f0f0"
                android:textSize="20sp" />
        </RelativeLayout>

        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content" >

            <ImageView
                android:id="@+id/three"
                android:layout_width="50dp"
                android:layout_height="50dp"
                android:layout_centerVertical="true"
                android:layout_marginLeft="20dp"
                android:layout_marginTop="20dp"
                android:src="@drawable/img_3" />

            <TextView
                android:layout_width="fill_parent"
                android:layout_height="wrap_content"
                android:layout_centerVertical="true"
                android:layout_marginLeft="20dp"
                android:layout_toRightOf="@id/three"
                android:text="第3个Item"
                android:textColor="#f0f0f0"
                android:textSize="20sp" />
        </RelativeLayout>

        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content" >

            <ImageView
                android:id="@+id/four"
                android:layout_width="50dp"
                android:layout_height="50dp"
                android:layout_centerVertical="true"
                android:layout_marginLeft="20dp"
                android:layout_marginTop="20dp"
                android:src="@drawable/img_4" />

            <TextView
                android:layout_width="fill_parent"
                android:layout_height="wrap_content"
                android:layout_centerVertical="true"
                android:layout_marginLeft="20dp"
                android:layout_toRightOf="@id/four"
                android:text="第一个Item"
                android:textColor="#f0f0f0"
                android:textSize="20sp" />
        </RelativeLayout>

        <RelativeLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content" >

            <ImageView
                android:id="@+id/five"
                android:layout_width="50dp"
                android:layout_height="50dp"
                android:layout_centerVertical="true"
                android:layout_marginLeft="20dp"
                android:layout_marginTop="20dp"
                android:src="@drawable/img_5" />

            <TextView
                android:layout_width="fill_parent"
                android:layout_height="wrap_content"
                android:layout_centerVertical="true"
                android:layout_marginLeft="20dp"
                android:layout_toRightOf="@id/five"
                android:text="第5个Item"
                android:textColor="#f0f0f0"
                android:textSize="20sp" />
        </RelativeLayout>
    </LinearLayout>

</RelativeLayout>
```

##### SlidingMenu.java

``` java
package yellow.com.horizontalscrollview;

import android.content.Context;
import android.util.AttributeSet;
import android.util.TypedValue;
import android.view.MotionEvent;
import android.view.ViewGroup;
import android.widget.HorizontalScrollView;
import android.widget.LinearLayout;

import com.nineoldandroids.view.ViewHelper;

public class SlidingMenu extends HorizontalScrollView {

    private int mMenuRightPadding = 50;//菜单和内容的间距

    private boolean once;//只在第一次加载的时候measure

    /**
     * 菜单的宽度
     */
    private int mMenuWidth;
    private int mHalfMenuWidth;

    private ViewGroup mMenu;
    private ViewGroup mContent;

    /**
     * 屏幕宽度
     */
    private int mScreenWidth;

    public SlidingMenu(Context context, AttributeSet attrs) {
        super(context, attrs);
        mScreenWidth = ScreenUtils.getScreenWidth(context);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        if (!once) {
            LinearLayout wrapper = (LinearLayout) getChildAt(0);
            mMenu = (ViewGroup) wrapper.getChildAt(0);
            mContent = (ViewGroup) wrapper.getChildAt(1);
            mMenuRightPadding = (int) TypedValue.applyDimension(
                    TypedValue.COMPLEX_UNIT_DIP, mMenuRightPadding, mContent
                            .getResources().getDisplayMetrics());
            mMenuWidth = mScreenWidth - mMenuRightPadding;
            mHalfMenuWidth = mMenuWidth / 2;
            mMenu.getLayoutParams().width = mMenuWidth;
            mContent.getLayoutParams().width = mScreenWidth;
        }
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        super.onLayout(changed, l, t, r, b);
        if (changed) {
            // 将菜单隐藏
            this.scrollTo(mMenuWidth, 0);
            once = true;
        }
    }

    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        int action = ev.getAction();
        switch (action) {
            // Up时，进行判断，如果显示区域大于菜单宽度一半则完全显示，否则隐藏
            case MotionEvent.ACTION_UP:
                int scrollX = getScrollX();
                if (scrollX > mHalfMenuWidth)
                    this.smoothScrollTo(mMenuWidth, 0);
                else
                    this.smoothScrollTo(0, 0);
                return true;
        }
        return super.onTouchEvent(ev);
    }

    //滑动显示动画
    @Override
    protected void onScrollChanged(int l, int t, int oldl, int oldt) {
        super.onScrollChanged(l, t, oldl, oldt);
        float scale = l * 1.0f / mMenuWidth;
        ViewHelper.setTranslationX(mMenu, mMenuWidth * scale );
    }
}

```

#####  ScreenUtils.java

``` java
package yellow.com.horizontalscrollview;

import android.content.Context;
import android.util.DisplayMetrics;
import android.view.WindowManager;
public class ScreenUtils
{
	private ScreenUtils()
	{
		throw new UnsupportedOperationException("cannot be instantiated");
	}

	/**
	 * 获得屏幕高度
	 *
	 * @param context
	 * @return
	 */
	public static int getScreenWidth(Context context)
	{
		WindowManager wm = (WindowManager) context
				.getSystemService(Context.WINDOW_SERVICE);
		DisplayMetrics outMetrics = new DisplayMetrics();
		wm.getDefaultDisplay().getMetrics(outMetrics);
		return outMetrics.widthPixels;
	}

}

```

##### MainActivity.java

``` java
package yellow.com.horizontalscrollview;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}

```

#### 效果
![enter description here][1]


  [1]: ./images/qq%E4%BE%A7%E6%BB%91%E6%95%88%E6%9E%9C.gif "qq侧滑效果"