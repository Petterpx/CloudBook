# ViewPager滑动方向监测

根据 **OnPageChangeListener** 方法的positionOffset判断方向

```java
/**
 * 一个自定义的ViewPager，可以监听ViewPager的滑动方向
 * Created by lwx on 2016-10-10.
 */

public class CustomViewPager extends ViewPager {
    public static final int SLIDE_LEFT_TO_RIGHT = 0x001;  //从左向右滑动
    public static final int SLIDE_RIGHT_TO_LEFT = 0x002;  //从右向左滑动
    private float lastValue;
    private boolean isScrolling = false;  //viewpager是否滑动
    private boolean isRightToLeft = false;  //从右向左滑动
    private boolean isLeftToRight = false;  //从左向右滑动
    private PageStateChangeListener mPageStateChangeListener;

    public CustomViewPager(Context context) {
        super(context, null);
    }

    public CustomViewPager(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    private void init() {
        addOnPageChangeListener(mOnPageChangeListener);
    }

    /**
     * 监听滑动状态
     */
    OnPageChangeListener mOnPageChangeListener = new OnPageChangeListener() {
        @Override
        public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
            if (positionOffset == 0.0){
                return;
            }
            if (lastValue > positionOffset) {
                // 递减，从左向右滑动
                isRightToLeft = false;
                isLeftToRight = true;
            } else if (lastValue < positionOffset) {
                // 递增，从右向左滑动
                isRightToLeft = true;
                isLeftToRight = false;
            } else if (lastValue == positionOffset) {
                isLeftToRight = isRightToLeft = false;
            }
            lastValue = positionOffset;
            isScrolling = true;

            if (mPageStateChangeListener != null){
                mPageStateChangeListener.onPageScrolled(position, positionOffset, positionOffsetPixels);
            }
        }

        @Override
        public void onPageSelected(int position) {
            if (mPageStateChangeListener != null){
                mPageStateChangeListener.onPageSelected(position);
            }
        }
        @Override
        public void onPageScrollStateChanged(int state) {
            if (mPageStateChangeListener == null){
                return;
            }

            mPageStateChangeListener.onPageScrollStateChanged(state);
            if (isScrolling){
                if (state == ViewPager.SCROLL_STATE_IDLE) {
                    if (isLeftToRight) {
                        mPageStateChangeListener.onSlideDirection(SLIDE_LEFT_TO_RIGHT);
                    } else if (isRightToLeft) {
                        mPageStateChangeListener.onSlideDirection(SLIDE_RIGHT_TO_LEFT);
                    }
                }
                isScrolling = false;
            }
        }
    };

    /**
     * 滑动状态改变回调
     */
    public interface PageStateChangeListener {
        void onPageScrolled(int position, float positionOffset, int positionOffsetPixels);
        void onPageSelected(int position);
        void onPageScrollStateChanged(int state);
        void onSlideDirection(int slideState);
    }

    /**
     * 设置监听
     */
    public void setPageChangeListener(PageStateChangeListener pageStateChangeListener){
        this.mPageStateChangeListener = pageStateChangeListener;
    }
}
```