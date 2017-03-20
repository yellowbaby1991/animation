### 使用RecyclerView实现Gallery效果
#### RecyclerView简介

 1. RecyclerView是ListView的升级版，性能更好，不需要自己写ViewHolder
 2. RecyclerView本身不关心任何View，只负责回收和重用，所有的东西都给了一些插件化的类来处理，需要新布局，使用另一个LayoutManager，需要新动画，使用另一个ItemAnimator
 3. 没有onItemClickListener，全部交给用户来进行自定义操作

#### 使用方法

 1. xm中进行定义，和ListView一样
 2. 设置LayoutManager进行布局，有三种布局方式，线性布局，网格布局，瀑布流布局
 3. 创建适配器
	 - 继承RecyclerView.Adapter
	 - 创建一个类ViewHolder继承RecyclerView.ViewHolder
	 - 重写适配器中的方法

#### Gallery效果思路

 1. 布局为一个FrameLayout + RecyclerView的垂直布局
 
 2. 1