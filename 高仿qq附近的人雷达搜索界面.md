### 高仿qq附近的人雷达搜索界面
#### 思路

##### 底部名片

1. 整体使用一个自定义的ViewPager，使用自定义主要是为了计算滑动速度
``` java
public class CustomViewPager extends ViewPager {
		...
		public boolean dispatchTouchEvent(MotionEvent ev) {
        float x = ev.getX();
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                downTime = System.currentTimeMillis();
                lastX = x;
                break;
            case MotionEvent.ACTION_MOVE:
                x = ev.getX();
                break;
            case MotionEvent.ACTION_UP:
                mSpeed = (x - lastX) * 1000 / (System.currentTimeMillis() - downTime);
                break;
        }
        return super.dispatchTouchEvent(ev);
    }
		...
		}
```

2. 设置clipChildren属性使得可以在一页显示多个page
``` xml
 <yellow.com.qqnearby.view.CustomViewPager
	android:id="@+id/vp"
	android:layout_width="130dp"
	android:layout_height="160dp"
	android:layout_centerInParent="true"
	android:layout_marginLeft="120dp"
	android:layout_marginRight="120dp"
	android:clipChildren="false"></yellow.com.qqnearby.view.CustomViewPager>
```
3. 使用shape-drawable作为背景来显示圆角名片
``` xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <corners android:radius="5dp"></corners>
    <solid android:color="#aaEBEBEB"></solid>
</shape>
```
	4. 自定义PagerAdapter来显示名片细节
	

``` stylus
enter code here
```




