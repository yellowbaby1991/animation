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
@Override
    public boolean onTouchEvent(MotionEvent ev) {
        int action = ev.getAction();
        if (isSliding) {//代表需要显示删除按钮
            switch (action) {
                case MotionEvent.ACTION_MOVE:
                    int[] location = new int[2];
                    mCurrentView.getLocationOnScreen(location);
                    mPopupWindow.setAnimationStyle(R.style.popwindow_delete_btn_anim_style);
                    mPopupWindow.update();
                    mPopupWindow.showAtLocation(mCurrentView, Gravity.LEFT | Gravity.TOP,
                            location[0] + mCurrentView.getWidth(), location[1] + mCurrentView.getHeight() / 2
                                    - mPopupWindowHeight / 2);
                    mDelBtn.setOnClickListener(new OnClickListener() {
                        @Override
                        public void onClick(View v) {
                            if (mListener != null) {
                                mListener.clickHappend(mCurrentViewPos);
                                mPopupWindow.dismiss();
                            }
                        }
                    });
                    break;
                case MotionEvent.ACTION_UP:
                    isSliding = false;
                    break;
            }
            return true;
        }
        return super.onTouchEvent(ev);
    }
```

#### 代码

[源码下载][1]

#### 效果

![enter description here][2]


  [1]: http://download.csdn.net/detail/lmj623565791/7148325
  [2]: http://img.blog.csdn.net/20140404233334812