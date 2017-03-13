### ViewPage切换碎片效果
#### 思路

 1. ViewPage可以使用FragmentPagerAdapter来显示多个Fragment进行切换
 2. 设置ViewPage的PageChangeListener来更新引导线的位置
 3. 设置TextView的ClickListener来切换不同的Page


#### 代码
##### activity_main.xml

``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <include layout="@layout/activity_main_top_tab"></include>

    <android.support.v4.view.ViewPager
        android:id="@+id/id_page_vp"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"></android.support.v4.view.ViewPager>

</LinearLayout>

```

##### activity_main_top.xml 导航条

``` xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">

    <LinearLayout
        android:id="@+id/id_switch_tab_ll"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:baselineAligned="false"
        android:orientation="horizontal">

        <LinearLayout
            android:id="@+id/id_tab_chat_ll"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:gravity="center"
            android:orientation="horizontal"
            android:padding="10dip">

            <TextView
                android:id="@+id/id_chat_tv"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:gravity="center"
                android:text="聊天"
                android:textColor="#0000FF"
                android:textSize="15dip" />
        </LinearLayout>

        <LinearLayout
            android:id="@+id/id_tab_friend_ll"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:clickable="true"
            android:gravity="center"
            android:orientation="horizontal"
            android:padding="10dip"
            android:saveEnabled="false">

            <TextView
                android:id="@+id/id_friend_tv"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:gravity="center"
                android:text="好友"
                android:textColor="#000000"
                android:textSize="15dip" />
        </LinearLayout>

        <LinearLayout
            android:id="@+id/id_tab_contacts_ll"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:focusable="false"
            android:gravity="center"
            android:orientation="horizontal"
            android:padding="10dip">

            <TextView
                android:id="@+id/id_contacts_tv"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:gravity="center"
                android:text="通讯录"
                android:textColor="#000000"
                android:textSize="15dip" />
        </LinearLayout>
    </LinearLayout>

    <ImageView
        android:id="@+id/id_tab_line_iv"
        android:layout_width="200dp"
        android:layout_height="wrap_content"
        android:contentDescription="tab"
        android:background="@drawable/tab_selected_pressed_holo" >
    </ImageView>

    <View
        android:layout_width="match_parent"
        android:layout_height="1dp"
        android:background="#000000" />

</LinearLayout>


```

##### activity_tab_chat.xml 聊天界面

``` xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:text="聊天界面"
        android:textColor="#FF0000"
        android:textSize="20sp"></TextView>

</LinearLayout>

```

##### activity_tab_contact.xml 通讯录界面

``` xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:text="通讯录"
        android:textColor="#FF0000"
        android:textSize="20sp"></TextView>

</LinearLayout>

```

##### activity_tab_friend.xml 好友界面

``` xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:text="好友"
        android:textColor="#FF0000"
        android:textSize="20sp"></TextView>

</LinearLayout>

```

##### ChatFragment.java 聊天fragment

``` java
package yellow.com.viewpagewithfragment;

import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

public class ChatFragment extends Fragment {

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        super.onCreateView(inflater, container, savedInstanceState);
        View chatView = inflater.inflate(R.layout.activity_tab_chat,container,false);
        return chatView;
    }
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
    }
}

```

##### ContactsFragment.java 通讯录fragment

``` java
package yellow.com.viewpagewithfragment;

import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

public class ContactsFragment extends Fragment {

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        super.onCreateView(inflater, container, savedInstanceState);
        View chatView = inflater.inflate(R.layout.activity_tab_contact,container,false);
        return chatView;
    }
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
    }
}

```

##### FriendFragment.java 好友fragment

``` java
package yellow.com.viewpagewithfragment;

import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

public class FriendFragment extends Fragment {

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        super.onCreateView(inflater, container, savedInstanceState);
        View chatView = inflater.inflate(R.layout.activity_tab_friend,container,false);
        return chatView;
    }
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
    }
}

```

##### FragmentAdapter.java

``` java
package yellow.com.viewpagewithfragment;


import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentPagerAdapter;

import java.util.ArrayList;
import java.util.List;

public class FragmentAdapter extends FragmentPagerAdapter {

    List<Fragment> fragmentList = new ArrayList<>();

    public FragmentAdapter(FragmentManager fm, List<Fragment> fragmentList) {
        super(fm);
        this.fragmentList = fragmentList;
    }

    @Override
    public Fragment getItem(int position) {
        return fragmentList.get(position);
    }

    @Override
    public int getCount() {
        return fragmentList.size();
    }
}

```

##### MainActivity.java

``` java
package yellow.com.viewpagewithfragment;

