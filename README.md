 I. Overview of the method
In the api 18 before the auxiliary function 'AccessibilityEvent.TYPE_NOTIFICATION_STATE_CHANGED' or reflection can take the information related to the notification bar. Now we can easily retrieve callback related information based on the NotificationListenerService class.

 Two. NotificationListenerService explain
The addition and deletion of notifications will call back the methods in your registered NLS, which can be obtained from the StatusBarNotification class object.

 NotificationListenerService main method (member variables):
Callback onNotificationRemoved delete notification, new notification or update callback onNotificationPosted

cancelAllNotifications (): delete all the system can be cleared notification;
cancelNotification (String pkg, String tag, int id): delete a specific notification;

getActiveNotifications (): Returns a list of all notifications for the current system to StatusBarNotification []

onNotificationPosted (StatusBarNotification sbn): The starting callback when the system receives a new notification;

onNotificationRemoved (StatusBarNotification sbn): The starting callback when the system notification is deleted;
 StatusBarNotification detailed category
StatusBarNotification, multi-process delivery object, all the notification information will be passed in this class through the Binder. Several important internal methods are as follows:

getId (): returns the id of the notification;
getNotification (): returns the notification object;
getPackageName (): returns the package name corresponding to the notification;
getPostTime (): returns the time the notification was initiated;
getTag (): return the notification Tag, if not set to return null;
isClearable (): return whether the notice can be clear, FLAG_ONGOING_EVENT, FLAG_NO_CLEAR;
isOngoing (): Check whether the notification flag is FLAG_ONGOING_EVENT;
Among them, we can get Notification object through getNotification (), Notification is a class we are more familiar with, we can be notified of the specific content can even restore RemoteViews to our local view.

 Third, the use of methods
The proper use of NotificationListenerService requires three steps:

 1. Create a new class and inherit from NotificationListenerService, override two of the important methods;
  public class NLService extends NotificationListenerService {
         @Override
         public void onNotificationPosted ( StatusBarNotification sbn ) {
         }
 
         @Override
         public void onNotificationRemoved ( StatusBarNotification sbn ) {
         }
     }
 Register the Service in AndroidManifest.xml and declare the related permissions;
  < service android : name = " .NLService "
           android : label = " @ string / service_name "
           android : permission = " android.permission.BIND_NOTIFICATION_LISTENER_SERVICE " >
      < intent-filter >
          < action android : name = " android.service.notification.NotificationListenerService " />
      </ intent-filter >
    </ service >
 3. Enable the monitoring function of NotificationMonitor
  if ( ! isEnabled ()) {
                 Intent intent = new Intent ( " android.settings.ACTION_NOTIFICATION_LISTENER_SETTINGS " );
                 startActivity (intent);
             } else {
                 Toast . MakeText ( this , " Service enabled " , Toast . LENGTH_LONG ) . Show ();
             }
            
  private boolean isEnabled () {
         String pkgName = getPackageName ();
         final String flat = Settings . Secure . getString (getContentResolver (),
                 ENABLED_NOTIFICATION_LISTENERS );
         if ( ! TextUtils . isEmpty (flat)) {
             final String [] names = flat . split ( " : " );
             for ( int i = 0 ; i < names . length; i ++ ) {
                 final ComponentName cn = ComponentName . unflattenFromString (names [i]);
                 if (cn ! = null ) {
                     if ( TextUtils . equals (pkgName, cn . getPackageName ())) {
                         return true ;
                     }
                 }
             }
         }
         return false ;
     }
    
 Four. Demo to explain
According to the above steps, it is possible to receive the callback when the notification bar is changed and obtain all the current notification lists. We limit to write a small example, take all the notification lists and monitor the notification bar changes, and display the received Notification and related information Go to our page ListView.

 1. Follow the steps above to create a basic example framework
Write layout, there must be open services btn, btn all the notifications and btn clear all the list, the interface is relatively simple, as shown:

![Demo截图](./screenshot/demo.png)

2. Establish BroadcastReceiver and Service interaction
Of course, you can also use binder that message communication, according to their own programming options, here for a simple demonstration with the BroadcastReceiver mechanism

  class NotificationReceiver extends BroadcastReceiver {
         @Override
         public void onReceive ( Context context , Intent intent ) {
             String temp = mInfoList . Size () + " : " + intent . GetStringExtra ( EVENT );
             NTBean bean = new NTBean ();
             bean . info = temp;

             Bundle budle = intent . GetExtras ();
             bean . title = budle . getString ( Notification . EXTRA_TITLE );
             bean . text = budle . getString ( Notification . EXTRA_TEXT );
             bean . subText = budle . getString ( Notification . EXTRA_SUB_TEXT );
             bean . largeIcon = budle . getParcelable ( Notification . EXTRA_LARGE_ICON );
             Icon icon = budle . GetParcelable ( Notification . EXTRA_SMALL_ICON );
             bean . smallIcon = icon;

             bean . viewS = budle . getParcelable ( VIEW_S );
             bean . viewL = budle . getParcelable ( View_L );

             mInfoList . add (bean);
             Log . E ( " changxing " , " receive: " + temp + " \ n " + budle);
             mAdapter . notifyDataSetChanged ();
         }

     }
    
 3. Live callback display page
The callback parameters passed to the activity, displayed in the listview, which can be used directly to RemoteViews # apply Notification displayed to our local ViewGroup .

The program runs as follows:

![程序运行截图](./screenshot/scr_a.png)
![程序运行截图](./screenshot/scr_b.png)
