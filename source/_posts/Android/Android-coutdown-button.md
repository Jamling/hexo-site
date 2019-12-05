---
title: Android倒计时控件
date: 2016-08-02 21:01:00
category: [Android]
tags: [Android, QuickAF]
toc: true
description:
---

Android倒计时控件，通常在发送验证码时用得最多。其实Android系统提供了一个倒计时控件叫做CountDownTimer，不过这个控件没有与界面控件绑定，在这里，我要讲述的是如何自己写一个倒计时的控件。

## 源码
首先放上CountDownButton的源码

```java
/**
 * CountDownButton used for send phone verify code etc.
 *
 * @author Jamling
 */
public class CountDownButton extends RoundButton {
    private long totalTime = 60 * 1000;// 默认60秒
    private String label = "秒后重发";
    private long time;
    private long step = 1000;
    private int interval = 1000;

    private Handler mHandler = new Handler() {
        public void handleMessage(android.os.Message msg) {
            time -= step;
            if (time <= 0) {
                reset();
            }
            else {
                refreshText();
                mHandler.sendEmptyMessageDelayed(0, step);
            }
        }
    };
    
    public CountDownButton(Context context) {
        this(context, null);
    }
    
    public CountDownButton(Context context, AttributeSet attrs) {
        super(context, attrs);
        init(context, attrs);
    }

    private void init(Context context, AttributeSet attrs) {
        setHintTextColor(getTextColors());
    }
    
    public void start() {
        this.setEnabled(false);
        this.time = totalTime;
        refreshText();
        mHandler.sendEmptyMessageDelayed(0, step);
    }

    private void refreshText() {
        long t = (time / interval);
        if (t > 0) {
            this.setText(t + label);
        }
    }
    
    public void reset() {
        setText(null);
        setEnabled(true);
    }
    
    /**
     * Set count down total time
     * <p>
     * <code> setTotalTime(30000);// 30s</code>
     * </p>
     *
     * @param totalTime
     *
     * @return CountDownButton self
     */
    public CountDownButton setTotalTime(long totalTime) {
        this.totalTime = totalTime;
        return this;
    }

    /**
     * Set count down step
     * <p>
     * <code> setStep(1000);// 1s</code>
     * </p>
     *
     * @param step count down step, micro seconds
     *
     * @return CountDownButton self
     */
    public CountDownButton setStep(long step) {
        if (step > 0) {
            this.step = step;
        }
        return this;
    }

    /**
     * Set count down text refresh interval.
     * <p>
     * <code> setInterval(1000);//1s, text display: (getRemainingTime() / interval) + label</code>
     * </p>
     *
     * @param interval count down text refresh interval, micro seconds
     *
     * @return CountDownButton self
     */
    public CountDownButton setInterval(int interval) {
        if (interval > 0) {
            this.interval = interval;
        }
        return this;
    }

    public long getRemainingTime() {
        return time;
    }

    public long getStep() {
        return step;
    }
}
```

实现相对简单，通过Handler的sendEmptyMessageDelayed来定时发送消息，实现计时并更新界面，当然，也可以通过Timer来实现，不过因为Android UI线程的限制，不如使用Handler来得简洁。CountDownButton向外暴露设置step 步长，totalTime总计时等方法。

## 布局

``` xml
<cn.ieclipse.af.view.CountDownButton
    android:id="@+id/btn2"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="center"
    android:background="@color/black_333333"
    android:hint="点击发送验证码"
    android:padding="8dp"
    android:radius="8dp"
    android:textColor="@color/white"
    android:textSize="14sp"
    app:borderColor="@color/black_alpha_60"
    app:borderWidth="0dp"
    android:layout_marginTop="20dp" />
```

## Activity中使用

``` java
myBtn2 = (CountDownButton) btn2;
// btn2 text: enable: white; normal: gray
ColorStateList csl2 = new ColorStateList(new int[][]{{android.R.attr.state_enabled}, {}},
    new int[]{0xffffffff, 0xffcccccc});
myBtn2.setTextColor(csl2);
myBtn2.setTotalTime(10000);
// btn2 bg
int r = AppUtils.dp2px(this, 4);
RoundedColorDrawable normal = new RoundedColorDrawable(r, AppUtils.getColor(this, R.color.black_333333));
normal.addStateColor(new int[]{android.R.attr.state_enabled, android.R.attr.state_window_focused}, AppUtils
    .getColor(this, R.color.colorPrimary)).applyTo(myBtn2);
```

当CountDownButton开时计时时，它将进入disable状态（不可点击），button的外观也随之变化，当计时结束后，button恢复原来的状态。
示例代码中的button样式与状态，都是通过代码控制的。详细可以查看本站另一篇文章：[Android使用代码来实现StateListDrawable]

计时开始代码：

```java
    @Override
    public void onClick(View v) {
        if (v == myBtn2) {
            myBtn2.start();
            return;
        }
        super.onClick(v);
    }
```


## 关于

[QuickAF] 是一个Android平台上的app快速开发框架，欢迎读者在github star或fork。本人写作水平有限，欢迎广大读者指正，如有问题，可与我直接联系或在我的官方博客中给出评论。


## 参考
Android使用代码来实现StateListDrawable: http://www.ieclipse.cn/2016/08/02/Android/Android-code-states/
QuickAF: https://github.com/Jamling/QuickAF

[Android使用代码来实现StateListDrawable]: http://www.ieclipse.cn/2016/08/02/Android/Android-code-states/
[QuickAF]: https://github.com/Jamling/QuickAF

