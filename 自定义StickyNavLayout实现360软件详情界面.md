### 自定义StickyNavLayout实现360软件详情界面
#### 思路
1. 整个StickyNavLayout由三部分组成，顶部软件详细栏mTop，中部导航栏mNav，底部详情列表栏mViewPage，StickyNavLayout继承LinearLayout垂直布局
2. 如何实现滑动？
   - 重写onTouchEvent，计算出垂直滑动的距离，然后使用scrollBy来进行滑动
3. 如何防止滑动出界？
   - 在scrollTo中限定y的范围，如果出界就强制定为边界
4. 如何处理滑动冲突？重写onInterceptTouchEvent，在需要拦截的时候返回TRUE
   - 头部被隐藏，并且在下拉的时候
   - 头部不被隐藏的时候，并且在最底部，并且上拉的时候
   - 头部不被隐藏的时候，并且不在最底部
5. 如何处理水平移动和竖直移动灵敏度问题？
   - 使用VelocityTracker计算水平和竖直移动速度，当竖直速度大的时候才滑动

#### 代码
##### activity_main.xml 布局文件
``` xml
<yellow.com.stickynavlayout.view.StickyNavLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <RelativeLayout
        android:id="@id/id_stickynavlayout_topview"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#4400ff00">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="240dp"
            android:gravity="center"
            android:text="软件介绍"
            android:textSize="30sp"
            android:textStyle="bold" />
    </RelativeLayout>

    <yellow.com.stickynavlayout.view.SimpleViewPagerIndicator
        android:id="@id/id_stickynavlayout_indicator"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:background="#ffffffff"></yellow.com.stickynavlayout.view.SimpleViewPagerIndicator>

    <android.support.v4.view.ViewPager
        android:id="@id/id_stickynavlayout_viewpager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="#44ff0000">
    </android.support.v4.view.ViewPager>

</yellow.com.stickynavlayout.view.StickyNavLayout>
```

##### fragment_tab.xml 底部的mViewPager

``` xml
<?xml version="1.0" encoding="utf-8"?>
<GridView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@id/id_stickynavlayout_innerscrollview"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_gravity="center"
    android:horizontalSpacing="10dp"
    android:scrollbars="none"
    android:verticalSpacing="4dp">

</GridView>
```

##### item_mine.xml  导航条
``` xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/id_info"
    android:layout_width="match_parent"
    android:layout_height="50dp"
    android:background="#ffffffff"
    android:gravity="center" >
    

</TextView>
```

