# Android常用代码1
1		活动管理器																																															

	权限		<uses-permission android:name="android.permission.GET_TASKS"/>																																														
	代码		ActivityManager activityManager = (ActivityManager) getSystemService(Context.ACTIVITY_SERVICE);																																														

2		警报管理器																																															

	权限																																																
	代码		AlarmManager alarmManager = (AlarmManager) getSystemService(Context.ALARM_SERVICE);																																														

3		音频管理器																																															

	权限																																																
	代码		AudioManager audioManager = (AudioManager) getSystemService(Context.AUDIO_SERVICE);																																														

4		剪贴板管理器																																															

	权限																																																
	代码		ClipboardManager clipboardManager = (ClipboardManager) getSystemService(Context.CLIPBOARD_SERVICE);																																										;				

5		连接管理器																																															

	权限		<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>																																														
	代码		ConnectivityManager connectivityManager = (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);																																														

6		输入法管理器																																															
																																																	
	权限																																																
	代码		InputMethodManager inputMethodManager = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);																																														

7		键盘管理器																																															

	权限																																																
	代码		KeyguardManager keyguardManager = (KeyguardManager) getSystemService(Context.KEYGUARD_SERVICE);																																														

8		布局解压器管理器																																															

	权限																																																
	代码		LayoutInflater layoutInflater = (LayoutInflater) getSystemService(Context.LAYOUT_INFLATER_SERVICE);																																														

9		位置管理器																																															

	权限																																																
	代码		LocationManager locationManager = (LocationManager) getSystemService(Context.LOCATION_SERVICE);																																														

10		通知管理器																																															

	权限																																																
	代码		NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);																																														

11		电源管理器																																															

	权限		<uses-permission android:name="android.permission.DEVICE_POWER"/>																																														
	代码		PowerManager powerManager = (PowerManager) getSystemService(Context.POWER_SERVICE); 																																														

12		搜索管理器																																															

	权限																																																
	代码		SearchManager searchManager = (SearchManager) getSystemService(Context.SEARCH_SERVICE);																																														

13		传感器管理器																																															

	权限																																																
	代码		SensorManager sensorManager = (SensorManager) getSystemService(Context.SENSOR_SERVICE);																																														

14		电话管理器																																															

	权限		<uses-permission android:name="android.permission.READ_PHONE_STATE"/>																																														
	代码		TelephonyManager telephonyManager = (TelephonyManager) getSystemService(Context.TELEPHONY_SERVICE); 																																														

15		振动器																																															

	权限		<uses-permission android:name="android.permission.VIBRATE"/>																																														
	代码		Vibrator vibrator = (Vibrator) getSystemService(Context.VIBRATOR_SERVICE); 																																														

16		墙纸																																															

	权限		<uses-permission android:name="android.permission.SET_WALLPAPER"/>																																														
	代码		WallpaperService wallpaperService = (WallpaperService) getSystemService(Context.WALLPAPER_SERVICE);																																														

17		Wi-Fi管理器																																															

	权限		<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>																																														
	代码		WifiManager wifiManager = (WifiManager) getSystemService(Context.WIFI_SERVICE); 																																														

18		窗口管理器																																															

	权限																																																
	代码		WindowManager windowManager = (WindowManager) getSystemService(Context.WINDOW_SERVICE);
	

# Android系统的常用权限

http://img.blog.csdn.net/20160131122057559
http://img.blog.csdn.net/20160131122113731
http://img.blog.csdn.net/20160131122129746
