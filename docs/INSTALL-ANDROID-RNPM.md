# Android Installation with `react-native link`

```shell
npm install git+https://git@github.com:transistorsoft/react-native-background-geolocation-android.git --save
```

#### With React Native 0.27+

```shell
react-native link react-native-background-geolocation-android
react-natvie link react-native-background-fetch
```

## Gradle Configuration

react-native link does a nice job, but we need to do a bit of manual setup.

### :open_file_folder: **`android/build.gradle`**

```diff
allprojects {
    repositories {
        mavenLocal()
        jcenter()
        maven {
            // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
            url "$rootDir/../node_modules/react-native/android"
        }
        // Google now hosts their latest API dependencies on their own maven server.  
        // React Native will eventually add this to their app template.
+       maven {
+           url 'https://maven.google.com'
+       }
+       maven {
+           url "$rootDir/../node_modules/react-native-background-geolocation-android/android/libs"
+       }
+       maven {
+           url "$rootDir/../node_modules/react-native-background-fetch/android/libs"
+       }
    }
}

/**
-* !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
-* !!! THE FOLLOWING IS OPTIONAL BUT HIGHLY RECOMMENDED FOR YOUR SANITY !!!
-* !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
*
* Do you hate Gradle conflicts where other plugin require some particular
* version of play-services or define a compileSdkVersion, buildToolsVersion
* which conflicts with that of your app?  Me too!
*
* If you define these key gradle configuration variables globally, the 
* background-geolocation plugin (and any other "wise" plugins you've installed) 
* can align themselves to YOUR desired versions!  You should define these variables 
* as desired according to current values in your app/build.gradle
*
* You'll find that more and more plugins are beginning to wise up to checking 
* for the presense of global gradle variables like this.
*
* BackgroundGeolocation is aware of the following variables:
*/
+ext {
+    compileSdkVersion   = 26
+    targetSdkVersion    = 26
+    buildToolsVersion   = "26.0.2"
+    supportLibVersion   = "26.1.0"
+    playServicesVersion = "11.8.0" 
+}
-// BackgroundGeolocation is also aware of googlePlayServicesVersion if you prefer
```

#### :information_source: Project-wide Configuration Properties

The technique of **defining project-wide properties** can be found in the **Android Developer Document** [Gradle Tip &amp; Tricks](https://developer.android.com/studio/build/gradle-tips.html) (see *Configure project-wide properties*) and another good explanation [here](https://segunfamisa.com/posts/android-gradle-extra-properties).  The *BackgroundGeolocation* plugin [is aware of the presense of these configuration properties](../android/build.gradle#L3-L18).

-------------------------------------------------------------------------------


### :open_file_folder: **`android/app/build.gradle`**

```diff
-/**
-* OPTIONAL:  If you've implemeted the "OPTIONAL BUT HIGHLY RECOMMENDED" note
-* above, you can define your compileSdkVersion, buildToolsVersion, targetSdkVersion 
-* using your own global variables as well:
-* Android Studio is smart enough to be aware of the evaulated values here,
-* to offer upgrade notices when applicable.
-*
-*/
android {
+    compileSdkVersion rootProject.compileSdkVersion
+    buildToolsVersion rootProject.buildToolsVersion

    defaultConfig {
+        targetSdkVersion rootProject.targetSdkVersion
         .
         .
         .
    }
}

dependencies {
    compile project(':react-native-background-geolocation-android')
    compile project(':react-native-background-fetch')

    // You are advised to use latest appcompat-v7 corresponding to your compileSdkVersion
    // eg:  if compileSdkVersion 27 -> appcompat-v7:27.x.x
    //      if compileSdkVersion 26 -> appcompat-v7:26.x.x
    //      if compileSdkVersion 25 -> appcompat-v7:25.x.x
    // NOTE:  It's up to you to define the variable supportLibVersion 
    // as noted above.  IT IS HIGHLY RECOMMENDED TO DO THIS.
+   compile "com.android.support:appcompat-v7:$rootProject.supportLibVersion"
}
```


## AndroidManifest.xml

```diff
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.transistorsoft.backgroundgeolocation.react">

  <application
    android:name=".MainApplication"
    android:allowBackup="true"
    android:label="@string/app_name"
    android:icon="@mipmap/ic_launcher"
    android:theme="@style/AppTheme">

    <!-- react-native-background-geolocation licence -->
+   <meta-data android:name="com.transistorsoft.locationmanager.license" android:value="YOUR_LICENCE_KEY_HERE" />
    .
    .
    .
  </application>
</manifest>

```


## Proguard Config

### :open_file_folder: `proguard-rules.pro` (`android/app/proguard-rules.pro`)

```proguard
# BackgroundGeolocation lib tslocationmanager.aar is *already* proguarded
-keep class com.transistorsoft.** { *; }
-dontwarn com.transistorsoft.**

# BackgroundGeolocation (EventBus)
-keepclassmembers class * extends de.greenrobot.event.util.ThrowableFailureEvent {
    <init>(java.lang.Throwable);
}
-keepattributes *Annotation*
-keepclassmembers class ** {
    @org.greenrobot.eventbus.Subscribe <methods>;
}
-keep enum org.greenrobot.eventbus.ThreadMode { *; }
-keepclassmembers class * extends org.greenrobot.eventbus.util.ThrowableFailureEvent {
    <init>(java.lang.Throwable);
}

# logback
-keep class ch.qos.** { *; }
-keep class org.slf4j.** { *; }
-dontwarn ch.qos.logback.core.net.*

# OkHttp3
-dontwarn okio.**
```

