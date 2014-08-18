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
    * [tada](#tada)
    * [nope](#nope)
    * [pulse](#pulse)
    * [flash](#flash)
    * [spring](#spring)


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
```


Animation
---------

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

### flash

```java
public static ObjectAnimator flash(View view, float alphaFactor) {
	PropertyValuesHolder pvhalpha = PropertyValuesHolder.ofKeyframe(android.view.View.ALPHA, Keyframe.ofFloat(0f, 1f), Keyframe.ofFloat(.25f, alphaFactor), Keyframe.ofFloat(0.5f, 1f), Keyframe.ofFloat(.75f, alphaFactor), Keyframe.ofFloat(1f, 1f));
	return ObjectAnimator.ofPropertyValuesHolder(view, pvhalpha).setDuration(800);
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
