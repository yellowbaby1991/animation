### 使用ViewDragHelper实现自定义抽屉效果
#### 思路

 1. 自定义ViewGroup--DragDownLayout，中间包含一个拖拽View和一个内容View
 2. 移动的控制全部在ViewDragHelper.Callback中进行，如计算拖拽的时候子空间的大小，释放鼠标后菜单是去顶端还是底部，限制内容框不允许拖拽等等
 3.  自定义内部listen来监听开关菜单效果，移动则使用dragHelper.smoothSlideViewTo配合computeScroll实现


#### 代码

##### DragDownLayout.java

``` java
package yellow.com.viewdraghelper;


import android.content.Context;
import android.support.v4.view.ViewCompat;
import android.support.v4.widget.ViewDragHelper;
import android.util.AttributeSet;
import android.util.Log;
import android.view.MotionEvent;
import android.view.View;
import android.view.ViewGroup;

public class DragDownLayout extends ViewGroup {

    private ViewDragHelper dragHelper;

    private View mDragbar, mContentView;//拖拽View和内容View

    private boolean isOpen = true;//内容是否打开着

    //对外的接口
    private OnOpenListener mOnOpenListener;
    private OnCloseListener mOnCloseListener;

    public DragDownLayout(Context context) {
        super(context);
        init();
    }

    public DragDownLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    public DragDownLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    //测量：宽度为父控件的宽度，高度为指定的值
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        mDragbar = getChildAt(0);
        mContentView = getChildAt(1);

        LayoutParams contPar = mDragbar.getLayoutParams();
        int heightContentSpec = MeasureSpec.makeMeasureSpec(contPar.height, MeasureSpec.EXACTLY);
        mDragbar.measure(widthMeasureSpec, heightContentSpec);

        LayoutParams delPar = mContentView.getLayoutParams();
        int heightDelSpec = MeasureSpec.makeMeasureSpec(delPar.height, MeasureSpec.EXACTLY);
        mContentView.measure(widthMeasureSpec, heightDelSpec);

        setMeasuredDimension(MeasureSpec.getSize(widthMeasureSpec), contPar.height + delPar.height);
    }

    //将滑动交给dragHelper去处理
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        return dragHelper.shouldInterceptTouchEvent(ev);
    }

    //将滑动交给dragHelper去处理
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        dragHelper.processTouchEvent(event);
        return true;
    }

    //定位：通过测量得到的数据进行子控件定位
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        mDragbar.layout(0, 0, getWidth(), mDragbar.getMeasuredHeight());
        mContentView.layout(0, -mContentView.getMeasuredHeight(), getWidth(), 0);
    }

    private void init() {
        dragHelper = ViewDragHelper.create(this, mCallback);
    }

    private ViewDragHelper.Callback mCallback = new ViewDragHelper.Callback() {

        //只允许拖拽顶部的拖拽栏
        @Override
        public boolean tryCaptureView(View child, int pointerId) {
            return child == mDragbar;
        }

        //顶部被拖拽的时候计算内容栏应该如何变化进行重新定位
        @Override
        public void onViewPositionChanged(View changedView, int left, int top, int dx, int dy) {
            mContentView.layout(0, top - mContentView.getHeight(), getWidth(), top);
            if (mContentView.getBottom() > mContentView.getHeight() / 2) {
                isOpen = true;
            } else {
                isOpen = false;
            }
        }

        //控制视图纵轴移动的距离
        @Override
        public int clampViewPositionVertical(View child, int top, int dy) {
            int topBound = getPaddingTop();
            int bottomBound = getHeight() - mDragbar.getHeight();
            Log.d("TAG","clampViewPositionVertical:" + Math.min(Math.max(topBound, top), bottomBound));
            return Math.min(Math.max(topBound, top), bottomBound);
        }


        //获得视图在纵轴移动的距离
        @Override
        public int getViewVerticalDragRange(View child) {
            Log.d("TAG","getViewVerticalDragRange:" + mContentView.getHeight());
            return mContentView.getHeight();
        }

        //是释放鼠标的时候如果下拉过半就直接到底，反之回到顶部
        @Override
        public void onViewReleased(View releasedChild, float xvel, float yvel) {
            if (mContentView.getBottom() > mContentView.getHeight() / 2) {
                smoothToBottom();
            } else {
                smoothToTop();
            }
            invalidate();
        }
    };

    //滑动到顶
    private void smoothToTop() {
        if (dragHelper.smoothSlideViewTo(mDragbar, getPaddingLeft(), getPaddingTop())) {
            ViewCompat.postInvalidateOnAnimation(this);
            isOpen = false;
            Log.d("TAG", "isOpen:" + isOpen);
            if (mOnCloseListener != null) {
                mOnCloseListener.close();
            }

        }
    }

    //滑动到底
    private void smoothToBottom() {
        if (dragHelper.smoothSlideViewTo(mDragbar, getPaddingLeft(), getHeight() - getPaddingBottom() - mDragbar.getHeight())) {
            ViewCompat.postInvalidateOnAnimation(this);
            isOpen = true;
            Log.d("TAG", "isOpen:" + isOpen);
            if (mOnOpenListener != null) {
                mOnOpenListener.open();
            }
        }
    }

    @Override
    public void computeScroll() {
        super.computeScroll();
        if (dragHelper.continueSettling(true)) {
            ViewCompat.postInvalidateOnAnimation(this);
        }
    }

    public boolean isOpen() {
        return isOpen;
    }

    public void setOnOpenListener(OnOpenListener onOpenListener) {
        this.mOnOpenListener = onOpenListener;
    }

    public void setmOnCloseListener(OnCloseListener mOnCloseListener) {
        this.mOnCloseListener = mOnCloseListener;
    }

    public void openContent() {
        if (!isOpen) smoothToBottom();
    }

    public void closeContent() {
        if (isOpen) smoothToTop();
    }

    public interface OnOpenListener {
        void open();
    }

    public interface OnCloseListener {
        void close();
    }


}

```

