---
layout: post
title: "[技術筆記]Flutter 與 FCM "
subtitle: "使用firebase_message和local_notification"
description: "FCM（firebase cloud message），是用Firebase來控制發送、接收notification。不過之前我在寫專案的時候，發現網路上很多分享如何設定的文章不多、而且也蠻不一致的，所以就紀錄一下"
date: 2022-03-28T12:00:00
author: "李定宇"
image: "/img/flutter_post_bg.jpg"
tags:
  - Flutter
URL: "/2022/03/28/flutter_widget"
categories: [Flutter]
---

# 在 Flutter 中使用 FCM

## 前言

FCM（firebase cloud message），是用 Firebase 來控制發送、接收 notification。不過之前我在寫專案的時候，發現網路上很多分享如何設定的文章不多、而且也蠻不一致的（有可能是 firebase_message library 太常更新的緣故），所以現在來紀錄一下目前專案中是如何執行的。

_不過目前還算是不太知其所以然的狀態，先紀錄下來，後續說明會陸續補上_

## 專案版本依賴

- Flutter：2.5.3
- firebase_core: ^1.9.0
- firebase_messaging: ^10.0.9
- flutter_local_notifications: ^9.1.5

## 過程

### FIrebase 專案初始化

- 先在 Firebase 專案中下載 ios/android 所需要的 SDK（GoogleService-Info.plist/google-services.json）
  - 這邊要注意專案名稱的一致（如：com.expamle.demo）
  - 在 Ios 的專案，最好用 Xcode 來引入 GoogleService-Info.plist _（但 Ios 還需要在 App Developer 上設定一些東西，這裏就先略過）_
- 在 pubspec.yaml 下載所需的 library

###Android 設定 - 在 **{Flutter project}/android/app/build.gradle**中 - (之後在運行的時候可能會有些 android 版本報錯，我也有做一些修改)

```
//firebase setting
apply plugin: 'com.google.gms.google-services'

android {
    // modify 29 to 31 as setting flutter persission
    compileSdkVersion 31

    defaultConfig {
        //modify from 20 to 23 due to firebase services
        minSdkVersion 23
    }
    // ... other setting
}

dependencies {
// ... other dependencies
// 新增這一行
implementation platform('com.google.firebase:firebase-bom:29.0.0')
}
```

- 在 **{Flutter project}/android/build.gradle**中

```
buildscript {
  dependencies {
    // ... other dependencies
    // 新增這一行
     classpath 'com.google.gms:google-services:4.3.10'
	}
}
```

## Ios 設定

- 在**{Flutter Project}/ios/Runner/AppDelegate.swift**中

```
import UIKit
import Flutter

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    //firebase setting
    if #available(iOS 10.0, *) {
      UNUserNotificationCenter.current().delegate = self as UNUserNotificationCenterDelegate
    }
    //firebase setting end
    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```

## Flutter 程式碼設定

### 在 main.dart 中

```
Future _firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  await Firebase.initializeApp();
}

void main() async {

	///Firebase.initializeApp()需要調用 native 程式碼來初始化Firebase，
	///而且由於這個library需要使用 flutter 的 channel來調用 native 程式碼，那是異步完成的	//因此需要調用WidgetsFlutterBinding.ensureInitialized() 來確保有一個WidgetsBinding實例
	WidgetsFlutterBinding.ensureInitialized()
	await Firebase.initializeApp();

	///讓app在沒有被啟動時也可以讓手機接受notification
	FirebaseMessaging.onBackgroundMessage(_firebaseMessagingBackgroundHandler);

	/// library: flutter_local_notifications
	LocalNotificationService.requestIOSPermissions();

	runApp(MyApp())
}

class MyApp extends StatelessWidget {
	@override
	Widget build(BuildContext context) {
		 FirebaseNotificationService.initFirebaseNotificaiton(context);
	}
}
```

### 在 lib/ 新增檔案**firebase_notification_service.dart**

```

class FirebaseNotificationService {
  static initFirebaseNotificaiton(BuildContext context, String route) {
    ///初始化LocalNotificationService
    ///傳context是為了使用Navigator
    LocalNotificationService.initialize(context, route);

    ///give the message which user taps when app is from terminated state
    FirebaseMessaging.instance.getInitialMessage().then((message) {
      if (message != null) {
        // go to main screen
        // Navigator.of(context).pushNamed(route);
      }
    });

    var lock = Lock();

    ///stream 的監聽
    ///only work in the foreground
    FirebaseMessaging.onMessage.listen((message) async {
      if (!lock.locked) {
        await lock.synchronized(() {
          String preMsgId = context.read<TicketsQueueProvider>().preMsgId;
          String newMsgId = message.messageId;
          if (newMsgId != preMsgId) {
            LocalNotificationService.display(message);
          }
          Provider.of<TicketsQueueProvider>(context, listen: false)
              .setPreMsgId(newMsgId);
        });
      }
    });

    ///stream 監聽
    ///only when app in the background but not close, and user tap notifiction
    ///user tap notification
    FirebaseMessaging.onMessageOpenedApp.listen((message) {
      // do nothing, could go to previous screen(route)
    });
  }
}
```

