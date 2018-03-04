RippleEffect源码分析
================

fork了一位国外友人的水波纹效果兼容方案，并修改成效果与原生一致（可能一致吧，至少看起来是差不多的 ：），这是它的项目page：
[RippleEffect](https://github.com/traex/RippleEffect)  

### Usage  
由于是一个自己fork下来的项目，只能导入里面的library来使用,控件使用方法和原项目的一致
### code analyse
RippleView这个控件的实现原理：在需要用到水波纹效果的控件A外加上一层自定义ViewGroup，通过拦截控件A的点击事件，
实现水波纹的动画效果并把点击事件传递到控件A上。
这里涉及到的一个比较难的技术点就是事件分发，水波纹控件RippleView通过onInterceptTouchEvent方法拦截了点击事件，
在onTouchEvent方法里实现动画效果并触发子控件的点击事件。以下是控件的相关代码：
```
  @Override
    public boolean onTouchEvent(MotionEvent event) {
        if (gestureDetector.onTouchEvent(event)) {
            animateRipple(event);
            sendClickEvent(false);
        }
        return super.onTouchEvent(event);
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent event) {
        this.onTouchEvent(event);
        return super.onInterceptTouchEvent(event);
    }
```
控件使用了GestureDetector类来简化长按事件的监听，GenstureDetector是一个系统的手势检测工具类
通过实现接口功能就能监听长按等事件，该工具类的控件代码如下：
```
 gestureDetector = new GestureDetector(context, new GestureDetector.SimpleOnGestureListener() {
            @Override
            public void onLongPress(MotionEvent event) {
                super.onLongPress(event);
                animateRipple(event);
                sendClickEvent(true);
            }

            @Override
            public boolean onSingleTapConfirmed(MotionEvent e) {
                return true;
            }

            @Override
            public boolean onSingleTapUp(MotionEvent e) {
                return true;
            }
        });
```
在LongPress里面就触发动画效果和控件的点击事件,onSingleTapConfirmed和onSingleTapUp方法是响应普通点击事件，
返回为true是把普通点击事件返回给RippleView作响应。

### modify function
和原生的效果做对比后发现：
1.rippleView的长按点击没有缓慢水波纹扩散的效果；
针对这个情况，我发现是代码里没有对长按事件做一个有效的区分和处理，导致不能实现与原生一致的效果
所以对GestureDetector进行了修改：
```
 gestureDetector = new GestureDetector(context, new GestureDetector.SimpleOnGestureListener() {
            @Override
            public void onLongPress(MotionEvent event) {
                super.onLongPress(event);
                setRippleDuration(longRippleDuration);
                mIsLongPress = true;
                animateRipple(event);
            }

//            @Override
//            public boolean onSingleTapConfirmed(MotionEvent e) {
//                mIsLongPress = false;
//                animateRipple(e);
//                return true;
//            }

            @Override
            public boolean onSingleTapUp(MotionEvent e) {
                mIsLongPress = false;
                animateRipple(e);
                return true;
            }
        }){
            @TargetApi(Build.VERSION_CODES.JELLY_BEAN)
            @Override
            public boolean onTouchEvent(MotionEvent ev) {
                mActionState = ev.getAction();
                if(ev.getAction() == MotionEvent.ACTION_UP){
                    setRippleDuration(100);
                    invalidate();
                } else if(ev.getAction() == MotionEvent.ACTION_CANCEL){
                    setRippleDuration(100);
                }
                return super.onTouchEvent(ev);
            }
        };
```
重写了GestureDetector的OnTouchEvent方法，用于区分事件类型为ACTION_UP时的操作。
在动画效果进行的过程中，通过状态变量mIsLongPress来判断是应否做长按的效果。