##### StickyNavLayout.java 核心代码
``` java
package yellow.com.stickynavlayout.view;

import android.content.Context;
import android.support.v4.view.ViewPager;
import android.util.AttributeSet;
import android.util.Log;
import android.view.MotionEvent;
import android.view.VelocityTracker;
import android.view.View;
import android.view.ViewGroup;
import android.widget.LinearLayout;
import android.widget.OverScroller;

import yellow.com.stickynavlayout.R;

public class StickyNavLayout extends LinearLayout {

    private View mTop;
    private View mNav;
    private ViewPager mViewPager;

    private int mTopViewHeight;

    private float mLastY;//上一次触摸点的纵坐标

    private boolean isTopHidden = false;

    private static final String TAG = "StickyNavLayout";

    private VelocityTracker mVelocityTracker;

    private OverScroller mScroller;

    public StickyNavLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
        mScroller = new OverScroller(context);
    }

    public StickyNavLayout(Context context) {
        super(context);
    }


    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        ViewGroup.LayoutParams params = mViewPager.getLayoutParams();
        params.height = getMeasuredHeight() - mNav.getMeasuredHeight();
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mTopViewHeight = mTop.getMeasuredHeight();
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent event) {
        int action = event.getAction();
        float y = event.getY();
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                mLastY = y;
                break;
            case MotionEvent.ACTION_MOVE:
                float dy = y - mLastY;
                initVelocityTrackerIfNotExists();
                mVelocityTracker.addMovement(event);
                mVelocityTracker.computeCurrentVelocity(1000);
                float xVelocity = Math.abs(mVelocityTracker.getXVelocity());
                float yVelocity = Math.abs(mVelocityTracker.getYVelocity());

                /**
                 * 以下情况需要拦截
                 * 1）头部被隐藏，并且在下拉的时候
                 * 2) 头部不被隐藏的时候,并且在最底部，并且上拉
                 * 3）头部不被隐藏的时候，并且不在最底部
                 */
                if ((isTopHidden && dy > 0 && yVelocity > xVelocity) ||
                        (!isTopHidden && getScrollY() == 0 && dy < 0 && yVelocity > xVelocity) ||
                        (!isTopHidden && getScrollY() != 0 && yVelocity > xVelocity)) {
                    mLastY = y;
                    Log.d(TAG, isTopHidden + "  " + getScrollY() + "  " + dy);
                    return true;
                }
            case MotionEvent.ACTION_UP:
                recycleVelocityTracker();
                break;
        }
        return super.onInterceptTouchEvent(event);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int action = event.getAction();
        float y = event.getY();
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                if (!mScroller.isFinished()) {//如果正在滑动就终止动画
                    mScroller.abortAnimation();
                }
                mLastY = y;
                return true;
            case MotionEvent.ACTION_MOVE://滑动
                float dy = y - mLastY;
                scrollBy(0, (int) -dy);
                mLastY = y;
                break;
            case MotionEvent.ACTION_UP:
        }
        return super.onTouchEvent(event);
    }

    @Override
    public void scrollTo(int x, int y) {
        if (y < 0) {//下边界
            y = 0;
        }
        if (y > mTopViewHeight) {//上边界
            y = mTopViewHeight;
            isTopHidden = true;
        }
        if (y != getScrollY()) {
            super.scrollTo(x, y);
        }

        if (mTopViewHeight != getScrollY()) {
            isTopHidden = false;
        }
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        mTop = findViewById(R.id.id_stickynavlayout_topview);
        mNav = findViewById(R.id.id_stickynavlayout_indicator);
        View view = findViewById(R.id.id_stickynavlayout_viewpager);
        if (!(view instanceof ViewPager)) {
            throw new RuntimeException(
                    "id_stickynavlayout_viewpager show used by ViewPager !");
        }
        mViewPager = (ViewPager) view;
    }

    private void initVelocityTrackerIfNotExists() {
        if (mVelocityTracker == null) {
            mVelocityTracker = VelocityTracker.obtain();
        }
    }

    private void recycleVelocityTracker() {
        if (mVelocityTracker != null) {
            mVelocityTracker.recycle();
            mVelocityTracker = null;
        }
    }

}

```

##### SimpleViewPagerIndicator.java 导航条

``` java
package yellow.com.stickynavlayout.view;

import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.util.AttributeSet;
import android.util.TypedValue;
import android.view.Gravity;
import android.view.WindowManager;
import android.widget.LinearLayout;
import android.widget.TextView;

public class SimpleViewPagerIndicator extends LinearLayout {

    private Paint mPaint = new Paint();

    private static final int COLOR_TEXT_NORMAL = 0xFF000000;

    private String[] mTitles;

    private int mTabWidth;
    private int mTabCount;
    private int mTabCutWidth = 50;

    private float mTranslationX;

    public SimpleViewPagerIndicator(Context context) {
        this(context, null);
    }

    public SimpleViewPagerIndicator(Context context, AttributeSet attrs) {
        super(context, attrs);
        mPaint.setColor(Color.BLACK);
        mPaint.setStrokeWidth(2.0F);//底部指示线的宽度
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mTabWidth = (w / mTabCount) - mTabCutWidth;//指示线的长度
    }

    @Override
    protected void dispatchDraw(Canvas canvas) {
        super.dispatchDraw(canvas);
        canvas.save();
        canvas.translate(mTranslationX, getHeight() - 2);
        canvas.drawLine(mTabCutWidth, 0, mTabWidth, 0, mPaint);//（startX, startY, stopX, stopY, paint）
        canvas.restore();
    }

    public void scroll(int position, float offset) {
        mTranslationX = getWidth() / mTabCount * (position + offset);
        invalidate();
    }

    public void setTitle(String[] titles) {
        mTitles = titles;
        mTabCount = titles.length;
        generateTitleView();
    }

    private void generateTitleView() {
        int count = mTitles.length;
        for (int i = 0; i < count; i++) {
            TextView tv = new TextView(getContext());
            LayoutParams lp = new LayoutParams(0, WindowManager.LayoutParams.MATCH_PARENT);
            lp.weight = 1;
            tv.setGravity(Gravity.CENTER);
            tv.setTextColor(COLOR_TEXT_NORMAL);
            tv.setText(mTitles[i]);
            tv.setTextSize(TypedValue.COMPLEX_UNIT_SP, 16);//字体大小
            tv.setLayoutParams(lp);
            addView(tv);
        }

    }
}

```

