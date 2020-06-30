---
tags: [Android, Android Automotive OS, Android Phone App]
title: Android-Notification
created: '2020-06-30T15:50:13.057Z'
modified: '2020-06-30T15:54:48.821Z'
---

# Android Notification

## Overview

* A notification can appear as an icon in the status bar (a.k.a., *notification center*), in a floating window (*heads-up notification*), on the lock screen, and on an app icon.
* The UI of a notification baiscally consists of a small icon, app name, time stamp, and a larg icon. Optionally, it may include the title and detail information of the notification.
* A notification can provide some basic actions such as openning an appropriate app activity when tapped.
* Multiple notifications of a single app should be grouped together and providing a summary.
* **All notifications must be assigned to channels (a.k.a., categories) or it will not appear.**
* Android uses the *importance* of a notification to determine how much the notification should interrupt the user (visually and audibly). The higher the importance of a notification, the more interruptive the notification will be. The possible importance levels are: urgent, high, medium, low.

## Notifications on AAOS

* AAOS only allows a notification appear in notification center or as a heads-up notification.
* Notifications on AAOS have been simplified (in order to ensure the safty of driver and minimize distraction):
    * The design of noticaitons is simple: `BigPictureStyle`, `BigTextStyle`, and `InboxStyle` are not supported.
    * There is no complex controls such as tapping to expand a notification, long pressing a notification for additional options, or using controls based on swipe-length gestures.
    * Notifications only play a sound if they trigger a heads-up notification whose information is drive-critical, time-sensitive, and actionable.
    * Message notifications can be *played* or *muted*.
    * Although there is no notification channel, related nottifications are automatically grouped together.

## *Heads-up notifications* in AAOS:

Apps have different requirements for triggering a heads-up notification:

1. **For system-privileged apps and apps that are signed with the platform key**. The app can trigger a heads-up notification by setting the notification channel importance to `IMPORTANCE_HIGH` or higher.
2. **For all other apps**. To trigger a heads-up notification, the app sets the notification channel importance to `IMPORTANCE_HIGH` or higher and ensures that the notification belongs to one of the following categories: `CATEGORY_CALL`, `CATEGORY_MESSAGE`, or `CATEGORY_NAVIGATION`.

## Notification programming

### Create an notification

1. Obtain `NotificationManager` service
    ~~~kotlin
    val notifyManager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
    ~~~

2. Create a notification channel
    ~~~kotlin
    val PRIMARY_CHANNEL_ID = "primary_notification_channel"
    ...
    val notificationChannel = NotificationChannel(
          Companion.PRIMARY_CHANNEL_ID,
          "Mascot Notification", NotificationManager.IMPORTANCE_HIGH
      )

    notificationChannel.description = "Notification from Mascot"
    notifyManager.createNotificationChannel(notificationChannel)
    ~~~

    Note that, notification channel has not been supported in AAOS, so we can ignore the creation but still need `PRIMARY_CHANNEL_ID` variable.

3. Create and send a notification
    ~~~kotlin
    val NOTIFICATION_ID = 0
    ...
    val notification= NotificationCompat.Builder(this, PRIMARY_CHANNEL_ID)
        .setSmallIcon(R.drawable.ic_android)
        .setContentTitle("You've been notified!")
        .setContentText("This is your notification text.")
        .setAutoCancel(true)
        .setDefaults(NotificationCompat.DEFAULT_ALL)
        // A navigation heads-up
        .setCategory(NotificationCompat.CATEGORY_NAVIGATION)
        .setPriority(NotificationCompat.PRIORITY_HIGH)
        .build()

    val notificationManager = NotificationManagerCompat.from(this)
    notificationManager.notify(NOTIFICATION_ID, notification)
    ~~~

### Handle "clear" action in Notification Center
In order to know that user has dismissed our notification by swipping it away or clearing the whole Notification Center, we have to "listen" to "delete intent":
1. Register to recieve the "delete intent"
    ~~~kotlin
    registerReceiver(notificationReceiver, IntentFilter(ACTION_DELETE_NOTIFICATION))
    ~~~
2. Allow our notification broadcast the "delete intent" when it is cleared out.
    ~~~kotlin
    private fun addDeletePendingIntent(notifyBuilder: NotificationCompat.Builder) {
        val deleteIntent = Intent(ACTION_DELETE_NOTIFICATION)
        val deletePendingIntent = PendingIntent.getBroadcast(
            this,
            NOTIFICATION_ID,
            deleteIntent,
            PendingIntent.FLAG_ONE_SHOT
        )

        notifyBuilder.setDeleteIntent(deletePendingIntent)
    }
    ~~~

## References
* [Android notification overview](https://developer.android.com/guide/topics/ui/notifiers/notifications)
* [Notifications on Android Automotive OS](https://developer.android.com/training/cars/notifications)
* [Notification design and interaction patterns](https://material.io/design/platform-guidance/android-notifications.html#usage)
* [Create a group of notifications](https://developer.android.com/training/notify-user/group)
* [Google codelabs on notifications](https://codelabs.developers.google.com/codelabs/advanced-android-kotlin-training-notifications/#0)
