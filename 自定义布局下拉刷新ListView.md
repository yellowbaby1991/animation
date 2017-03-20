### 自定义布局下拉刷新ListView
#### 思路
##### 布局

 1. 刷新头布局较简单，由图片+文本组成，图片根据状态切换箭头和进度条，文本用来显示一次刷新时间
 
 2. 自定义一个LinearLayout，命名RefreshableView，里面只包含了一个ListView

