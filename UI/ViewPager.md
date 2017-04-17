## ViewPager

### 1. 常用方法

| setAdapter()                           |               |
| -------------------------------------- | ------------- |
| setOnPageChangeListner()               |               |
| addOnPageChangeListener()              |               |
| getCurrentItem()                       |               |
| setCurrentItem(int position)           |               |
| setCurrentItem(int pos,Boolean smooth) | 参数2表示是否执行切换动画 |
| setPageTransformer()                   | 设置切换动画        |
| setId()                                |               |
| setOffscreenPageLimit()                | 设置预加载的页面数     |
| clipChildren()                         |               |
| beginFakeDrag()                        | 全权控制滑动事件      |
| endFakeDrag()                          | 终止控制滑动事件      |
| isFakeDragging()                       |               |
| fakeDragBy()                           |               |

## 2. 预加载

l  Fragment

setUserVisibleHint：设置Fragment可见状态

getUserVisibleHint：获取Fragment可见状态

在可见状态的时候再加载数据

l  ViewPager

setOffscreenPageLimit(int limit)：其中参数可以设为0或者1，参数小于1时，会默认用1来作为参数，未设置之前，ViewPager会默认加载两个Fragment。

 

修改源码：private static final int DEFAULT_OFFSCREEN_PAGES = 1;

# 第二章  Adapter

## 1. PagerAdapter

```
class MyPagerAdapter extends PagerAdapter{



    @Override

    public int getCount() {

        return 0;

    }



    /**

     * 类似getView()方法

     */

    @Override

    public Object instantiateItem(ViewGroup container, int position) {

        return super.instantiateItem(container, position);

    }



    /**

     * 滑动一半的时候，判断当前滑动的view和即将进来的view是否是同一个view，true则复用，false则重新创建

     */

    @Override

    public boolean isViewFromObject(View view, Object object) {

        return view == object;

    }



    /**

     * 最多保存3个page

     *

     */

    @Override

    public void destroyItem(ViewGroup container, int position, Object object) {

        super.destroyItem(container, position, object);

    }



    @Override

    public CharSequence getPageTitle(int position) {

        return super.getPageTitle(position);

    }

}
```

 

## 2. FragmentPagerAdapter

在同级屏幕只有少量的几个固定页面时，使用这个最好

```
class MyFragmenAdapter extends FragmentPagerAdapter{



    public MyFragmenAdapter(FragmentManager fm) {

        super(fm);

    }



    @Override

    public Fragment getItem(int position) {

        return null;

    }



    @Override

    public int getCount() {

        return 0;

    }



    @Override

    public CharSequence getPageTitle(int position) {

        return super.getPageTitle(position);

    }

}
```

 

## 3. FragmentStatePagerAdapter

当根据对象集的数量来划分页面，即一开始页面的数量未确定时，使用这个最好。当用户切换到其他页面时，fragment会被销毁来降低内存消耗

## 4. 区别

FragmentPagerAdapter：该类内的每一个生成的 Fragment 都将保存在内存之中，因此适用于那些相对静态的页，数量也比较少的那种；如果需要处理有很多页，并且数据动态性较大、占用内存较多的情况，应该使用FragmentStatePagerAdapter

 

FragmentStatePagerAdapter：该PagerAdapter的实现将只保留当前页面，当页面离开视线后，就会被消除，释放其资源；而在页面需要显示时，生成新的页面(就像ListView的实现一样)。这么实现的好处就是当拥有大量的页面时，不必在内存中占用大量的内存

 

# 第三章  OnPageChangeListener

```
viewPager.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {

    @Override

    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {

        

    }



    @Override

    public void onPageSelected(int position) {



    }



    @Override

    public void onPageScrollStateChanged(int state) {



    }

});
```

# 第四章  PageTransformer

当滑动到正全屏时，position 是0，而向左滑动，使得右边刚好有一部被进入屏幕时，position 是1，如果前一页和下一页基本各在屏幕占一半时，前一页的position 是-0.5，后一页的posiotn是0.5

# 第五章  PagerTitleStrip

## 1. PagerTitleStrip

不可交互

## 2. PagerTabStrip

可交互

setTabIndicatorColorResource() 设置下划线颜色

 

 

# 第六章  ViewPagerIndicator