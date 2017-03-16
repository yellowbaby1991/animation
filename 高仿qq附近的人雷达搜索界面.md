### 高仿qq附近的人雷达搜索界面
#### 思路

 - 底部名片
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
	2. 整体使用一个自定义的ViewPager，使用自定义主要是为了计算滑动速度
		``` java
		enter code here
		```


