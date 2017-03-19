### ListView滑动删除
#### 思路

 1. 很明显需要继承ListView然后自定义控件实现，按钮的显示采用PopupWindow，重写dispatchTouchEvent判断是否需要响应滑动
``` java
@Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        int action = ev.getAction();
        int x = (int) ev.getX();
        int y = (int) ev.getY();
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                xDown = x;
                yDown = y;
                if (mPopupWindow.isShowing()) {//如果按钮已经显示了就先隐藏
                    dismissPopWindow();
                    return false;
                }
                //获得当前点击的是那一个View
                mCurrentViewPos = pointToPosition(xDown, yDown);
                View view = getChildAt(mCurrentViewPos - getFirstVisiblePosition());
                mCurrentView = view;
                break;
            case MotionEvent.ACTION_MOVE:
                xMove = x;
                yMove = y;
                int dx = xMove - xDown;
                int dy = yMove - yDown;
                if (xMove < xDown && Math.abs(dx) > toughSlop && Math.abs(dy) < toughSlop) {//如果向左滑动，并且滑动距离大于最小滑动距离说明需要响应
                    isSliding = true;
                }
                break;
        }

        return super.dispatchTouchEvent(ev);
    }
```

 2. 在onTouchEvent中显示PopupWindow
``` java
enter code here
```


 3. 1