## Window

Window是一个抽象类，它概括了Android窗口的基本属性和基本功能。唯一实现了这个抽象类的是PhoneWindow，实例化PhoneWindow需要一个窗口，只需要通过WindowManager即可完成，Window类的具体实现位于WindowManagerService中，WindowManager和WindowManagerService的交互是一个IPC过程。Android中的所有视图都是通过Window来呈现的，不管是Activity，Dialog还是Toast，他们的视图实际上都是附加在Window上的，因此Window实际上是View的直接管理者，点击事件也是由Window传递给view的。

创建完Window之后，Activity会为该Window设置回调，Window接收到外界状态改变时就会回调到Activity中。在activity中会调用setContentView()函数，它是调用window.setContentView()完成的，而Window的具体实现是PhoneWindow，所以最终的具体操作是在PhoneWindow中，PhoneWindow的setContentView方法第一步会检测DecorView是否存在，如果不存在，就会调用generateDecor函数直接创建一个DecorView；第二步就是将Activity的视图添加到DecorView的mContentParent中；第三步是回调Activity中的onContentChanged方法通知Activity视图已经发生改变。这些步骤完成之后，DecorView还没有被WindowManager正式添加到Window中，最后调用Activity的onResume方法中的makeVisible方法才能真正地完成添加和现实过程，Activity的视图才能被用户看到。

Window的实现类是PhoneWindow

Window. FEATURE_NO_TITLE

![img](file:///C:/Users/ALLENI~1/AppData/Local/Temp/msohtmlclip1/01/clip_image001.png) ![img](file:///C:/Users/ALLENI~1/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

 

![img](file:///C:/Users/ALLENI~1/AppData/Local/Temp/msohtmlclip1/01/clip_image004.jpg)

activity.getWindow().getDecorView();

## WindowManager

WindowManager主要用来管理窗口的一些状态、属性、view增加、删除、更新、窗口顺序、消息收集和处理等。通过代码Context.getSystemService(Context.WINDOW_SERVICE)可以获得WindowManager的实例。WindowManager所提供的功能很简单，常用的只有三个方法，即添加View、更新View和删除View，这三个方法定义在ViewManager中，而WindowManager继承了ViewManager

 

```
WindowManager manager = (WindowManager) getSystemService(Context.WINDOW_SERVICE);

WindowManager.LayoutParams params = new WindowManager.LayoutParams();

params.format = PixelFormat.TRANSLUCENT;//背景透明

manager.addView(imageView,params);

manager.removeView(imageView);
```

 

| Flag                  |      |
| --------------------- | ---- |
| FLAG_NOT_TOUCH_MODAL  |      |
| FLAG_NOT_FOCUSABLE    |      |
| FLAG_SHOW_WHEN_LOCKED |      |
| Type                  |      |
| TYPE_SYSTEM_OVERLAY   |      |
| TYPE_SYSTEM_ERROR     |      |

