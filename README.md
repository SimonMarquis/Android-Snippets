Android Snippets
================


  * [ADB](#adb-android-debug-bridge)
    * [list of devices](#list-of-devices)
    * [screen capture](#screen-capture)
    * [screen record](#screen-record)
    * [run monkey](#run-monkey)
    * [sqlite](#sqlite)
    * [handy commands](#handy-commands)
  * [Animation](#animation)
    * [flash](#flash)
    * [nope](#nope)
    * [pulse](#pulse)
    * [spring](#spring)
    * [tada](#tada)
  * [Application](#application)
    * [application name](#applicatio-nname)
    * [version name](#version-name)
    * [version code](#version-code)
    * [kill background processes](#kill-background-processes)
    * [restart](#restart)
  * [Build](#build)
    * [OS version](#os-version)
    * [has Cupcake](#has-cupcake)
    * [has Donut](#has-donut)
    * [has Eclair](#has-eclair)
    * [has Froyo](#has-froyo)
    * [has Gingerbread](#has-gingerbread)
    * [has Honeycomb](#has-honeycomb)
    * [has Ice Cream Sandwich](#has-ice-cream-sandwich)
    * [has Jelly Bean](#has-jelly-bean)
    * [has KitKat](#has-kitkat)


ADB (Android Debug Bridge)
--------------------------

### list of devices

```bash
#!/bin/bash
if [[ ! $PATH_TO_ADB ]]; then
	PATH_TO_ADB=`which adb`
fi

if [[ ! $PATH_TO_ADB ]]; then
	if [[ ! $ANDROID_HOME ]]; then
		echo "Failed to determine path to adb; consider setting ANDROID_HOME to your SDK directory or PATH_TO_ADB to the path to ADB"
		exit 1
	fi
	PATH_TO_ADB="$ANDROID_HOME/platform-tools/adb"
fi

devices=`$PATH_TO_ADB devices | grep -E "device\$" | cut -f1`

for device in $devices; do
	model=$($PATH_TO_ADB -s $device shell getprop ro.product.model | tr -d '\r')
	version=$($PATH_TO_ADB -s $device shell getprop ro.build.version.release | tr -d '\r')
	sdk=$($PATH_TO_ADB -s $device shell getprop ro.build.version.sdk | tr -d '\r')

	printf '%-20s [%5s ~ %2s]: %-20s \n' "$device" "$version" "$sdk" "$model"
done
```

### screen capture

```bash
name="/sdcard/`date +%Y.%m.%d\ -\ %H.%M.%S`.png" && \
	adb shell screencap -p $name && \
	adb pull $name && \
	adb shell rm $name
```

### screen record

 * **bit rate**  `--bit-rate <rate>`
 * **size**  `--size <widthXheight>`
 * **time limit**  `--time-limit <time>`

```bash
name="/sdcard/`date +%Y.%m.%d\ -\ %H.%M.%S`.mp4" && \
	adb shell screenrecord --bit-rate 20000000 --time-limit 30 $name && \
	adb pull $name && \
	adb shell rm $name
```

### run monkey

 * **verbose**  `-v`
 * **package name**  `-p <your.package.name>`
 * **throttling**  `--throttle <milliseconds>`

```bash
adb shell monkey -p your.package.name -v 500
```

### sqlite

```bash
adb shell
$ sqlite3 /data/data/your.package.name/databases/your-database.db
	SQLite version 3.3.12
	Enter ".help" for instructions
	.... enter commands, then quit...
sqlite> .exit 
```

### handy commands 

```bash
# install apk
adb install -r app.apk

# uninstall apk
adb uninstall your.package.name

# start activity
adb shell am start -n your.package.name/.YourActivity

# start scheme intent
adb shell am start -W -a android.intent.action.VIEW -d "yourscheme://something" your.package.name

# installed packages
adb shell pm list packages -f

# enable CheckJNI (0 to disable)
adb shell setprop debug.checkjni 1
```


Animation
---------

### flash

```java
public static ObjectAnimator flash(View view, float alphaFactor) {
	PropertyValuesHolder pvhalpha = PropertyValuesHolder.ofKeyframe(android.view.View.ALPHA, Keyframe.ofFloat(0f, 1f), Keyframe.ofFloat(.25f, alphaFactor), Keyframe.ofFloat(0.5f, 1f), Keyframe.ofFloat(.75f, alphaFactor), Keyframe.ofFloat(1f, 1f));
	return ObjectAnimator.ofPropertyValuesHolder(view, pvhalpha).setDuration(800);
}
```

### nope

Introduced by @cyrilmottier on [Google+](https://plus.google.com/+CyrilMottier/posts/FABaJhRMCuy)

```java
public static ObjectAnimator nope(View view, int delta) {
    PropertyValuesHolder pvhTranslateX = PropertyValuesHolder.ofKeyframe(View.TRANSLATION_X, Keyframe.ofFloat(0f, 0),
            Keyframe.ofFloat(.10f, -delta), Keyframe.ofFloat(.26f, delta), Keyframe.ofFloat(.42f, -delta), Keyframe.ofFloat(.58f, delta),
            Keyframe.ofFloat(.74f, -delta), Keyframe.ofFloat(.90f, delta), Keyframe.ofFloat(1f, 0f));
    return ObjectAnimator.ofPropertyValuesHolder(view, pvhTranslateX).setDuration(500);
}
```

### pulse

```java
public static ObjectAnimator pulse(View view, float pulseFactor) {
	PropertyValuesHolder pvhScaleX = PropertyValuesHolder.ofKeyframe(android.view.View.SCALE_X, Keyframe.ofFloat(0f, 1f), Keyframe.ofFloat(.5f, pulseFactor * 1f), Keyframe.ofFloat(1f, 1f));
	PropertyValuesHolder pvhScaleY = PropertyValuesHolder.ofKeyframe(android.view.View.SCALE_Y, Keyframe.ofFloat(0f, 1f), Keyframe.ofFloat(.5f, pulseFactor * 1f), Keyframe.ofFloat(1f, 1f));
	return ObjectAnimator.ofPropertyValuesHolder(view, pvhScaleX, pvhScaleY).setDuration(500);
}
```

### spring

```java
public static ObjectAnimator spring(View view, float springFactor) {
	PropertyValuesHolder pvhScaleX = PropertyValuesHolder.ofKeyframe(android.view.View.SCALE_X, Keyframe.ofFloat(0f, 1f),
			Keyframe.ofFloat(0.25f, springFactor * 1.35f), Keyframe.ofFloat(0.5f, 0.65f / springFactor),
			Keyframe.ofFloat(0.75f, springFactor * 1.15f), Keyframe.ofFloat(1f, 1f));
	PropertyValuesHolder pvhScaleY = PropertyValuesHolder.ofKeyframe(android.view.View.SCALE_Y, Keyframe.ofFloat(0f, 1f),
			Keyframe.ofFloat(0.25f, 0.65f / springFactor), Keyframe.ofFloat(0.5f, springFactor * 1.35f),
			Keyframe.ofFloat(0.75f, 0.85f / springFactor), Keyframe.ofFloat(1f, 1f));
	return ObjectAnimator.ofPropertyValuesHolder(view, pvhScaleX, pvhScaleY).setDuration(600);
}
```

### tada

Introduced by @cyrilmottier on [Google+](https://plus.google.com/+CyrilMottier/posts/FABaJhRMCuy)

```java
public static ObjectAnimator tada(View view, float shakeFactor) {
    PropertyValuesHolder pvhScaleX = PropertyValuesHolder.ofKeyframe(View.SCALE_X, Keyframe.ofFloat(0f, 1f),
            Keyframe.ofFloat(.1f, .9f), Keyframe.ofFloat(.2f, .9f), Keyframe.ofFloat(.3f, 1.1f), Keyframe.ofFloat(.4f, 1.1f),
            Keyframe.ofFloat(.5f, 1.1f), Keyframe.ofFloat(.6f, 1.1f), Keyframe.ofFloat(.7f, 1.1f), Keyframe.ofFloat(.8f, 1.1f),
            Keyframe.ofFloat(.9f, 1.1f), Keyframe.ofFloat(1f, 1f));

    PropertyValuesHolder pvhScaleY = PropertyValuesHolder.ofKeyframe(View.SCALE_Y, Keyframe.ofFloat(0f, 1f),
            Keyframe.ofFloat(.1f, .9f), Keyframe.ofFloat(.2f, .9f), Keyframe.ofFloat(.3f, 1.1f), Keyframe.ofFloat(.4f, 1.1f),
            Keyframe.ofFloat(.5f, 1.1f), Keyframe.ofFloat(.6f, 1.1f), Keyframe.ofFloat(.7f, 1.1f), Keyframe.ofFloat(.8f, 1.1f),
            Keyframe.ofFloat(.9f, 1.1f), Keyframe.ofFloat(1f, 1f));

    PropertyValuesHolder pvhRotate = PropertyValuesHolder.ofKeyframe(View.ROTATION, Keyframe.ofFloat(0f, 0f),
            Keyframe.ofFloat(.1f, -3f * shakeFactor), Keyframe.ofFloat(.2f, -3f * shakeFactor), Keyframe.ofFloat(.3f, 3f * shakeFactor),
            Keyframe.ofFloat(.4f, -3f * shakeFactor), Keyframe.ofFloat(.5f, 3f * shakeFactor), Keyframe.ofFloat(.6f, -3f * shakeFactor),
            Keyframe.ofFloat(.7f, 3f * shakeFactor), Keyframe.ofFloat(.8f, -3f * shakeFactor), Keyframe.ofFloat(.9f, 3f * shakeFactor),
            Keyframe.ofFloat(1f, 0));

    return ObjectAnimator.ofPropertyValuesHolder(view, pvhScaleX, pvhScaleY, pvhRotate).setDuration(1000);
}
```

Application
-----------

### application name

```java
public static String getApplicationName(Context context) {
    try {
        return context.getString(context.getPackageManager().getPackageInfo(context.getPackageName(), 0).applicationInfo.labelRes);
    } catch (Exception e) {
        Log.e(TAG, "Failed to get application name", e);
        return null;
    }
}
```

### version name

```java
public static String getVersionName(Context context) {
	try {
		return context.getPackageManager().getPackageInfo(context.getPackageName(), 0).versionName;
	} catch (Exception e) {
		Log.e(TAG, "Failed to get application version number", e);
		return null;
	}
}
```

### version code

```java
public static String getVersionCode(Context context) {
	try {
		return Integer.toString(context.getPackageManager().getPackageInfo(context.getPackageName(), 0).versionCode);
	} catch (Exception e) {
		Log.e(TAG, "Failed to get application version code", e);
		return null;
	}
}
```

### kill background processes

```xml
<uses-permission android:name="android.permission.KILL_BACKGROUND_PROCESSES" />
```

```java
public static void killBackgroundProcesses(Context context, String packageName) {
	((ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE)).killBackgroundProcesses(packageName);
}
```

### restart

```java
public static void restart(Activity activity) {
	Intent intent = activity.getPackageManager().getLaunchIntentForPackage(activity.getPackageName());
	intent.setData(activity.getIntent().getData());
	PendingIntent pi = PendingIntent.getActivity(activity, 1111, intent, PendingIntent.FLAG_CANCEL_CURRENT);
	AlarmManager mgr = (AlarmManager) activity.getSystemService(Context.ALARM_SERVICE);
	mgr.set(AlarmManager.RTC, System.currentTimeMillis() + 100, pi);
	android.os.Process.killProcess(android.os.Process.myPid());
}
```

Build
-----

### OS version

```java
public static String getOsVersion() {
    return android.os.Build.VERSION.RELEASE;
}
```

### has Cupcake

```java
public static boolean hasCupcake() {
    return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.CUPCAKE;
}
```

### has Donut

```java
public static boolean hasDonut() {
    return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.DONUT;
}
```

### has Eclair

```java
public static boolean hasEclair() {
    return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.ECLAIR;
}
```

### has Froyo

```java
public static boolean hasFroyo() {
    return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.FROYO;
}
```

### has Gingerbread

```java
public static boolean hasGingerbread() {
    return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.GINGERBREAD;
}

```

### has Honeycomb

```java
public static boolean hasHoneycomb() {
    return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.HONEYCOMB;
}
```

### has Ice Cream Sandwich

```java
public static boolean hasIceCreamSandwich() {
    return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.ICE_CREAM_SANDWICH;
}
```

### has Jelly Bean

```java
public static boolean hasJellyBean() {
    return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.JELLY_BEAN;
}
```

### has KitKat

```java
public static boolean hasKitKat() {
    return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.KITKAT;
}
```