##### activity_main.xml

``` java
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <yellow.com.viewdraghelper.DragDownLayout
        android:id="@+id/myDragDownLayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#e0ffff">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="56dp"
            android:background="#e000ff"
            android:gravity="center_vertical"
            android:paddingLeft="15dp"
            android:text="欢迎"
            android:textColor="#ffffff"
            android:textSize="19sp">

        </TextView>

        <TextView
            android:id="@+id/content"
            android:layout_width="match_parent"
            android:layout_height="150dp"
            android:background="#e000ff"
            android:gravity="center"
            android:text="Content"
            android:textSize="20sp">

        </TextView>
    </yellow.com.viewdraghelper.DragDownLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:gravity="center"
        android:orientation="horizontal"
        android:padding="20dp">

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_margin="20dp"
            android:onClick="openContent"
            android:text="open" />


        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_margin="30dp"
            android:onClick="closeContent"
            android:text="close" />


    </LinearLayout>
</RelativeLayout>
```

##### MainActivity.java

``` java
package yellow.com.viewdraghelper;

import android.app.Activity;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.TextView;
import android.widget.Toast;

public class MainActivity extends Activity {

    private DragDownLayout mDragDownLayout;
    private TextView content;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        content = (TextView) findViewById(R.id.content);
        content.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(MainActivity.this,"可点击",Toast.LENGTH_SHORT).show();
            }
        });

        mDragDownLayout = (DragDownLayout) findViewById(R.id.myDragDownLayout);
        mDragDownLayout.setOnOpenListener(new DragDownLayout.OnOpenListener() {
            @Override
            public void open() {
                Toast.makeText(MainActivity.this,"open",Toast.LENGTH_SHORT).show();
            }
        });
    }

    public void openContent(View view) {
        mDragDownLayout.openContent();
    }

    public void closeContent(View view) {
        mDragDownLayout.closeContent();
    }
}

```

#### 效果

![enter description here][1]


  [1]: ./images/%E4%BD%BF%E7%94%A8ViewDragHelper%E5%AE%9E%E7%8E%B0%E8%87%AA%E5%AE%9A%E4%B9%89%E6%8A%BD%E5%B1%89%E6%95%88%E6%9E%9C.gif "使用ViewDragHelper实现自定义抽屉效果"