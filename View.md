# View

View是Android中所有控件的基类，不管是简单的TextView，Button还是复杂的LinearLayout和ListView，它们的共同基类都是View；
 View是一种界面层的控件的一种抽象，它代表了一个控件，除了View还有ViewGroup，从名字来看ViewGroup可以翻译为控件组，即一组View；
 在Android中，ViewGroup也继承了View，这就意味着View可以是单个控件，也可以是由多个控件组成的一组控件；

![img](https://img-blog.csdn.net/20160416151944699)

![img](https://img-blog.csdn.net/20160416152117873)

view提供的方法

getTop：获取到的，是view自身的顶边到其父布局顶边的距离 
getLeft：获取到的，是view自身的左边到其父布局左边的距离 
getRight：获取到的，是view自身的右边到其父布局左边的距离 
getBottom：获取到的，是view自身的底边到其父布局顶边的距离

MotionEvent提供的方法

getX()：获取点击事件相对控件左边的x轴坐标，即点击事件距离控件左边的距离 
getY()：获取点击事件相对控件顶边的y轴坐标，即点击事件距离控件顶边的距离 
getRawX()：获取点击事件相对整个屏幕左边的x轴坐标，即点击事件距离整个屏幕左边的距离 

getRawY()：获取点击事件相对整个屏幕顶边的y轴坐标，即点击事件距离整个屏幕顶边的距离

![img](https://img-blog.csdn.net/20160419001204719)

scrollTo是基于所传递参数的绝对移动，而scrollBy是基于当前位置的相对移动；就是To是我就移动到这个位置就不动了，By是基于我当前的位置继续偏移；

ScrollTo和ScrollBy滑动的是view的显示内容，并不改变view的坐标



## 获取View宽高

### View.post(new Runnable())

这里的view可以是你需要获取宽高的View。要注意的是view要执行此方法必须保证它已经attached到了window上，因此在此之前是不能调用这个方法的。

原理在于：View的宽高需要在Measure 过程后才能确定，直接在onCreate()等回调方法里获取只能得到0，因为此时还没有开始Measure操作。

而通过view.post()在主线程的消息队列尾部插入了一个消息，也就是说执行获取宽高的操作被延后了，并且能够保证Measure操作在此之前，所以就能够在这里获取到正确的宽高了。



网上还可以搜到其他类似方法如使用

ViewTreeObserver.addOnGlobalLayout()/addOnPreDrawLayout()或Activity/View.onWindowFocusChanged()方法中获取的，本质也是延后了操作，需要回调被调用后才能获取。



优点是保证获取到的宽高是准确的；

缺点是不能及时获取到，实际上还是把操作延后了，需要在Runnable里再执行相应回调。



### 通过LayoutParams获取

对于在XML文件里设置了具体宽高的View可以通过view.getLayoutParams().height/width获取到宽高。

总结：优点是能及时获取到，且操作简单

缺点是不够通用，没有设置具体宽高的获取到的值就是0了。



### 手动Measure再获取

自行调用view.measure()



总结：优点也是可以立即获取到宽高；缺点是无法解决match_parent的情况。