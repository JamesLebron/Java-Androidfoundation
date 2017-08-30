#View 相关知识总结(Android开发艺术探索笔记)

	1.View和ViewGroup之间的关系
		ViewGroup继承View,ViewGroup可以包含多个View
	2.View的位置参数
![image](http://img.blog.csdn.net/20150115155321445?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamFzb24wNTM5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

	3.MotionEvent和TouchSlop
	
		3.1 MotionEvent
		action_down
		action_move
		action_up
	
		3.2 TouchSlop 系统默认所能识别的最小滑动距离
		
		
		4.VelocityTraker 追踪手指滑动的速度,使用方法如下
```java
	 private void init(Context context) {
        this.mContext = context;
        mVelocityTracker = VelocityTracker.obtain();
    }

    public void setCallBack(callBack callBack) {
        mCallBack = callBack;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int action = event.getAction();
        mVelocityTracker.addMovement(event);
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                mPointerId = event.getPointerId(0);
                break;
            case MotionEvent.ACTION_MOVE:
                mVelocityTracker.computeCurrentVelocity(1000);
                mXVelocity = mVelocityTracker.getXVelocity(mPointerId);
                mYVelocity = mVelocityTracker.getYVelocity(mPointerId);
                if (mCallBack!=null){
                    mCallBack.setText(mXVelocity,mYVelocity);
                }
                break;
            case MotionEvent.ACTION_UP:

                break;
        }
        //这里return true 代表本View 消费了事件  才会继续执行 action move 和action down
        return true;
    }
    //不使用的时候,回收
 @Override
    protected void onDetachedFromWindow() {
        mVelocityTracker.clear();
        mVelocityTracker.recycle();
        super.onDetachedFromWindow();
    }
	
```

	5.GestureDetector的使用
```java

    private void init(Context context) {
        this.mContext = context;
        mGestureDetector = new GestureDetector(context, new GestureDetector.OnGestureListener() {
            @Override
            //手指轻触屏幕触发
            public boolean onDown(MotionEvent e) {
                return true;
            }

            //手指轻触屏幕,尚未松开或拖动
            @Override
            public void onShowPress(MotionEvent e) {
            }

            //单击
            @Override
            public boolean onSingleTapUp(MotionEvent e) {
                return true;
            }

            //手指按下并拖动
            @Override
            public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) {
                Toast.makeText(mContext, "这是一次拖动", Toast.LENGTH_SHORT).show();
                return true;
            }

            //用户长按屏幕不放
            @Override
            public void onLongPress(MotionEvent e) {
                Toast.makeText(mContext, "这是一次长按", Toast.LENGTH_SHORT).show();
            }

            //快速滑动
            @Override
            public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
                Toast.makeText(mContext, "这是一次快速滑动", Toast.LENGTH_SHORT).show();
                return true;
            }
        });
        mGestureDetector.setIsLongpressEnabled(true);
        //双击
        mGestureDetector.setOnDoubleTapListener(new GestureDetector.OnDoubleTapListener() {
            /**
             * onSingleTapUp()和onSingleTapConfirmed()的区别
             onSingleTapUp() - 在按下并抬起时发生，只要符合这个条件就触发该函数，没有任何附加条件。
             onSingleTapConfirmed() 同上者，但有附加条件，就是Android会确保单击之后短时间内没有再次单击，才会触发该函数。
             举个列子，如果监听双击事件：onSingleTapUp()会被触发两次。但是onSingleTapConfirmed()一次都不会被触发。
             所以，如果你既想监听单击事件，又想监听双击时间，那么请使用onSingleTapConfirmed()函数。
             * @param e
             * @return
             */
            @Override
            public boolean onSingleTapConfirmed(MotionEvent e) {
                Toast.makeText(mContext, "这是一次单击", Toast.LENGTH_SHORT).show();
                return true;
            }

            @Override
            public boolean onDoubleTap(MotionEvent e) {
                Toast.makeText(mContext, "这是一次双击", Toast.LENGTH_SHORT).show();
                return true;
            }

            @Override
            public boolean onDoubleTapEvent(MotionEvent e) {
                return true;
            }
        });
    }


    @Override
    public boolean onTouchEvent(MotionEvent event) {
        return mGestureDetector.onTouchEvent(event);
    }

```
	
	6.View的滑动
	
		6.1 使用ScrollTo/ScrollBy
			两者的区别:
			ScrollTo是滑动到某个位置
			ScrollBy内部调用的ScrollTo,是基于上一次的ScrollX+this distance增量滑动的
	
		6.2 使用动画
			ObjectAnimator.ofFloat(targetView,"translationX",0,100).setDuration(100).start();
	
		6.3改变布局参数
			改变View的layoutParams
	
		6.4各种滑动的对比
			ScrollTo/ScrollBy:操作简单,适用于View内容的滑动
			动画:操作简单,适用于没有交互的View和实现复杂的动画效果
			改变布局参数:操作复杂,适用于有交互的View
	
	7.弹性滚动的几种方式
		使用Scroller
		使用动画
		使用延迟策略(使用handler.postDelayed(doScroll))
	
	8.Scroller的个人简单解析
	 	Scroller本身并不能让View 进行滑动,需要配合View 的computeScroll才能进行滑动
		1.当调用Scroller的startScroll
		2.紧接着调用invalidate让View重绘,view的draw 方法会调用View 的computeScroll方法
		3.computeScroll方法会进行Scroller.computeScrollOffset 判断滑动是否完成(Scroller 内部调用computeScrollOffset方法计算根据传入的需要滑动的时间,计算出当前需要滑动多少距离,类似属性动画的插值器(此方法会返回一个boolean值,如果为true,说明滑动还未完成))
		4.如果滑动未完成,根据计算出的值调用ScrollTo进行小距离滑动,继续调用postInvalidate,又调用View的draw方法....
		这样每次滑动一点,这样就连续组成了弹性滑动