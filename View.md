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