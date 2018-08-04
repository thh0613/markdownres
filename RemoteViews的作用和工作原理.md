##RemoteViews的作用和工作原理

### 1. RemoteViews 是什么

查看`RemoteViews`的层级结构,`RemoteViews`没有继承`View`， 却实现了`parcelable`这个接口。

![extends.png](https://raw.githubusercontent.com/southmoney/markdownres/master/extends.png)

查看文档说明：

>A class that describes a view hierarchy that can be displayed in another process. The hierarchy is inflated from a layout resource file, and this class provides some basic operations for modifying the content of the inflated hierarchy.

翻译出来: 这是一个可以跨进程显示`view`的类，显示的`view`是从布局文件`inflate`出来，且该类提供了一些基本的方法来修改这个`view`的内容。

那么提炼一下就是：

- 跨进程使用

- 显示`view`

- 修改`view`的内容。

  接下来就以这三条线索来介绍`RemoteViews`的使用。

  

### 2. RemoteViews的使用

`RemoteViews`主要用在通知栏和桌面小部件上。

`RemoteViews`并不是一个`view`, 但可以表示一个`layout`的布局；又因为是继承`parcelable`,所以可以跨进程使用，但因为是跨进程，所以没办法像我们之前通过`findviewById`方法来访问布局里的每个`view`，所以`RemoteViews`提供了一些`set`方法来更新`view` 的显示；但`RemoteViews`可以支持的布局和控件是有限的。

***支持的布局：***

- `AdapterViewFlipper`
- `FrameLayout`
- `GridLayout`
- `GridView`
- `LinearLayout`
- `ListView`
- `RelativeLayout`
- `StackView`
- `ViewFlipper`

***支持的控件：***

- `AnalogClock`
- `Button`
- `Chronometer`
- `ImageButton`
- `ImageView`
- `ProgressBar`
- `TextClock`
- `TextView`

并且官网上说：

> Descendants of these classes are not supported.

即自定义的`view`就不支持了。

上面啰嗦了一堆，其实又可以提炼使用`RemoteViews`的关键字了

- 展示一个`layout`显示的`view`

- `set`方法更新内容

- 支持有限的布局和控件

  

  接下来利用以上三点，来尝试使用`RemoteViews`



####  2.1 通知栏的使用

我们可以调用`NotificationCompat.Builder.build()`来创建一个通知，然后调用`NotificationManager.notify()`显示通知栏，但有时候我们需要自定义通知栏的UI，这时候就需要`RemoteViews`来帮忙了

**第一步: 自定义个性化view**

**第二步：使用remoteviews**

注意：手机8.0以上要使用`notificationChannel` 来实现通知栏。

```java
 Intent intent = new Intent(this, NotiActivity.class);
        PendingIntent pendingIntent = PendingIntent.getActivity(this, 0,
                intent, PendingIntent.FLAG_UPDATE_CURRENT);

        String id = "my_channel_01";
        CharSequence name = "channel";
        String description = "description";
        int importance = NotificationManager.IMPORTANCE_DEFAULT;
        NotificationChannel mChannel = new NotificationChannel(id, name, importance);

        mChannel.setDescription(description);
        mChannel.enableLights(true);
        mChannel.setLightColor(Color.RED);
        mChannel.enableVibration(true);
        mChannel.setVibrationPattern(new long[]{100, 200, 300, 400, 500, 400, 300, 200, 400});

        NotificationManager manager = (NotificationManager) getSystemService(Context
                .NOTIFICATION_SERVICE);
        manager.createNotificationChannel(mChannel);


        RemoteViews remoteView = new RemoteViews(getPackageName(), R.layout.layout_notification);
        remoteView.setTextColor(R.id.re_text, Color.RED);
        remoteView.setTextViewText(R.id.re_text, "remote view demo");
        remoteView.setImageViewResource(R.id.re_image, R.drawable.btn_me_share);
        remoteView.setOnClickPendingIntent(R.id.notification, pendingIntent);

        Notification notification = new Notification.Builder(this, id)
                .setAutoCancel(false)
                .setContentTitle("title")
                .setContentText("describe")
                .setContentIntent(pendingIntent)
                .setSmallIcon(R.drawable.btn_me_share)
                .setOngoing(true)
                .setCustomContentView(remoteView)
                .setWhen(System.currentTimeMillis())
                .build();
        manager.notify(1, notification);
```

从上面代码发现，`RemoteViews`的方法使用起来很简单。利用构造函数`new RemoteViews(packagename, layoutId)` 来关联一个`view`的布局，并通过一些`set` 方法更新布局，最后利用`notification.Builder().setCustomContentView(RemoteViews)` 来设置通知栏的`view`



####2.2 桌面小部件的使用

桌面小部件主要是利用`RemoteViews`和`AppWidgetProvider`结合使用，而`AppWidgetProvider`又是` extends BroadcastReceiver`, 所以再使用的时候，多了一些关于广播的知识。

同样，我们也是提炼出关键步骤。



**第一步：自定义桌面小部件的布局 **

```xml
#layout_widget.xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:background="#09C"
                android:padding="@dimen/widget_margin">

    <TextView
        android:id="@+id/appwidget_text"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_centerHorizontal="true"
        android:layout_centerVertical="true"
        android:layout_margin="8dp"
        android:background="#09C"
        android:contentDescription="@string/appwidget_text"
        android:text="@string/appwidget_text"
        android:textColor="#ffffff"
        android:textSize="24sp"
        android:clickable="true"
        android:textStyle="bold|italic"/>

</RelativeLayout>
```



**第二步：配置**

注意：该配置需要在目录`res/xml`内。

```xml
#app_widget_info.xml
<?xml version="1.0" encoding="utf-8"?>
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
                    android:initialKeyguardLayout="@layout/layout_widget"
                    android:initialLayout="@layout/layout_widget"
                    android:minHeight="40dp"
                    android:minWidth="40dp"
                    android:previewImage="@drawable/example_appwidget_preview"
                    android:resizeMode="horizontal|vertical"
                    android:updatePeriodMillis="86400000"
                    android:widgetCategory="home_screen">
</appwidget-provider>
```



**第三步：使用RemoteViews**

```java
public class NewAppWidget extends AppWidgetProvider {
    private static final String CLICK_ACTION = "com.taohuahua.action.click";

    static void updateAppWidget(final Context context, final AppWidgetManager appWidgetManager, final int appWidgetId) {
        final RemoteViews views = new RemoteViews(context.getPackageName(), R.layout
                .layout_widget);

        Intent anIntent = new Intent();
        anIntent.setAction(CLICK_ACTION);
        PendingIntent anPendingIntent = PendingIntent.getBroadcast(context, 0, anIntent, 0);
        views.setOnClickPendingIntent(R.id.appwidget_text, anPendingIntent);
        appWidgetManager.updateAppWidget(appWidgetId, views);
    }

    @Override
    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        for (int appWidgetId : appWidgetIds) {
            updateAppWidget(context, appWidgetManager, appWidgetId);
        }
    }

    @Override
    public void onEnabled(Context context) {
        // Enter relevant functionality for when the first widget is created
    }

    @Override
    public void onDisabled(Context context) {
        // Enter relevant functionality for when the last widget is disabled
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        super.onReceive(context, intent);
        final RemoteViews views = new RemoteViews(context.getPackageName(), R.layout
                .layout_widget);

        if (Objects.equals(intent.getAction(), CLICK_ACTION)) {
            Toast.makeText(context, "hello world", Toast.LENGTH_SHORT).show();

            //获得appwidget管理实例，用于管理appwidget以便进行更新操作
            AppWidgetManager manger = AppWidgetManager.getInstance(context);
            // 相当于获得所有本程序创建的appwidget
            ComponentName thisName = new ComponentName(context, NewAppWidget.class);
            //更新widget
            manger.updateAppWidget(thisName, views);
        }

    }
}
```



从代码中可以看出，里面有几个重要的方法。

`onUpdate`: 小部件被添加时或者每次更新时调用。更新时间由第二步配置中`updatePeriodMills`来决定，单位为毫秒。

`onReceive`: 广播内置方法，用于分发接收到的事件。

`onEnable`: 当该窗口小部件第一次添加时调用。

`onDelete`:每删除一次调用一次。

`onDisabled`:最后一个该桌面小部件被删除时调用。

所以在`onUpdate`方法中利用`RemoteViews`来显示了新的布局，并利用`pendingIntent`来实现点击小部件控件跳转的方法。



**最后一步:在AndroidManifest.xml中声明广播**

```xml
<activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <receiver android:name=".NewAppWidget">
            <intent-filter>
                <action android:name="com.taohuahua.action.click" />
                <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
            </intent-filter>

            <meta-data
                android:name="android.appwidget.provider"
                android:resource="@xml/app_widget_info" />
        </receiver>
```

这样，通过四步就可以完成一个桌面小部件了。



###3 RemoteViews的原理

自定义通知栏和桌面小部件，他们是运行在系统进程中，即`SystemServer`进程, 而我们是要在自身的应用进程中来更新远程系统进程的UI。 最开始的一节我们知道`RemoteViews` 是实现了`Parcelable`接口的，这样就可以跨进程使用了。

在`RemoteViews`的源码中，可以看到定义了一个`Action`对象的列表

```java
    /**
     * An array of actions to perform on the view tree once it has been
     * inflated
     */
    private ArrayList<Action> mActions;
```

而`Action`的是对远程视图进行的操作的一个封装。因为我们无法通过`RemoteViews`的`findViewById`方法来操作视图，所以`RemoteViews`每次视图的操作都会创建一个`action`对象添加到列表中。

```java
 /**
     * Base class for all actions that can be performed on an
     * inflated view.
     *
     *  SUBCLASSES MUST BE IMMUTABLE SO CLONE WORKS!!!!!
     */
    private abstract static class Action implements Parcelable {
        public abstract void apply(View root, ViewGroup rootParent,
                OnClickHandler handler) throws ActionException;

        public static final int MERGE_REPLACE = 0;
        public static final int MERGE_APPEND = 1;
        public static final int MERGE_IGNORE = 2;

        public int describeContents() {
            return 0;
        }

        /**
         * Overridden by each class to report on it's own memory usage
         */
        public void updateMemoryUsageEstimate(MemoryUsageCounter counter) {
            // We currently only calculate Bitmap memory usage, so by default,
            // don't do anything here
        }

        public void setBitmapCache(BitmapCache bitmapCache) {
            // Do nothing
        }

        public int mergeBehavior() {
            return MERGE_REPLACE;
        }

        public abstract String getActionName();

        public String getUniqueKey() {
            return (getActionName() + viewId);
        }

        /**
         * This is called on the background thread. It should perform any non-ui computations
         * and return the final action which will run on the UI thread.
         * Override this if some of the tasks can be performed async.
         */
        public Action initActionAsync(ViewTree root, ViewGroup rootParent, OnClickHandler handler) {
            return this;
        }

        public boolean prefersAsyncApply() {
            return false;
        }

        /**
         * Overridden by subclasses which have (or inherit) an ApplicationInfo instance
         * as member variable
         */
        public boolean hasSameAppInfo(ApplicationInfo parentInfo) {
            return true;
        }

        int viewId;
    }
```



从源码中可以看出，`action`提供了一个抽象的方法

```java
  public abstract void apply(View root, ViewGroup rootParent,
                OnClickHandler handler) throws ActionException;
```



在`RemoteViews.java`中发现了有很多`Action`的子类, 这里重点讲解一个类

```java
 /**
     * Base class for the reflection actions.
     */
    private final class ReflectionAction extends Action {
    ...
    }
```



因为很多更新视图的方法最后都走到

```java
addAction(new ReflectionAction(viewId, methodName, type, value));
```

可以发现，当`RemoteViews`通过`set`方法来更新一个视图时，并没有立即更新，而是添加到`action`列表中。这样可以大大提高跨进程通信的性能，至于什么时候更新，对于自定义通知栏，需要NotificationManager调用notify()之后；而对于桌面小部件，则需要AppWidgetManager调用updateAppWidget()之后。



最后进入`ReflectionAction`类中的`apply`方法看一下，发现内部就是利用反射机制设置相关视图的。

```java

        @Override
        public void apply(View root, ViewGroup rootParent, OnClickHandler handler) {
            final View view = root.findViewById(viewId);
            if (view == null) return;

            Class<?> param = getParameterType();
            if (param == null) {
                throw new ActionException("bad type: " + this.type);
            }

            try {
                getMethod(view, this.methodName, param).invoke(view, wrapArg(this.value));
            } catch (ActionException e) {
                throw e;
            } catch (Exception ex) {
                throw new ActionException(ex);
            }
        }

```







