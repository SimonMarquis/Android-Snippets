Android Snippets
================


  * [ADB](#adb-android-debug-bridge)
    * [list of devices](#list-of-devices)
    * [screen capture](#screen-capture)
    * [screen record](#screen-record)
    * [run monkey](#run-monkey)
    * [sqlite](#sqlite)
    * [handy commands](#handy-commands)


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
