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

 1. 布局为一个FrameLayout（大图片） + RecyclerView（图片列表）的垂直布局

``` xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <FrameLayout
        android:layout_width="fill_parent"
        android:layout_height="0dp"
        android:layout_weight="1">

        <ImageView
            android:id="@+id/id_content"
            android:layout_width="fill_parent"
            android:layout_height="fill_parent"
            android:layout_gravity="center"
            android:layout_margin="10dp"
            android:scaleType="centerCrop"
            android:src="@drawable/ic_launcher" />
    </FrameLayout>

    <yellow.com.recyclerview.MyRecyclerView
        android:id="@+id/id_recyclerview_horizontal"
        android:layout_width="match_parent"
        android:layout_height="120dp"
        android:layout_centerVertical="true"
        android:background="#FF0000"
        android:scrollbars="none" />

</LinearLayout>
```

 2. 创建自定义MyRecyclerView，然后自定义OnScrollListener监听，当滑动到新View的时候回调接口改变大图片
 
``` java
package yellow.com.recyclerview;

import android.content.Context;
import android.support.annotation.Nullable;
import android.support.v7.widget.RecyclerView;
import android.util.AttributeSet;
import android.view.View;

public class MyRecyclerView extends RecyclerView {

    private View mCurrentView;

    private OnItemScrollChangeListener mItemScrollChangeListener;

    public void setOnItemScrollChangeListener(
            OnItemScrollChangeListener mItemScrollChangeListener) {
        this.mItemScrollChangeListener = mItemScrollChangeListener;
    }

    public interface OnItemScrollChangeListener {
        void onChange(View view, int position);
    }

    private OnScrollListener mOnScrollListener = new OnScrollListener() {
        @Override
        public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
            super.onScrollStateChanged(recyclerView, newState);
        }

        //只有当滑动到新图片的时候才回调接口
        @Override
        public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
            View newView = getChildAt(0);
            if (newView != null && newView != mCurrentView){
                mCurrentView = newView;
                mItemScrollChangeListener.onChange(mCurrentView, getChildPosition(mCurrentView));
            }
        }
    };

    public MyRecyclerView(Context context) {
        this(context, null);
    }

    public MyRecyclerView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public MyRecyclerView(Context context, @Nullable AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        this.setOnScrollListener(mOnScrollListener);
    }

    //在滑动的时候回调监听接口
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        super.onLayout(changed, l, t, r, b);
        mCurrentView = getChildAt(0);
        if (mItemScrollChangeListener != null) {
            mItemScrollChangeListener.onChange(mCurrentView, getChildPosition(mCurrentView));
        }
    }

}

```

 3. 创建适配器
 
``` java
package yellow.com.recyclerview;

import android.content.Context;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.TextView;

import java.util.List;

public class GalleryAdapter extends RecyclerView.Adapter<GalleryAdapter.ViewHolder> {//1、继承RecyclerView.Adapter

    private LayoutInflater mInflater;
    private List<Integer> mDatas;

    public GalleryAdapter(Context context, List<Integer> datas) {
        mInflater = LayoutInflater.from(context);
        mDatas = datas;
    }

    //3、重写适配器中的方法
    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = mInflater.inflate(R.layout.item, parent, false);
        ViewHolder viewHolder = new ViewHolder(view);
        viewHolder.mImg = (ImageView) view.findViewById(R.id.id_index_gallery_item_image);
        return viewHolder;
    }

    //3、重写适配器中的方法
    @Override
    public void onBindViewHolder(final ViewHolder holder, final int position) {
        holder.mImg.setImageResource(mDatas.get(position));
        if (mOnItemClickLitener != null) {
            holder.itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    mOnItemClickLitener.onItemClick(holder.itemView, position);
                }
            });
        }
    }

    @Override
    public int getItemCount() {
        return mDatas.size();
    }

    public static class ViewHolder extends RecyclerView.ViewHolder {//2、创建一个类ViewHolder继承RecyclerView.ViewHolder
        public ViewHolder(View arg0) {
            super(arg0);
        }

        ImageView mImg;
        TextView mTxt;
    }

    /**
     * ItemClick的回调接口
     */
    public interface OnItemClickLitener {
        void onItemClick(View view, int position);
    }

    public void setOnItemClickLitener(OnItemClickLitener mOnItemClickLitener) {
        this.mOnItemClickLitener = mOnItemClickLitener;
    }

    private OnItemClickLitener mOnItemClickLitener;
}

```

 4. 在MainActivity中设置布局管理器，数据源和适配器