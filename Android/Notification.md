# Notification

## 基本的Notification

```java
private void basicNotification(){
        //创建一个Notification的Builder
        Notification.Builder builder = new Notification.Builder(this);
        //点击Notification后的延时操作
        Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse("http://www.baidu.com"));
        PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, 0);
        //设置各种属性
        builder.setSmallIcon(R.mipmap.ic_launcher);
        builder.setContentIntent(pendingIntent);
        builder.setAutoCancel(true);
        builder.setLargeIcon(BitmapFactory.decodeResource(this.getResources(),R.mipmap.ic_launcher_round));
        builder.setContentTitle("Basic Notification");
        builder.setContentText("This is a basic notification");
        builder.setSubText("it is really basic");
        //通过NotificationManager 来发出Notification
        NotificationManager notificationManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
        notificationManager.notify(NOTIFICATION_ID_BASIC,builder.build());

    }
```

## 折叠式Notification

一种自定义视图的Notification,常常用于显示长文本，拥有两个视图状态，一个是普通的状态，另一个是展开状态下的视图状态，在Notification中，使用RemoteViews 帮助我们创建一个自定义的Notification视图。

```java
 private void collapsedNotification(){

        Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse("http://www.baidu.com"));
        PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, 0);

        Notification.Builder builder = new Notification.Builder(this);
        builder.setSmallIcon(R.mipmap.ic_launcher);
        builder.setContentIntent(pendingIntent);
        builder.setAutoCancel(true);
        builder.setLargeIcon(BitmapFactory.decodeResource(this.getResources(),R.mipmap.ic_launcher_round));
        //通过RemoteViews 来创建自定义的Notification视图
        RemoteViews remoteViews = new RemoteViews(getPackageName(), R.layout.collapsed);
        remoteViews.setTextViewText(R.id.textView,"show me when collapsed");
        
        Notification notification = builder.build();
        //指定视图
        notification.contentView = remoteViews;
        //通过RemoteViews 来创建自定义的Notification视图
        RemoteViews expandedView = new RemoteViews(getPackageName(), R.layout.expanded);
        //指定视图
        notification.bigContentView = expandedView;

        NotificationManager systemService = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);

        systemService.notify(NOTIFICATION_ID_COLLAPSE,notification);

    }
```

## 悬挂式 Notification 

Android 5.0 之后新增加的方式，可在屏幕上方产生Notification且不会打断用户操作，能给用户以Notification形式的通知。

```java
 private void headsupNotification(){

        Notification.Builder builder = new Notification.Builder(this)
                .setSmallIcon(R.mipmap.ic_launcher_round)
                .setPriority(Notification.PRIORITY_DEFAULT)
                .setCategory(Notification.CATEGORY_MESSAGE)
                .setContentTitle("Headsup Notification")
                .setContentText("i am a headsup notification");

        Intent intent = new Intent();
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        intent.setClass(this,MainActivity.class);
        PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, PendingIntent.FLAG_CANCEL_CURRENT);
        builder.setContentText("Heads_Up Notification on Android 5.0")
                .setFullScreenIntent(pendingIntent,true);

        NotificationManager nm = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
        nm.notify(NOTIFICATION_ID_HEADSUP,builder.build());

    }
```

## 显示等级的 Notification

在Android 5.X 中加入了一种新模式：Notification 的显示等级。

- VISIBILITY_PRIVATE  :表明当前没有锁屏的时候会显示
- VISIBILITY_PUBLIC  : 表明在任何情况下都会显示
- VISIBILITY_SECRET : 表明在pin 、password 等安全锁和没有锁屏的情况下才能够显示

设置Notification 等级的方式：

```java
builder.setVisibility(Notification.VISIBILITY_PUBLIC);
```

