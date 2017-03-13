### HorizontalScrollView结合ViewPager高仿QQ
#### 思路

 1. 根据上一篇的思路，同样使用HorizontalScrollView来实现侧滑菜单，只是这次左边的菜单是一个ListView，右侧的内容变成了一个ViewPager
 2. HorizontalScrollView和ViewPager会产生滑动冲突，解决办法如下：
	 - 重写ViewPager的dispatchTouchEvent方法，让父类不拦截事件
	 - 当处于ViewPage第一页，并且向左滑动的时候，让父类不再拦截

  
 #### 代码

https://github.com/yellowbaby1991/qq.git

#### 效果

![enter description here][1]


  [1]: ./images/ViewPage+HorizontalScrollView%E9%AB%98%E4%BB%BFQQ.gif "ViewPage+HorizontalScrollView高仿QQ"
