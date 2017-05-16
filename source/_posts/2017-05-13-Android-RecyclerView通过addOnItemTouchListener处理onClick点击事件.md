---
title: Android-RecyclerView通过addOnItemTouchListener处理onClick点击事件
date: 2017-05-13 14:30:54
tags: 
- Android
categories:
- Android
---

### 原因

现在android开发中ListView效果越来越多的用RecyclerView来实现了，RecyclerView相比前者有非常多的优点，还可以轻松实现瀑布流布局、扇形列表等。。。

****但是！！**** 



用的时候发现RecyclerView没有提供OnItemClick()，网上查的大多数博客都采用在Adapter里面自己定义接口的方法来模拟ListView的点击事件,有些还模拟了长按、短按等功能。

---

### 解决

这样模拟终究不是好方法，官方虽然没有提供click方法，但是提供了****更为强大****的addOnItemTouchListener接口，****结合GestureDetectorCompat手势检测****可以方便的实现很多功能。

#### 1. 为recycleView添加监听器

        recyclerView.addOnItemTouchListener(new RecyclerView.OnItemTouchListener() {
            @Override
            public boolean onInterceptTouchEvent(RecyclerView rv, MotionEvent e) {
                return false;
            }
            @Override
            public void onTouchEvent(RecyclerView rv, MotionEvent e) {

            }
            @Override
            public void onRequestDisallowInterceptTouchEvent(boolean disallowIntercept) {
            }
        });

其中：
* onInterceptTouchEvent：拦截触摸事件
* onTouchEvent：处理触摸事件
* onRequestDisallowInterceptTouchEvent：通常用于请求ViewPager不要拦截该控件上的触摸事件。

很明显，现在需要拦截触摸事件，拦截下来才能实现功能

#### 2. 既然只要处理onInterceptTouchEvent，那就可以删掉另外两个方法。

#### 3. 声明一个GestureDetector，传入GestureDetector.SimpleOnGestureListener实例化。

    SimpleOnGestureListener提供的onSingleTapUp和onLongPress，正是想要的功能！！


        GestureDetector gestureDetector;
        gestureDetector = new GestureDetector(this, new GestureDetector.SimpleOnGestureListener(){
            @Override
            public boolean onSingleTapUp(MotionEvent e){
                View childView = recyclerView.findChildViewUnder(e.getX(), e.getY());
                if (childView != null) {
                    int position = recyclerView.getChildLayoutPosition(childView);
                    Toast.makeText(getApplication(), "single click:" + position, Toast.LENGTH_SHORT).show();

                    return true;
                }
                return super.onSingleTapUp(e);
            }
            @Override
            public void onLongPress(MotionEvent e) {
                super.onLongPress(e);
                View childView = recyclerView.findChildViewUnder(e.getX(), e.getY());
                if (childView != null) {
                    int position = recyclerView.getChildLayoutPosition(childView);
                    Toast.makeText(getApplication(), "long click:" + position, Toast.LENGTH_SHORT).show();
                }
            }
        });

    这里不能直接声明为SimpleOnGestureListener，因为要用到父类的onTouchEvent


#### 4. 最后，在第1步的onInterceptTouchEvent里面加上一个简单的判断就可以了。

        @Override
        public boolean onInterceptTouchEvent(RecyclerView rv, MotionEvent e) {
            if (gestureDetector.onTouchEvent(e)) {
                return true;
            }
            return false;
        }

#### 5. 完成！


---

### 进阶：更完善的解决方法 - 封装成接口，方便重用

* 先说结果，封装成接口后，调用代码从上面一大堆变成短短几行，清晰明了。

        recyclerView.addOnItemTouchListener(new RecyclerItemClickListener(getBaseContext(), recyclerView,
            new RecyclerItemClickListener.OnItemClickListener() {
                @Override
                public void onItemClick(View view, int position) {
                    //轻触...
                }

                @Override
                public void onItemLongClick(View view, int position) {
                    //长按...
                }
            }));

        
    只需要写回调函数里的逻辑即可。


* 怎么封装类呢？

#### 1. 新建一个类 RecyclerItemClickListener，包含一个Item点击监听器和一个手势检测：

        public class RecyclerItemClickListener implements RecyclerView.OnItemTouchListener {
            private OnItemClickListener mListener;
            private GestureDetector gestureDetector;
        }


#### 2. 定义一个OnItemClickListener接口，里面包含两个回调函数
    
        public interface OnItemClickListener {
            void onItemClick(View view, int position);
            void onItemLongClick(View view, int position);
        }

#### 3. 构造函数

    构造函数需要接收Context、RecyclerView、OnItemClickListener三个参数.

    Context 上下文传递

    RecyclerView 接口实现的功能跟RecyclerView密切相关，可以通过这个参数获取position

    OnItemClickListener 可以理解为回调函数的通道


        public RecyclerItemClickListener(Context context, final RecyclerView recyclerView, OnItemClickListener listener) {
            mListener = listener;
            gestureDetector = new GestureDetector(context, new GestureDetector.SimpleOnGestureListener() {
                @Override
                public boolean onSingleTapUp(MotionEvent e) {
                    View child = recyclerView.findChildViewUnder(e.getX(), e.getY());
                    if (child != null && mListener != null) {
                        mListener.onItemClick(child, recyclerView.getChildAdapterPosition(child));
                    }
                    return true;
                }

                @Override
                public void onLongPress(MotionEvent e) {
                    View child = recyclerView.findChildViewUnder(e.getX(), e.getY());
                    if (child != null && mListener != null) {
                        mListener.onItemLongClick(child, recyclerView.getChildAdapterPosition(child));
                    }
                }
            });
        }


#### 4. 重写onInterceptTouchEvent()

        @Override
        public boolean onInterceptTouchEvent(RecyclerView view, MotionEvent e) {
            View childView = view.findChildViewUnder(e.getX(), e.getY());
            if (childView != null && mListener != null && gestureDetector.onTouchEvent(e)) {
                mListener.onItemClick(childView, view.getChildAdapterPosition(childView));
                return true;
            }
            return false;
        }

#### 5. 完成

---

>参考资料:  [android-RecyclerView onClick-Stack Overflow](http://stackoverflow.com/questions/24471109/recyclerview-onclick)