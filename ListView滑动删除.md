### ListView滑动删除
#### 思路

 1. 很明显需要继承ListView然后自定义控件实现，按钮的显示采用PopupWindow，重写dispatchTouchEvent判断是否需要响应滑动
 
 2. 1