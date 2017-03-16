### 高仿qq附近的人雷达搜索界面
#### 思路

##### 底部滚动名片

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
``` java
class ViewPagerAdapter extends PagerAdapter {

        @Override
        public Object instantiateItem(ViewGroup container, int position) {
            final Info info = mDatas.get(position);
            View view = LayoutInflater.from(MainActivity.this).inflate(R.layout.viewpager_layout, null);
            ImageView iv_protrait = (ImageView) view.findViewById(R.id.iv);
            ImageView iv_sex = (ImageView) view.findViewById(R.id.iv_sex);
            TextView tv_name = (TextView) view.findViewById(R.id.tv_name);
            TextView tv_distance = (TextView) view.findViewById(R.id.tv_distance);
            tv_name.setText(info.getName());
            tv_distance.setText(info.getDistance() + "km");
            iv_protrait.setImageResource(info.getPortraitId());
            if (info.getSex()) {
                iv_sex.setImageResource(R.drawable.girl);
            } else {
                iv_sex.setImageResource(R.drawable.boy);
            }
            iv_protrait.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    Toast.makeText(MainActivity.this, "这是 " + info.getName(), Toast.LENGTH_SHORT).show();
                }
            });
            container.addView(view);
            return view;
        }

        @Override
        public int getCount() {
            return mImgs.length;
        }

        @Override
        public boolean isViewFromObject(View view, Object object) {
            return view == object;
        }

        @Override
        public void destroyItem(ViewGroup container, int position, Object object) {
            View view = (View) object;
            container.removeView(view);
        }
```






##### 雷达界面

 1. 先画最外圈的5个白色线圈
 
``` java
 //画五个外围的圆线,五个圈的区别只有比例的不同而已
    private void drawCircle(Canvas canvas) {
        canvas.drawCircle(mWidth / 2, mHeight / 2, mWidth * circleProportion[1], mPaintLine);
        canvas.drawCircle(mWidth / 2, mHeight / 2, mWidth * circleProportion[2], mPaintLine);
        canvas.drawCircle(mWidth / 2, mHeight / 2, mWidth * circleProportion[3], mPaintLine);
        canvas.drawCircle(mWidth / 2, mHeight / 2, mWidth * circleProportion[4], mPaintLine);
        canvas.drawCircle(mWidth / 2, mHeight / 2, mWidth * circleProportion[5], mPaintLine);
    }
	
	private void init() {
        mPaintLine = new Paint();
        mPaintLine.setColor(getResources().getColor(R.color.line_color_blue));
        mPaintLine.setAntiAlias(true);
        mPaintLine.setStrokeWidth(1);
        mPaintLine.setStyle(Paint.Style.STROKE);
		...
    }
```

 2. 然后画阴影
 
``` java

private void init() {
       ...
  		scanShader = new SweepGradient(mWidth / 2, mHeight / 2,
                new int[]{Color.TRANSPARENT, Color.parseColor("#84B5CA")}, null);
        mPaintScan = new Paint();
        mPaintScan.setStyle(Paint.Style.FILL_AND_STROKE);
    }

private void drawScan(Canvas canvas) {
        canvas.save();//使用save和restore来避免对其他部件的影响
        mPaintScan.setShader(scanShader);
        canvas.concat(matrix);
        canvas.drawCircle(mWidth / 2, mHeight / 2, mWidth * circleProportion[4], mPaintScan);
        canvas.restore();
    }
```
 3. 阴影post一个线程让它转动起来

``` java
private Runnable run = new Runnable() {
        @Override
        public void run() {
            scanAngle = (scanAngle + scanSpeed) % 360;
            matrix.postRotate(scanSpeed, mWidth / 2, mHeight / 2);
            invalidate();
            postDelayed(run, 130);
            ...
        }
    };
```

4. 最后画中间的图标

``` java
private void drawCenterIcon(Canvas canvas) {
        canvas.drawBitmap(centerBitmap, null,
                new Rect(
                        (int) (mWidth / 2 - mWidth * circleProportion[0]),
                        (int) (mHeight / 2 - mWidth * circleProportion[0]),
                        (int) (mWidth / 2 + mWidth * circleProportion[0]),
                        (int) (mHeight / 2 + mWidth * circleProportion[0])
                ), mPaintCircle);
    }
```




##### 雷达扫描出现的小圈
 1. 整体布局为，一个RadarViewGroup下一个雷达RadarView界面和若干个小圆圈CircleView，在RadarViewGroup中计算出雷达和圆圈的位置，然后分别layout，如果没有角度就不layout等待扫描
``` java
@Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        int childCount = getChildCount();
        View view = findViewById(R.id.id_scan_circle);
        if (view != null) {
            view.layout(0, 0, view.getMeasuredWidth(), view.getMeasuredHeight());
        }
        for (int i = 0; i < childCount; i++) {
            final int j = i;
            final View child = getChildAt(i);
            if (child.getId() == R.id.id_scan_circle) {
                continue;
            }
            //设置CircleView小圆点的坐标信息
            //坐标 = 旋转角度 * 半径 * 根据远近距离的不同计算得到的应该占的半径比例
            ((CircleView) child).setDisX((float) Math.cos(Math.toRadians(scanAngleList.get(i - 1) - 5))
                    * ((CircleView) child).getProportion() * mWidth / 2);
            ((CircleView) child).setDisY((float) Math.sin(Math.toRadians(scanAngleList.get(i - 1) - 5))
                    * ((CircleView) child).getProportion() * mWidth / 2);
            if (scanAngleList.get(i - 1) == 0) {
                continue;
            }
            //放置Circle小圆点
            child.layout((int) ((CircleView) child).getDisX() + mWidth / 2, (int) ((CircleView) child).getDisY() + mHeight / 2,
                    (int) ((CircleView) child).getDisX() + child.getMeasuredWidth() + mWidth / 2,
                    (int) ((CircleView) child).getDisY() + child.getMeasuredHeight() + mHeight / 2);

            child.setOnClickListener(new OnClickListener() {
                @Override
                public void onClick(View v) {
                    resetAnim(currentShowChild);
                    currentShowChild = (CircleView) child;
                    startAnim(currentShowChild, j - 1);
                    iRadarClickListener.onRadarItemClick(j - 1);
                }
            });
        }
```
 2. 扫描过程中根据距离远近，计算出角度后requestLayout父布局
``` java
@Override
    public void onScanning(int postion, float scanAngle) {
        if (scanAngle == 0) {
            scanAngleList.put(postion, 1f);
        } else {
            scanAngleList.put(postion, scanAngle);
        }
        Log.d("TAG", scanAngle + " ");
        requestLayout();
    }
```
 3. 扫描完所有的点后优先显示距离最近的点，通过设置缩放度来突出小圆圈
``` java
/**
     * 放大CircleView小圆点大小
     *
     * @param object
     * @param position
     */
    private void startAnim(CircleView object, int position) {
        if (object != null) {
            object.setPortraitIcon(mDatas.get(position).getPortraitId());
            ObjectAnimator.ofFloat(object, "scaleX", 2f).setDuration(300).start();
            ObjectAnimator.ofFloat(object, "scaleY", 2f).setDuration(300).start();
        }
    }
	
	/**
     * 恢复CircleView小圆点原大小
     *
     * @param object
     */
    private void resetAnim(CircleView object) {
        if (object != null) {
            object.clearPortaitIcon();
            ObjectAnimator.ofFloat(object, "scaleX", 1f).setDuration(300).start();
            ObjectAnimator.ofFloat(object, "scaleY", 1f).setDuration(300).start();
        }
    }
```


##### 雷达和滚动名片的联动

``` stylus
enter code here
```