##### TabFragment.java 动态生成四个子fragment

``` java
package yellow.com.stickynavlayout;

import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.GridView;
import android.widget.Toast;

import java.util.ArrayList;
import java.util.List;

/**
 * 动态生成子Fragment
 */
public class TabFragment extends Fragment {

    private int pageNum = -1;//第几页选项卡
    private GridView mListView;
    private String mTitle;
    private List<String> mDatas = new ArrayList<>();

    public TabFragment() {
    }

    public TabFragment(int pageNum) {
        this.pageNum = pageNum;
    }

    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_tab, container, false);
        mListView = (GridView) view.findViewById(R.id.id_stickynavlayout_innerscrollview);
        initPage();
        return view;
    }


    private int currentPage = 1;

    private void initPage() {
        switch (pageNum) {
            case 0:
                mTitle = "GridView的上拉加载";
                mListView.setNumColumns(2);//两栏
                loadSpareItems(currentPage);
                break;
            case 1:
                mTitle = "GridView的上拉加载";
                mListView.setNumColumns(3);//三栏
                loadSpareItems(currentPage);
                break;
            case 2:
                mTitle = "ListView的上拉加载";
                loadSpareItems(currentPage);
                break;
            case 3:
                mTitle = "ListView的上拉加载";
                loadSpareItems(currentPage);
                break;
        }
    }

    private void loadSpareItems(int page) {
        mDatas.clear();
        if (page == 1) {
            for (int i = 0; i < 10; i++) {
                mDatas.add(mTitle + " -> " + i);
            }
        } else if (page > 1 && page <= 10) {
            currentPage = page;
            for (int i = (page - 1) * 10; i < page * 10; i++) {
                mDatas.add(mTitle + " -> " + i);
            }
            Toast.makeText(getActivity(), "加载更多成功", Toast.LENGTH_SHORT).show();
        } else {
            Toast.makeText(getActivity(), "没有更多数据啦", Toast.LENGTH_SHORT).show();
        }

        mListView.setAdapter(new ArrayAdapter<String>(getActivity(), R.layout.item_mine, R.id.id_info, mDatas));

    }

}

```

#####  MainActivity.java

``` java
package yellow.com.stickynavlayout;

import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentActivity;
import android.support.v4.app.FragmentPagerAdapter;
import android.support.v4.view.ViewPager;

import yellow.com.stickynavlayout.view.SimpleViewPagerIndicator;


public class MainActivity extends FragmentActivity {

    private String[] mTitles = new String[]{100 + "\n晒物", "100\n众测", "100\n关注", "100\n粉丝"};

    private TabFragment[] mFragments = new TabFragment[mTitles.length];

    private ViewPager mViewPager;
    private SimpleViewPagerIndicator mIndicator;
    private FragmentPagerAdapter mAdapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initViews();
        initDatas();
        initEvents();
    }

    private void initEvents() {
        
        mViewPager.setOnPageChangeListener(new ViewPager.OnPageChangeListener() {
            @Override
            public void onPageSelected(int position) {
            }

            @Override
            public void onPageScrolled(int position, float positionOffset,
                                       int positionOffsetPixels) {
                mIndicator.scroll(position, positionOffset);
            }

            @Override
            public void onPageScrollStateChanged(int state) {

            }
        });
    }

    private void initDatas() {
        mIndicator.setTitle(mTitles);
        mFragments[0] = new TabFragment(0);
        mFragments[1] = new TabFragment(1);
        mFragments[2] = new TabFragment(2);
        mFragments[3] = new TabFragment(3);
        mAdapter = new FragmentPagerAdapter(getSupportFragmentManager()) {
            @Override
            public Fragment getItem(int position) {
                return mFragments[position];
            }

            @Override
            public int getCount() {
                return mTitles.length;
            }
        };
        mViewPager.setAdapter(mAdapter);
        mViewPager.setCurrentItem(0);
    }

    private void initViews() {
        mViewPager = (ViewPager) findViewById(R.id.id_stickynavlayout_viewpager);
        mIndicator = (SimpleViewPagerIndicator) findViewById(R.id.id_stickynavlayout_indicator);
    }
}
```
#### 效果
![enter description here][1]


  [1]: ./images/%E8%87%AA%E5%AE%9A%E4%B9%89360%E8%BD%AF%E4%BB%B6%E8%AF%A6%E6%83%85%E7%95%8C%E9%9D%A2.gif "自定义360软件详情界面"