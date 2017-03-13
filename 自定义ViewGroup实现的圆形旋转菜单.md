### 自定义ViewGroup实现的圆形旋转菜单
#### 思路

 1. 总体布局为背景图片+若干子菜单组成
 2. 在onMeasure进行自身和子菜单的测量计算出每个子菜单的圆心，半径等必要数据，在onLayout中设置每个子菜单的位置
 3. 在dispatchTouchEvent中处理滑动事件，计算移动的角度和速度来重写布局
 4. 如果速度移动大于规定值，post一个线程去快速转动
 5. 给每一个子菜单添加一个监听


#### 代码

https://github.com/hongyangAndroid/Android-CircleMenu


#### 效果

![enter description here][1]

  [1]: ./images/%E8%87%AA%E5%AE%9A%E4%B9%89%E5%9C%86%E5%BD%A2%E8%BD%AC%E5%8A%A8%E8%8F%9C%E5%8D%95.gif "自定义圆形转动菜单"