import android.graphics.Color;
import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentActivity;
import android.support.v4.view.ViewPager;
import android.util.DisplayMetrics;
import android.view.View;
import android.widget.ImageView;
import android.widget.LinearLayout;
import android.widget.TextView;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends FragmentActivity {

    private ViewPager mPageVp;
    private FragmentAdapter mFragmentAdapter;
    private List<Fragment> mFragmentList = new ArrayList<Fragment>();

    /**
     * Fragment
     */
    private ChatFragment mChatFg;
    private FriendFragment mFriendFg;
    private ContactsFragment mContactsFg;


    /**
     * Tab显示内容TextView
     */
    private TextView mTabChatTv, mTabContactsTv, mTabFriendTv;

    /**
     * Tab的那个引导线
     */
    private ImageView mTabLineIv;

    /**
     * ViewPager的当前选中页
     */
    private int currentIndex;

    /**
     * 屏幕的宽度
     */
    private int screenWidth;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        findById();
        init();
        initTabLineWidth();
    }

    /**
     * 初始化导航条的宽度
     */
    private void initTabLineWidth() {
        DisplayMetrics dpMetrics = new DisplayMetrics();
        getWindow().getWindowManager().getDefaultDisplay().getMetrics(dpMetrics);
        screenWidth = dpMetrics.widthPixels;
        LinearLayout.LayoutParams lp = (LinearLayout.LayoutParams) mTabLineIv.getLayoutParams();
        lp.width = screenWidth / 3;
        mTabLineIv.setLayoutParams(lp);
    }

    private void init() {
        mFriendFg = new FriendFragment();
        mContactsFg = new ContactsFragment();
        mChatFg = new ChatFragment();
        mFragmentList.add(mChatFg);
        mFragmentList.add(mFriendFg);
        mFragmentList.add(mContactsFg);

        mFragmentAdapter = new FragmentAdapter(this.getSupportFragmentManager(), mFragmentList);
        mPageVp.setAdapter(mFragmentAdapter);
        mPageVp.setCurrentItem(0);

        mTabChatTv.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mPageVp.setCurrentItem(0);
            }
        });
        mTabFriendTv.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mPageVp.setCurrentItem(1);
            }
        });
        mTabContactsTv.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mPageVp.setCurrentItem(2);
            }
        });

        //切换导航条的位置
        mPageVp.setOnPageChangeListener(new ViewPager.OnPageChangeListener() {
            @Override
            public void onPageScrolled(int position, float offset, int positionOffsetPixels) {
                LinearLayout.LayoutParams lp = (LinearLayout.LayoutParams) mTabLineIv.getLayoutParams();
                if (currentIndex == 0 && position == 0)// 0->1
                {
                    lp.leftMargin = (int) (offset * (screenWidth * 1.0 / 3) + currentIndex
                            * (screenWidth / 3));

                } else if (currentIndex == 1 && position == 0) // 1->0
                {
                    lp.leftMargin = (int) (-(1 - offset)
                            * (screenWidth * 1.0 / 3) + currentIndex
                            * (screenWidth / 3));

                } else if (currentIndex == 1 && position == 1) // 1->2
                {
                    lp.leftMargin = (int) (offset * (screenWidth * 1.0 / 3) + currentIndex
                            * (screenWidth / 3));
                } else if (currentIndex == 2 && position == 1) // 2->1
                {
                    lp.leftMargin = (int) (-(1 - offset)
                            * (screenWidth * 1.0 / 3) + currentIndex
                            * (screenWidth / 3));
                }
                mTabLineIv.setLayoutParams(lp);
            }

            //设置被选中的字体
            @Override
            public void onPageSelected(int position) {
                resetTextView();
                switch (position) {
                    case 0:
                        mTabChatTv.setTextColor(Color.BLUE);
                        break;
                    case 1:
                        mTabFriendTv.setTextColor(Color.BLUE);
                        break;
                    case 2:
                        mTabContactsTv.setTextColor(Color.BLUE);
                        break;
                }
                currentIndex = position;
            }

            /**
             * state滑动中的状态 有三种状态（0，1，2） 1：正在滑动 2：滑动完毕 0：什么都没做。
             */
            @Override
            public void onPageScrollStateChanged(int state) {
            }
        });
    }

    private void findById() {
        mTabContactsTv = (TextView) this.findViewById(R.id.id_contacts_tv);
        mTabChatTv = (TextView) this.findViewById(R.id.id_chat_tv);
        mTabFriendTv = (TextView) this.findViewById(R.id.id_friend_tv);
        mTabLineIv = (ImageView) this.findViewById(R.id.id_tab_line_iv);
        mPageVp = (ViewPager) this.findViewById(R.id.id_page_vp);
    }

    /**
     * 重置颜色
     */
    private void resetTextView() {
        mTabChatTv.setTextColor(Color.BLACK);
        mTabFriendTv.setTextColor(Color.BLACK);
        mTabContactsTv.setTextColor(Color.BLACK);
    }
}

```

#### 效果
![enter description here][1]


  [1]: ./images/ViewPage%E5%88%87%E6%8D%A2%E7%A2%8E%E7%89%87.gif "ViewPage切换碎片"