- 會加上`synchronized` 這個鎖的原因，是因為在 IOS 實機測試中，發現 FirebaseMessaging.onMessage 會重複接收同一個 message.id 的 message。所以在專案上，用`Lock`來確保一次比較一個 message.id，然後用 Provider 來全局存取前一個 message.id

### 在 lib/ 新增檔案**local_notification_service.dart**

```

class LocalNotificationService {
  ///Singleton pattern
  ///目的：保證一個類別只會產生一個物件，而且要提供存取該物件的統一方法
  static final LocalNotificationService _notificationService =
      LocalNotificationService._internal();
  factory LocalNotificationService() {
    return _notificationService;
  }
  LocalNotificationService._internal();


  static void requestIOSPermissions() {
    notificationsPlugin
        .resolvePlatformSpecificImplementation<
            IOSFlutterLocalNotificationsPlugin>()
        ?.requestPermissions(
          alert: true,
          badge: true,
          sound: true,
        );
  }

  static final FlutterLocalNotificationsPlugin notificationsPlugin =
      FlutterLocalNotificationsPlugin();

  static void initialize(
      BuildContext context, String defaultRouteForMessage) async {
    final InitializationSettings initializationSettings = InitializationSettings(

        ///android default icon
        ///route should be: {project}\android\app\src\main\res\drawable\YOUR_APP_ICON.png
        android: AndroidInitializationSettings("@drawable/ic_notification"),

        ///ios default setting
        iOS: IOSInitializationSettings(
          requestSoundPermission: false,
          requestBadgePermission: false,
          requestAlertPermission: false,
        ));

    /// Create an Android Notification Channel.
    /// We use this channel in the `AndroidManifest.xml` file to override the
    /// default FCM channel to enable heads up notifications.
    AndroidNotificationChannel channel = const AndroidNotificationChannel(
      'high_importance_channel', // id
      'High Importance Notifications', // title
      importance: Importance.high,
    );
    await notificationsPlugin
        .resolvePlatformSpecificImplementation<
            AndroidFlutterLocalNotificationsPlugin>()
        ?.createNotificationChannel(channel);

    await notificationsPlugin.initialize(initializationSettings,
        onSelectNotification: (String route) async {
      print('---selected notification');
      // Navigator.of(context)
      //     .pushNamedAndRemoveUntil(defaultRouteForMessage, (route) => false);
      // Navigator.of(context).pushNamed(defaultRouteForMessage);
    });
  }

  static void testDisplay() async {
    try {
      final id = DateTime.now().millisecondsSinceEpoch ~/ 1000;

      ///notification 的一些設置，如channel、優先級等
      final notificationDetails = NotificationDetails(
        android: AndroidNotificationDetails(
          'high_importance_channel', //channel id
          'High Importance Notifications', //channel name
          importance: Importance.max,
          priority: Priority.high,
        ),
      );

      await notificationsPlugin.show(
        id,
        'notificaion title',
        'notificaion body',
        notificationDetails,
        payload: "",
      );
    } catch (e) {
      print(e);
    }
  }

  static void display(RemoteMessage message) async {
    try {
      ///確保每次都是獨立的id，用datetime
      final id = DateTime.now().millisecondsSinceEpoch ~/ 1000;

      AndroidNotificationDetails _androidNotificationDetails =
          AndroidNotificationDetails(
              'high_importance_channel', 'High Importance Notifications',
              playSound: true,
              priority: Priority.high,
              importance: Importance.max,);

      IOSNotificationDetails _iosNotificationDetails = IOSNotificationDetails(
//         presentAlert: bool?,  // Present an alert when the notification is displayed and the application is in the foreground (only from iOS 10 onwards)
//         presentBadge: bool?,  // Present the badge number when the notification is displayed and the application is in the foreground (only from iOS 10 onwards)
//         presentSound: bool?,  // Play a sound when the notification is displayed and the application is in the foreground (only from iOS 10 onwards)
//         sound: String?,  // Specifics the file path to play (only from iOS 10 onwards)
//         badgeNumber: int?, // The application's icon badge number
//         attachments: List<IOSNotificationAttachment>?, (only from iOS 10 onwards)
//         subtitle: String?, //Secondary description  (only from iOS 10 onwards)
//         threadIdentifier: String? (only from iOS 10 onwards)
          );
      NotificationDetails platformChannelSpecifics = NotificationDetails(
          android: _androidNotificationDetails, iOS: _iosNotificationDetails);

      await notificationsPlugin.show(
          id,
          message.notification.title,
          message.notification.body,
          // notificationDetails,
          platformChannelSpecifics,
          payload: ""
          ///傳route進去
          // payload: message.data["route"],
          );
    } catch (e) {
      print(e);
    }
  }
}
```

#Change Log

- 20220328- 初稿
