# Android 流式布局-自定义ViewGroup

需要注意的几个地方。

- 考虑场景 => onMeasure
- onMeasure spec值+mode
- Mode 是由哪些操作影响的
- view.gone 的处理
- 自定义属性
- 利用系统的属性完成自定义属性操作
- Layoutparams 相关4个方法
- padding的处理



测试demo

```java

public class FlowLayoutActivity extends AppCompatActivity {

    private FlowLayout mFlowLayout;
    private String[] texts = {"王大强", "adsasda", "ADSadsa", "Adsadsa"};

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_flow);
        mFlowLayout = findViewById(R.id.flow_Layout);
        findViewById(R.id.btn_add_tag).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                addTag();
            }
        });
    }

    private void addTag() {
        TextView textView = (TextView) LayoutInflater.from(this).inflate(R.layout.view_flow_tv, mFlowLayout,false);
        textView.setText(texts[(int) (Math.random() * 3)]);
        mFlowLayout.addView(textView);
    }
}

```



```java
public class FlowLayout extends ViewGroup {

    private List<List<View>> mAllView = new ArrayList<>();
    private List<Integer> mLineHeight = new ArrayList<>();

    private static final int[] ll = new int[]{android.R.attr.maxLines};
    private int mMaxLines;


    public FlowLayout(Context context) {
        super(context);
    }


    //适用于xml
    public FlowLayout(Context context, AttributeSet attrs) {
        super(context, attrs);

        //利用系统自定义属性
        TypedArray array = context.obtainStyledAttributes(attrs, ll);
        mMaxLines = array.getInt(0, Integer.MAX_VALUE);
        array.recycle();
        Log.e("demo", "maxLine=" + mMaxLines);
    }


    /**
     * flowlayout 宽度确定
     * 高度 wrapcontent,exactlu,unspe
     *
     * @param widthMeasureSpec
     * @param heightMeasureSpec
     */
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        mAllView.clear();
        mLineHeight.clear();

        super.onMeasure(widthMeasureSpec, heightMeasureSpec);

        //宽度无需更改，因为宽度可确定
        int sizeWidth = MeasureSpec.getSize(widthMeasureSpec);

        int sizeHeight = MeasureSpec.getSize(heightMeasureSpec);
        int modeHeight = MeasureSpec.getMode(heightMeasureSpec);

        //当前行的宽度
        int lineWidth = 0;
        //当前行的高度
        int lineHeight = 0;

        //最终的高度
        int height = 0;
        //拿到所有子View
        int cCount = getChildCount();

        //如果发现当前height 模式是 exactly,则无需测量
        if (modeHeight == MeasureSpec.EXACTLY) {
            setMeasuredDimension(sizeWidth, sizeHeight);
            return;
        }

        @SuppressLint("DrawAllocation") List<View> viewList = new ArrayList<>();


        //需要拿到当前child 需要占据的高度，用于设置给我们的容器
        for (int i = 0; i < cCount; i++) {
            View child = getChildAt(i);

            if (child.getVisibility() == GONE) {
                continue;
            }

            // child 也需要确定宽高
            measureChild(child, widthMeasureSpec, heightMeasureSpec);

            MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

            int cWidth = child.getMeasuredWidth() + lp.leftMargin + lp.rightMargin;
            int cHeight = child.getMeasuredHeight() + lp.topMargin + lp.bottomMargin;

            //如果大于当前宽度，则换行
            //否则找高度最大值，并储存  当前子view的高与宽
            if (lineWidth + cWidth > sizeWidth-(getPaddingLeft()+getPaddingRight())) {

                mLineHeight.add(lineHeight);

                height += lineHeight;
                //找最大值
                lineWidth = cWidth;
                lineHeight = cHeight;
                mAllView.add(viewList);
                viewList = new ArrayList<>();


            } else {
                //未换行
                lineWidth += cWidth;
                //设置最大的高度
                lineHeight = Math.max(lineHeight, cHeight);
            }
            viewList.add(child);

            if (i == cCount - 1) {
                height += lineHeight;
                mLineHeight.add(lineHeight);
                mAllView.add(viewList);
            }


            //MaxLine校准
            //如果大于设置的行数，这里强行控制其最终高度
            if (mMaxLines < mLineHeight.size()) {
                height = getMaxLineHeight();
            }

            if (modeHeight == MeasureSpec.AT_MOST) {
                height = Math.min(sizeHeight, height);
                height+=getPaddingBottom()+getPaddingBottom();
            }else if (modeHeight==MeasureSpec.UNSPECIFIED){
                height+=getPaddingBottom()+getPaddingBottom();
            }

            setMeasuredDimension(sizeWidth, height);
        }
    }

    public int getMaxLineHeight() {
        int height = 0;
        for (int i = 0; i < mMaxLines; i++) {
            height += mLineHeight.get(i);
        }
        return height;
    }


    /**
     * 负责view的摆放
     *
     * @param changed
     * @param l
     * @param t
     * @param r
     * @param b
     */
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        //摆放 view
        // lineHeight
        int left = getPaddingLeft();
        int top = getPaddingTop();

        int lineNums = mAllView.size();

        for (int i = 0; i < lineNums; i++) {
            //当前行View
            List<View> viewList = mAllView.get(i);

            //当前行高
            int lineHight = mLineHeight.get(i);

            //设置一行view
            for (int j = 0; j < viewList.size(); j++) {
                View child = viewList.get(j);

                MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

                int lc = left + lp.leftMargin;
                int tc = top + lp.topMargin;
                int rc = lc + child.getMeasuredWidth();
                int bc = tc + child.getMeasuredHeight();

                child.layout(lc, tc, rc, bc);

                left += child.getMeasuredWidth() + lp.leftMargin + lp.rightMargin;
            }

            left = getPaddingLeft();
            top += lineHight;
        }

    }

    /**
     * child 没有设置layoutParams，父控件默认生成
     *
     * @return
     */
    @Override
    protected LayoutParams generateDefaultLayoutParams() {
        return new MarginLayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);
    }

    /**
     * infalater 布局生成空间调用
     *
     * @param attrs
     * @return
     */
    @Override
    public LayoutParams generateLayoutParams(AttributeSet attrs) {
        return new MarginLayoutParams(getContext(), attrs);
    }

    /**
     * addView时 如果当前传递的LayoutParams不是我们需要的，需要手动转化
     *
     * @param p
     * @return
     */
    @Override
    protected LayoutParams generateLayoutParams(LayoutParams p) {
        return new MarginLayoutParams(p);
    }

    /**
     * addView时 如果当前传递的LayoutParams不是我们需要的，需要手动转化
     *
     * @param p
     * @return
     */
    @Override
    protected boolean checkLayoutParams(LayoutParams p) {
        return p instanceof MarginLayoutParams;
    }
}

```

