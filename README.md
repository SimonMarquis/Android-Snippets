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
  * [Device](#device)
    * [Device name](#device-name)
    * [SDK version](#sdk-version)
    * [Phone number](#phone-number)
    * [has Camera](#has-camera)
    * [is emulator](#is-emulator)
  * [InputFilter](#inputfilter)
    * [All Lower](#all-lower)
    * [Restricted chars](#restricted-chars)
  * [Intent](#intent)
    * [is Intent available](#is-intent-available)
    * [browse](#browse)
    * [share](#share)
    * [dial](#dial)
    * [call](#call)
    * [sms](#sms)
    * [mms](#mms)
    * [email](#email)
    * [maps](#maps)
    * [navigation](#navigation)
    * [install](#install)
    * [uninstall](#uninstall)
    * [playStore](#playstore)
    * [select contact](#select-contact)
    * [take picture](#take-picture)
    * [take video](#take-video)
    * [wifi settings](#wifi-settings)
  * [Logcat](#logcat)
    * [clear](#clear)
    * [capture](#capture)
  * [Network](#network)
    * [is online](#is-online)
    * [enable wifi](#enable-wifi)
  * [UI](#ui)
    * [dp2px](#dp2px)
    * [px2dp](#px2dp)
    * [show soft keyboard](#show-soft-keyboard)
    * [hide soft keyboard](#hide-soft-keyboard)
    * [keep screen on](#keep-screen-on)
    * [colored drawable](#colored-drawable)
    * [capture view](#capture-view)
    * [capture layout](#capture-layout)
    * [increase hit rect](#increase-hit-rect)
  * [Others](#others)
    * [bytes2size](#bytes2size)
    * [small indeterminate progress bar](#small-indeterminate-progress-bar)
    * [ActionBar height](#actionbar-height)
    * [ListView padding](#listview-padding)
  * [Online tools](#online-tools)


ADB (Android Debug Bridge)
--------------------------

### list of devices

```bash
#!/bin/bash
if [[ ! $PATH_TO_ADB ]]; then
	PATH_TO_ADB=$(which adb)
fi
 
if [[ ! $PATH_TO_ADB ]]; then
	if [[ ! $ANDROID_HOME ]]; then
		echo "Failed to determine path to adb; consider setting ANDROID_HOME to your SDK directory or PATH_TO_ADB to the path to ADB"
		exit 1
	fi
	PATH_TO_ADB="$ANDROID_HOME/platform-tools/adb"
fi
 
devices=$($PATH_TO_ADB devices | grep -E "device\$" | cut -f1)
 
echo
echo " MODEL           | PRODUCT         | SDK          | SERIAL"
echo '-----------------|-----------------|--------------|----------------------'
for device in $devices; do
	model=$($PATH_TO_ADB -s "$device" shell getprop ro.product.model | tr -d '\r')
	version=$($PATH_TO_ADB -s "$device" shell getprop ro.build.version.release | tr -d '\r')
	sdk=$($PATH_TO_ADB -s "$device" shell getprop ro.build.version.sdk | tr -d '\r')
	product=$($PATH_TO_ADB -s "$device" shell getprop ro.build.product | tr -d '\r')
	printf ' %-15s | %-15s | [%-5s ~ %2s] | %-20s \n' "$model" "$product" "$version" "$sdk" "$device"
done
echo
```

### screen capture

```bash
file=$(date +%Y.%m.%d-%H.%M.%S).png \
	&& path="/sdcard/$file" \
	&& adb shell screencap -p "$path" \
	&& adb pull "$path" \
	&& adb shell rm "$path" \
	&& echo "Screenshot saved at $(pwd)/$file" \
	|| echo "Screenshot failed"
```

### screen record

 * **bit rate**  `--bit-rate <rate>`
 * **size**  `--size <widthXheight>`
 * **time limit**  `--time-limit <time>`

```bash
file=$(date +%Y.%m.%d-%H.%M.%S).mp4 \
	&& path="/sdcard/$file" \
	&& time=${1-30} \
	&& bitrate=${2-20000000} \
	&& adb shell screenrecord --time-limit "$time" --bit-rate "$bitrate" "$path" \
	&& echo "Success" \
	&& echo "Uploading..." \
	&& adb pull "$path" \
	&& adb shell rm "$path" \
	&& echo "Screencapture saved at $(pwd)/$file" \
	|| echo "Screencapture failed"
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

# send broadcast (--es | --esn | --ez | --ei | --el | --ef | --eu | --eia | --ela | --efa)
adb shell am broadcast -a your.custom.ACTION -n your.package.name/.YourBroadcastReceiver -e "key" "value"

```


Animation
---------

### flash

```java
public static ObjectAnimator flash(final View view, final float alphaFactor) {
	final PropertyValuesHolder pvhAlpha = PropertyValuesHolder.ofKeyframe(android.view.View.ALPHA, Keyframe.ofFloat(0f, 1f), Keyframe.ofFloat(.25f, alphaFactor), Keyframe.ofFloat(0.5f, 1f), Keyframe.ofFloat(.75f, alphaFactor), Keyframe.ofFloat(1f, 1f));
	return ObjectAnimator.ofPropertyValuesHolder(view, pvhAlpha).setDuration(800);
}
```

### nope

Introduced by [@cyrilmottier](https://github.com/cyrilmottier) on [Google+](https://plus.google.com/+CyrilMottier/posts/FABaJhRMCuy)

```java
public static ObjectAnimator nope(final View view, final int delta) {
	final PropertyValuesHolder pvhTranslateX = PropertyValuesHolder.ofKeyframe(View.TRANSLATION_X, Keyframe.ofFloat(0f, 0),
			Keyframe.ofFloat(.10f, -delta), Keyframe.ofFloat(.26f, delta), Keyframe.ofFloat(.42f, -delta), Keyframe.ofFloat(.58f, delta),
			Keyframe.ofFloat(.74f, -delta), Keyframe.ofFloat(.90f, delta), Keyframe.ofFloat(1f, 0f));
	return ObjectAnimator.ofPropertyValuesHolder(view, pvhTranslateX).setDuration(500);
}
```

### pulse

```java
public static ObjectAnimator pulse(final View view, final float pulseFactor) {
	final PropertyValuesHolder pvhScaleX = PropertyValuesHolder.ofKeyframe(android.view.View.SCALE_X, Keyframe.ofFloat(0f, 1f), Keyframe.ofFloat(.5f, pulseFactor * 1f), Keyframe.ofFloat(1f, 1f));
	final PropertyValuesHolder pvhScaleY = PropertyValuesHolder.ofKeyframe(android.view.View.SCALE_Y, Keyframe.ofFloat(0f, 1f), Keyframe.ofFloat(.5f, pulseFactor * 1f), Keyframe.ofFloat(1f, 1f));
	return ObjectAnimator.ofPropertyValuesHolder(view, pvhScaleX, pvhScaleY).setDuration(500);
}
```

### spring

```java
public static ObjectAnimator spring(final View view, final float springFactor) {
	final PropertyValuesHolder pvhScaleX = PropertyValuesHolder.ofKeyframe(android.view.View.SCALE_X, Keyframe.ofFloat(0f, 1f),
			Keyframe.ofFloat(0.25f, springFactor * 1.35f), Keyframe.ofFloat(0.5f, 0.65f / springFactor),
			Keyframe.ofFloat(0.75f, springFactor * 1.15f), Keyframe.ofFloat(1f, 1f));
	final PropertyValuesHolder pvhScaleY = PropertyValuesHolder.ofKeyframe(android.view.View.SCALE_Y, Keyframe.ofFloat(0f, 1f),
			Keyframe.ofFloat(0.25f, 0.65f / springFactor), Keyframe.ofFloat(0.5f, springFactor * 1.35f),
			Keyframe.ofFloat(0.75f, 0.85f / springFactor), Keyframe.ofFloat(1f, 1f));
	return ObjectAnimator.ofPropertyValuesHolder(view, pvhScaleX, pvhScaleY).setDuration(600);
}
```

### tada

Introduced by [@cyrilmottier](https://github.com/cyrilmottier) on [Google+](https://plus.google.com/+CyrilMottier/posts/FABaJhRMCuy)

```java
public static ObjectAnimator tada(final View view, final float shakeFactor) {
	final PropertyValuesHolder pvhScaleX = PropertyValuesHolder.ofKeyframe(View.SCALE_X, Keyframe.ofFloat(0f, 1f),
			Keyframe.ofFloat(.1f, .9f), Keyframe.ofFloat(.2f, .9f), Keyframe.ofFloat(.3f, 1.1f), Keyframe.ofFloat(.4f, 1.1f),
			Keyframe.ofFloat(.5f, 1.1f), Keyframe.ofFloat(.6f, 1.1f), Keyframe.ofFloat(.7f, 1.1f), Keyframe.ofFloat(.8f, 1.1f),
			Keyframe.ofFloat(.9f, 1.1f), Keyframe.ofFloat(1f, 1f));

	final PropertyValuesHolder pvhScaleY = PropertyValuesHolder.ofKeyframe(View.SCALE_Y, Keyframe.ofFloat(0f, 1f),
			Keyframe.ofFloat(.1f, .9f), Keyframe.ofFloat(.2f, .9f), Keyframe.ofFloat(.3f, 1.1f), Keyframe.ofFloat(.4f, 1.1f),
			Keyframe.ofFloat(.5f, 1.1f), Keyframe.ofFloat(.6f, 1.1f), Keyframe.ofFloat(.7f, 1.1f), Keyframe.ofFloat(.8f, 1.1f),
			Keyframe.ofFloat(.9f, 1.1f), Keyframe.ofFloat(1f, 1f));

	final PropertyValuesHolder pvhRotate = PropertyValuesHolder.ofKeyframe(View.ROTATION, Keyframe.ofFloat(0f, 0f),
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
public static String getApplicationName(final Context context) {
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
public static String getVersionName(final Context context) {
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
public static String getVersionCode(final Context context) {
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
public static void killBackgroundProcesses(final Context context, final String packageName) {
	((ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE)).killBackgroundProcesses(packageName);
}
```

### restart

```java
public static void restart(final Activity activity) {
	final Intent intent = activity.getPackageManager().getLaunchIntentForPackage(activity.getPackageName());
	final intent.setData(activity.getIntent().getData());
	final PendingIntent pi = PendingIntent.getActivity(activity, 1111, intent, PendingIntent.FLAG_CANCEL_CURRENT);
	final AlarmManager am = (AlarmManager) activity.getSystemService(Context.ALARM_SERVICE);
	am.set(AlarmManager.RTC, System.currentTimeMillis() + 100, pi);
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

public static boolean hasGingerbreadMR1() {
	return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.GINGERBREAD_MR1;
}
```

### has Honeycomb

```java
public static boolean hasHoneycomb() {
	return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.HONEYCOMB;
}

public static boolean hasHoneycombMR1() {
	return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.HONEYCOMB_MR1;
}

public static boolean hasHoneycombMR2() {
	return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.HONEYCOMB_MR2;
}
```

### has Ice Cream Sandwich

```java
public static boolean hasIceCreamSandwich() {
	return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.ICE_CREAM_SANDWICH;
}

public static boolean hasIceCreamSandwichMR1() {
	return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.ICE_CREAM_SANDWICH_MR1;
}
```

### has Jelly Bean

```java
public static boolean hasJellyBean() {
	return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.JELLY_BEAN;
}

public static boolean hasJellyBeanMR1() {
	return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.JELLY_BEAN_MR1;
}

public static boolean hasJellyBeanMR2() {
	return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.JELLY_BEAN_MR2;
}
```

### has KitKat

```java
public static boolean hasKitKat() {
	return android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.KITKAT;
}
```

Device
------

### Device name

```java
public static String getDeviceName() {
	final String manufacturer = android.os.Build.MANUFACTURER;
	final String model = android.os.Build.MODEL;
	return model.startsWith(manufacturer) ? model : manufacturer + " " + model;
}
```

### SDK version

```java
public static int getSdkVersion() {
	try {
		return android.os.Build.VERSION.class.getField("SDK_INT").getInt(null);
	} catch (Exception e) {
		Log.e(TAG, "Failed to get SDK version", e);
	}
	return 0;
}
```

### Phone number

```xml
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
```

```java
public static String getPhoneNumber(final Context context) {
	return ((TelephonyManager) context.getSystemService(Context.TELEPHONY_SERVICE)).getLine1Number();
}
```

### has Camera

```java
public static boolean hasCamera(final Context context) {
	final PackageManager pm = context.getPackageManager();
	return pm.hasSystemFeature(PackageManager.FEATURE_CAMERA) || pm.hasSystemFeature(PackageManager.FEATURE_CAMERA_FRONT);
}
```

### is emulator

```java
public static boolean isEmulator() {
	return android.os.Build.MODEL.equals("sdk") || android.os.Build.MODEL.equals("google_sdk");
}
```

InputFilter
-----------

### All Lower

```java
public class AllLower implements InputFilter {
	@Override
	public CharSequence filter(final CharSequence source, final int start, final int end, final Spanned dest, final int dstart, final int dend) {
		for (int i = start; i < end; i++) {
			if (Character.isUpperCase(source.charAt(i))) {
				final char[] v = new char[end - start];
				TextUtils.getChars(source, start, end, v, 0);
				final String s = new String(v).toLowerCase();
				if (source instanceof Spanned) {
					final SpannableString sp = new SpannableString(s);
					TextUtils.copySpansFrom((Spanned) source, start, end, null, sp, 0);
					return sp;
				} else {
					return s;
				}
			}
		}
		return null;
	}
}
```

### Restricted chars

```java
public class RestrictedChars implements android.text.InputFilter {

	private char[] restrictedChars;

	public RestrictedChars(char... acceptedChars) {
		this.restrictedChars = acceptedChars;
	}

	@Override
	public CharSequence filter(final CharSequence source, final int start, int end, final Spanned dest, final int dstart, final int dend) {
		int i;
		for (i = start; i < end; i++) {
			if (!ok(restrictedChars, source.charAt(i))) {
				break;
			}
		}

		if (i == end) {
			return null;
		}

		if (end - start == 1) {
			return "";
		}

		final SpannableStringBuilder filtered = new SpannableStringBuilder(source, start, end);
		i -= start;
		end -= start;

		for (int j = end - 1; j >= i; j--) {
			if (!ok(restrictedChars, source.charAt(j))) {
				filtered.delete(j, j + 1);
			}
		}
		return filtered;
	}

	protected boolean ok(char[] accept, char c) {
		for (int i = accept.length - 1; i >= 0; i--) {
			if (accept[i] == c) {
				return true;
			}
		}
		return false;
	}
}
```

Intent
------

### is Intent available

```java
public static boolean isIntentAvailable(final Context context, final Intent intent) {
	return !context.getPackageManager().queryIntentActivities(intent, PackageManager.MATCH_DEFAULT_ONLY).isEmpty();
}
```

### browse

```java
public static Intent browse(final Context context, final String url) {
	return new Intent(Intent.ACTION_VIEW, Uri.parse(url));
}
```

### share

```java
public static Intent share(final Context context, final String subject, final String message) {
	final Intent intent = new Intent(Intent.ACTION_SEND);
	intent.setType("text/plain");
	intent.putExtra(Intent.EXTRA_TEXT, message);
	intent.putExtra(Intent.EXTRA_SUBJECT, subject);
	intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
	return intent;
}
```

### dial

```java
public static Intent dial(final Context context, final String number) {
	return new Intent(Intent.ACTION_DIAL, Uri.parse("tel:" + number));
}
```

### call

```xml
<uses-permission android:name="android.permission.CALL_PHONE" />
```

```java
public static Intent call(final Context context, final String number) {
	return new Intent(Intent.ACTION_CALL, Uri.parse("tel:" + number));
}
```

### sms

```java
public static Intent sms(final Context context, final String number, final String message) {
	final Uri uri = Uri.parse("smsto:" + number);
	final Intent intent = new Intent(Intent.ACTION_SENDTO, uri);
	intent.putExtra("sms_body", message);
	return intent;
}
```

### mms

```java
public static Intent mms(final Context context, final String number, final String subject, final String message, final Uri attachment) {
	final Uri uri = Uri.parse("mmsto:" + number);
	final Intent intent = new Intent(Intent.ACTION_SENDTO, uri);
	intent.putExtra("subject", subject);
	intent.putExtra("sms_body", message);
	if (attachment != null)
		intent.putExtra(Intent.EXTRA_STREAM, attachment);
	return intent;
}
```

### email

```java
public static Intent email(final Context context, final String[] to, final String [] cc, final String [] bcc, final String subject, final String body, final Uri attachment) {
	final Intent intent = new Intent(Intent.ACTION_SENDTO);
	intent.setData(Uri.parse("mailto:"));
	if (to != null)
		intent.putExtra(Intent.EXTRA_EMAIL, to);
	if (cc != null)
		intent.putExtra(Intent.EXTRA_CC, cc);
	if (bcc != null)
		intent.putExtra(Intent.EXTRA_BCC, bcc);
	if (body != null)
		intent.putExtra(Intent.EXTRA_TEXT, body);
	if (subject != null)
		intent.putExtra(Intent.EXTRA_SUBJECT, subject);
	if (attachment != null)
		intent.putExtra(Intent.EXTRA_STREAM, attachment);
	intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
	return intent;
}
```

### maps

```java
public static Intent maps(final Context context, final double lat, final double lng) {
	return new Intent(Intent.ACTION_VIEW, Uri.parse("geo:" + lat + "," + lng));
}
```

```java
public static Intent maps(final Context context, final double lat, final double lng, final int zoom) {
	return new Intent(Intent.ACTION_VIEW, Uri.parse("geo:" + lat + "," + lng + "?z=" + zoom));
}
```

```java
public static Intent maps(final Context context, final double lat, final double lng, final String label) throws UnsupportedEncodingException {
	return new Intent(Intent.ACTION_VIEW, Uri.parse("geo:0,0?q=" + lat + "," + lng + "(" + URLEncoder.encode(label, "UTF-8") + ")"));
}
```

```java
public static Intent maps(final Context context, final String query) throws UnsupportedEncodingException {
    return new Intent(Intent.ACTION_VIEW, Uri.parse("geo:0,0?q=" + URLEncoder.encode(query, "UTF-8")));
}
```

### navigation

```java
/**
 * @param mode d: Driving, w: Walking, r: Public transit, b: Biking
 */
public static Intent navigation(final Context context, final String address, final String mode) throws UnsupportedEncodingException {
	return new Intent(Intent.ACTION_VIEW, Uri.parse("google.navigation:q=" + URLEncoder.encode(address, "UTF-8") +( mode == null ? "" : ("&mode=" + mode))));
}
```

```java
/**
 * @param mode d: Driving, w: Walking, r: Public transit, b: Biking
 */
public static Intent navigate(final Context context, final double lat, final double lng, final String mode) {
	return new Intent(Intent.ACTION_VIEW, Uri.parse("google.navigation:q=" + lat + "," + lng + (mode == null ? "" : ("&mode=" + mode))));
}
```

### install

```java
public static Intent install(final Context context, final Uri file) {
	final Intent intent = new Intent(Intent.ACTION_VIEW);
	intent.setDataAndType(file, "application/vnd.android.package-archive");
	return intent;
}
```

### uninstall

```java
public static Intent uninstall(final Context context, final String packageName) {
	return new Intent(Intent.ACTION_DELETE, Uri.parse("package:" + packageName));
}
```

### playStore

```java
public static Intent playStore(final Context context, final String packageName) {
	return new Intent(Intent.ACTION_VIEW, Uri.parse("market://details?id=" + packageName));
}
```

```java
public static Intent playStore(final Context context, final String publisherName) {
	return new Intent(Intent.ACTION_VIEW, Uri.parse("market://search?q=pub:" + publisherName));
}
```

```java
/**
 * @param category apps, movies, music, newsstand, devices
 */
public static Intent playStore(final Context context, final String search, final String category) {
	return new Intent(Intent.ACTION_VIEW, Uri.parse("market://search?q=" + search + (category == null ? "" : ("&c=" + category))));
}
```

```java
/**
 * @param collection featured, editors_choice, topselling_paid, topselling_free, topselling_new_free, topselling_new_paid, topgrossing, movers_shakers, topselling_paid_game
 */
public static Intent playStore(final Context context, final String collection) {
	return new Intent(Intent.ACTION_VIEW, Uri.parse("market://apps/collection/" + collection));
}
```

### select contact

```java
public static Intent selectContact(final Context context) {
	final Intent intent = new Intent(Intent.ACTION_PICK);
	intent.setType(ContactsContract.Contacts.CONTENT_TYPE);
	return intent;
}
```

### take picture

```java
public static Intent picture(final Context context, final File file) {
	final Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
	intent.putExtra(MediaStore.EXTRA_OUTPUT, Uri.fromFile(file));
	return intent;
}
```

### take video

```java
public static Intent video(final Context context, final File file) {
	final Intent intent = new Intent(MediaStore.ACTION_VIDEO_CAPTURE);
	intent.putExtra(MediaStore.EXTRA_OUTPUT, Uri.fromFile(file));
	return intent;
}
```

### wifi settings

```java
public static Intent wifi(final Context context) {
	return new Intent(Settings.ACTION_WIFI_SETTINGS);
}
```

Logcat
------

### clear

```java
public static void clear() {
	try {
		Runtime.getRuntime().exec(new String[] { "logcat", "-c" });
	} catch (Exception e) {
		Log.e(TAG, "Failed to clear logcat " + e.getMessage());
	}
}
```

### capture

```java
/**
 * @param args More intel on <a href="http://developer.android.com/tools/debugging/debugging-log.html">developer.android.com</a>
 */
public static String logcat(final String[] args) {
	try {
		final Process process = Runtime.getRuntime().exec(args != null ? args : new String[] { "logcat", "-d" });
		final BufferedReader br = new BufferedReader(new InputStreamReader(process.getInputStream()));
		final String separator = System.getProperty("line.separator");
		final StringBuilder sb = new StringBuilder();
		String line;
		while ((line = br.readLine()) != null) {
			sb.append(line).append(separator);
		}
		return sb.toString();
	} catch (Exception e) {
		Log.e(TAG, "Failed to capture logcat " + e.getMessage());
		return null;
	}
}
```

Network
-------

### is online

```xml
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

```java
public static boolean isOnline(final Context context) {
	final NetworkInfo ni = ((ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE)).getActiveNetworkInfo();
	return (ni != null && ni.isAvailable() && ni.isConnected());
}
```

```java
/**
 * {@link ConnectivityManager#TYPE_WIFI},
 * {@link ConnectivityManager#TYPE_ETHERNET},
 * {@link ConnectivityManager#TYPE_MOBILE}, ...
 */
public static boolean isOnline(final Context context, final int type) {
	final NetworkInfo ni = ((ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE)).getNetworkInfo(type);
	return ni != null && ni.isAvailable() && ni.isConnected();
}
```

### enable wifi

```xml
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
```

```java
public static boolean enableWifi(final Context context, final boolean enable) {
	return ((WifiManager) context.getSystemService(Context.WIFI_SERVICE)).setWifiEnabled(enable);
}
```

UI
---

### dp2px

```java
public static float dp2px(final float dp, final Context context) {
	final DisplayMetrics metrics = context.getResources().getDisplayMetrics();
	return dp * metrics.densityDpi / 160f;
}
```

### px2dp

```java
public static float px2dp(final float px, final Context context) {
	final DisplayMetrics metrics = context.getResources().getDisplayMetrics();
	return px / (metrics.densityDpi / 160f);
}
```

### show soft keyboard

```java
public static void showSoftKeyboard(final View view, final ResultReceiver resultReceiver) {
	final Configuration config = view.getContext().getResources().getConfiguration();
	if (config.hardKeyboardHidden == Configuration.HARDKEYBOARDHIDDEN_YES) {
		final InputMethodManager imm = (InputMethodManager) view.getContext().getSystemService(Context.INPUT_METHOD_SERVICE);
		if (resultReceiver != null) {
			imm.showSoftInput(view, InputMethodManager.SHOW_IMPLICIT, resultReceiver);
		} else {
			imm.showSoftInput(view, InputMethodManager.SHOW_IMPLICIT);
		}
	}
}
```

### hide soft keyboard

```java
public static void hideSoftKeyboard(final View view) {
	final InputMethodManager imm = (InputMethodManager) view.getContext().getSystemService(Context.INPUT_METHOD_SERVICE);
	imm.hideSoftInputFromWindow(view.getWindowToken(), 0);
}
```

### keep screen on

```java
public static void keepScreenOn(final Activity activity, final boolean keep) {
	if (keep) {
		activity.getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
	} else {
		activity.getWindow().clearFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
	}
}
```

### colored drawable

Introduced by [@dlew](https://github.com/dlew) on [his blog](http://blog.danlew.net/2014/08/18/fast-android-asset-theming-with-colorfilter/)

```java
public static Drawable getColoredDrawable(final Resources res, final int drawableResId, final int color) {
	final Drawable drawable = res.getDrawable(drawableResId);
	drawable.setColorFilter(color, PorterDuff.Mode.SRC_IN);
	return drawable;
}
```

### capture view

```java
public static Bitmap captureView(final View view) {
	final Bitmap bitmap = Bitmap.createBitmap(view.getWidth(), view.getHeight(), Bitmap.Config.ARGB_8888);
	view.draw(new Canvas(bitmap));
	return bitmap;
}
```

### capture layout

```java
public static Bitmap captureLayout(final Context context, final int width, final int height, final int layoutResId) {
    final Bitmap bitmap = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
	final Canvas canvas = new Canvas(bitmap);
	final LayoutInflater inflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
	final View view = inflater.inflate(layoutResId, null);
	view.setDrawingCacheEnabled(true);
	view.measure(MeasureSpec.makeMeasureSpec(canvas.getWidth(), MeasureSpec.EXACTLY), MeasureSpec.makeMeasureSpec(canvas.getHeight(), MeasureSpec.EXACTLY));
	view.layout(0, 0, view.getMeasuredWidth(), view.getMeasuredHeight());
	canvas.drawBitmap(view.getDrawingCache(), 0, 0, new Paint());
	return bitmap;
}
```

### increase hit rect

```java
public static void increaseHitRect(final int top, final int left, final int bottom, final int right, final View delegate) {
	final View parent = (View) delegate.getParent();
	parent.post(new Runnable() {
		public void run() {
			final Rect r = new Rect();
			delegate.getHitRect(r);
			r.top -= top;
			r.left -= left;
			r.bottom += bottom;
			r.right += right;
			parent.setTouchDelegate(new TouchDelegate(r, delegate));
		}
	});
}
```

Others
------

### bytes2size

```java
public static String bytes2size(final long size) {
	if (size <= 0)
		return "0";
	final String[] units = new String[] { "B", "KB", "MB", "GB", "TB", "PB", "EB" };
	final int digitGroups = (int) (Math.log10(size) / Math.log10(1024));
	return new DecimalFormat("#,##0.#").format(size / Math.pow(1024, digitGroups)) + " " + units[digitGroups];
}
```

### small indeterminate progress bar

```java
<style name="Widget.ActionBar.ActivityIndicator" parent="...">
	<item name="android:minWidth">48dp</item>
	<item name="android:maxWidth">48dp</item>
	<item name="android:minHeight">32dp</item>
	<item name="android:maxHeight">32dp</item>
</style>
```

### ActionBar height

```java
public static int getActionBarHeight(final Activity activity) {
	final TypedValue value = new TypedValue();
	activity.getTheme().resolveAttribute(android.R.attr.actionBarSize, value, true);
	return activity.getResources().getDimensionPixelSize(value.resourceId);
}
```

### ListView padding

```xml
<ListView
    ...
    android:clipToPadding="false"
    android:scrollbarStyle="outsideOverlay" />
```

Online tools
------------

[Android Asset Studio](http://romannurik.github.io/AndroidAssetStudio) by [@romannurik](https://github.com/romannurik)

[Android Asset Studio for Material Design](http://shreyasachar.github.io/AndroidAssetStudio) by [@shreyasachar](https://github.com/shreyasachar)

[Gradle, Please](http://gradleplease.appspot.com) by [@broady](https://github.com/broady)

[parcelabler](http://www.parcelabler.com) by [@dallasgutauckis](https://github.com/dallasgutauckis)

[gitignore.io](https://www.gitignore.io)

[Android Arsenal](http://android-arsenal.com)

[Cheat Sheet for Graphic Designer](http://petrnohejl.github.io/Android-Cheatsheet-For-Graphic-Designers)

[SystemUiHelper.java](https://gist.github.com/chrisbanes/73de18faffca571f7292) by [@chrisbanes](https://github.com/chrisbanes)
