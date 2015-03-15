# Android Developer Kit #

[ADK](ADK.md) at it's core offers an off-[Market](http://market.android.com) updating service that lets you provide your users with quicker updates and more intuitive changelog information. With [ADK](ADK.md), your applications can check for published updates on demand without using your own servers or waiting for Android Market to publish.

Using the service, you can also apply a flexible licensing policy on an user-by-user basis — each user is given a unique identifier (User ID) and is listed from the ADK service website. You can then programmatically and manually decide to ban (and unban) users if necessary.

The service includes built in application protection, sideload detection, which automatically checks the running application for tampering or modification, and upon detection locks the application from continued use.

The [ADK](ADK.md) service is a secure means of controlling access to your applications.

This document explains how the [ADK](ADK.md) service works and how to add it to your application.

## Setting Up Your Application on ADK ##

To begin using [ADK](ADK.md) you must first register your application on our service. This can be done after logging in from:

> http://adk.leadbulb.com/apps

Adding a new application is quick and simple, by completing this process you are given a generated App Key. You are now ready to continue.

## Setting Up the with the SDK ##

### Downloading the SDK ###
First you should download the latest SDK from the the [Downloads tab](http://code.google.com/p/adk/downloads/list).

### Including the ADK in your application ###
Next you need to include our jar file in your project.

Open the application's project properties window, as shown below. Select the "Java Build Path" properties group and click Add External JARs..., then choose the ADK Jar and click OK.

![http://adk.leadbulb.com/assets/wiki/AddExternalJar.png](http://adk.leadbulb.com/assets/wiki/AddExternalJar.png)

## Integrating ADK with Your Application ##
Once you've followed the steps above to set up your application on the ADK service and and setting up your development environment, you are ready to begin integrating [ADK](ADK.md) with your application.

Integrating the [ADK](ADK.md) with your application code involves these tasks:

  1. Adding the required permissions your application's manifest.
  1. Implementing a Policy — you can choose one of the full implementations provided in ADK or create your own.
  1. Adding code to check for updates in your application's main Activity or in your Application.

The sections below describe these tasks. When you are done with the integration, you should be able to compile your application successfully and you can begin testing.

For an overview of the full set of source files included in ADK, read our [javadocs](http://adk.leadbulb.com/javadoc/index.html).

### Adding the required permissions your AndroidManifest.xml ###

To use [ADK](ADK.md) your application must request the proper permissions, specifically <font color='green'><code>android.permission.INTERNET</code></font>. If your application does not declare the licensing permission but attempts to initiate an update check, ADK throws an `ApplicationErrorCode MISSING_PERMISSION`.

To request the licensing permission in your application, declare a [&lt;uses-permission&gt;](http://developer.android.com/guide/topics/manifest/uses-permission-element.html) element as a child of `<manifest>`, as follows:

<font color='green'><code>&lt;uses-permission android:name="android.permission.INTERNET"&gt;</code></font>

```xml

<?xml version="1.0" encoding="utf-8"?>

<manifest xmlns:android="http://schemas.android.com/apk/res/android" ...">
<!-- Devices >= 4 will work with ADK. -->
<uses-sdk android:minSdkVersion="4" />
<!-- Required permission to access Internet. -->
<uses-permission android:name="android.permission.INTERNET" />
<!-- Optional permissions, but improve the usage of ADK -->
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
...
<application ...>
...
<activity android:name="com.leadbulb.adk.DialogHandler"
android:theme="@android:style/Theme.Dialog" android:launchMode="singleInstance"
android:excludeFromRecents="true" android:finishOnTaskLaunch="true" />
...


Unknown end tag for &lt;/application&gt;


...


Unknown end tag for &lt;/manifest&gt;


```

### Implementing a Policy ###

[ADK](ADK.md) does not itself determine whether a given user is banned or an whether the application is up to date with the latest version. Rather, that responsibility is left to a Policy implementation that you provide in your application.

Policy is an interface declared by [ADK](ADK.md) that is designed to hold your application's logic for allowing or disallowing user access and when to check for updates, based on the result of an update check to [ADK](ADK.md). To use [ADK](ADK.md), your application must provide an implementation of Policy.

The Policy interface declares three methods, updateAvailable(), allowAccess() and processServerResponse(), which are called by a UpdateChecker instance when processing a response from the [ADK](ADK.md) server. It also declares an enum called ADKResponse, which specifies the update response value passed in calls to processServerResponse().

  * <font color='green'><code>processServerResponse()</code></font> lets you preprocess the raw response data received from the [ADK](ADK.md) server, prior to determining whether to grant access. A typical implementation would extract some or all fields from the response and store the data locally to a persistent store, such as through [SharedPreferences](http://developer.android.com/reference/android/content/SharedPreferences.html) storage, to ensure that the data is accessible across application invocations and device power cycles. For example, a Policy would maintain the timestamp of last successful updater check, the retry count, the  validity period, and similar information in a persistent store, rather than resetting the values each time the application is launched or each time the update check is called.

  * <font color='green'><code>allowAccess()</code></font> determines whether to grant the user access to your application, based on any available response data (from the [ADK](ADK.md) server or from cache) or other application-specific information. For example, your implementation of allowAccess() could take into account additional criteria, such as usage or other data retrieved from a backend server. In all cases, an implementation of allowAccess() should only return true if the user is allowed to use the application. In such cases, your implementation can maintain a count of retry responses and provisionally allow access until the next update check is complete.

  * <font color='green'><code>updateAvailable()</code></font> determines whether there is an update available, based on any available response data from the [ADK](ADK.md) server or from cache.


To simplify the process of adding [ADK](ADK.md) to your application and to provide an illustration of how a Policy should be designed, [ADK](ADK.md) includes two full Policy implementations that you can use without modification or adapt to your needs:

  * [ServerManagedPolicy](ServerManagedPolicy.md), a flexible Policy that uses server-provided settings and cached responses to manage up to date status and access across varied network conditions, and
  * [StrictPolicy](StrictPolicy.md), which does not cache any response data, checking in demand if an update is available and whether to allow the user access to your application.

For most applications, the use of [ServerManagedPolicy](ServerManagedPolicy.md) is highly recommended. [ServerManagedPolicy](ServerManagedPolicy.md) is [ADK](ADK.md) default and is integrated with [ADK](ADK.md).

#### ServerManagedPolicy ####

[ADK](ADK.md) includes a full and recommended implementation of the Policy interface called ServerManagedPolicy. The implementation is integrated with the ADK Jar and serves as the default Policy in the library.

ServerManagedPolicy provides all of the handling for update, license and retry responses. It caches all of the response data locally in a SharedPreferences file. This ensures that the license response data is secure and persists across device power cycles. ServerManagedPolicy provides concrete implementations of the interface methods <font color='green'><code>processServerResponse()</code></font> and <font color='green'><code>allowAccess()</code></font> and <font color='green'><code>updateAvailable()</code></font> and also includes a set of supporting methods and types for managing responses.

Importantly, a key feature of ServerMangedPolicy is its use of server-provided settings as the basis for managing licensing across an application's refund period and through varying network and error conditions. When an application contacts the Android Market server for a license check, the server appends several settings as key-value pairs in the extras field of certain license response types. For example, the server provides recommended values for the application's license validity period, retry grace period, and maximum allowable retry count, among others. ServerManagedPolicy extracts the values from the license response in its processServerResponse() method and checks them in its allowAccess() method. For a list of the server-provided settings used by ServerManagedPolicy, see Server Response Extras in the Appendix of this document.

For convenience, best performance, and the benefit of using license settings from the Android Market server, using ServerManagedPolicy as your licensing Policy is strongly recommended.

If you are concerned about the security of license response data that is stored locally in SharedPreferences, you can use a stronger obfuscation algorithm or design a stricter Policy that does not store license data. The LVL includes an example of such a Policy — see StrictPolicy for more information.

To use ServerManagedPolicy, simply import it to your Activity, create an instance, and pass a reference to the instance when constructing your LicenseChecker. See Instantiate LicenseChecker and LicenseCheckerCallback for more information.

#### StrictPolicy ####

[ADK](ADK.md) includes an alternative full implementation of the Policy interface called StrictPolicy. The StrictPolicy implementation provides a more restrictive Policy than ServerManagedPolicy, in that it does not allow the user to access the application unless a the [ADK](ADK.md) response is received from the server at the time of access that indicates that the user is allowed to access, and because of that will always find an update if there is one available.

The principal feature of StrictPolicy is that it does not store any license response data locally, in a persistent store. Because no data is stored, retry requests are not tracked and cached responses can not be used to fulfill license checks. The Policy allows access only if:

  * The license response is received from the licensing server, and
  * The license response indicates that the user is licensed to access the application.

Using StrictPolicy is appropriate if your primary concern is to ensure that, in all possible cases, no user will be allowed to access the application unless the user is confirmed to be licensed at the time of use. Additionally, the Policy offers slightly more security than ServerManagedPolicy — since there is no data cached locally, there is no way a malicious user could tamper with the cached data and obtain access to the application.

At the same time, this Policy presents a challenge for normal users, since it means that they won't be able to access the application when there is no network (cell or wi-fi) connection available. Another side-effect is that your application will send more license check requests to the server, since using a cached response is not possible.

Overall, this policy represents a tradeoff of some degree of user convenience for absolute security and control over access. Consider the tradeoff carefully before using this Policy.

To use StrictPolicy, simply import it to your Activity, create an instance, and pass a reference to it when constructing your LicenseChecker. See Instantiate LicenseChecker and LicenseCheckerCallback for more information.

### Checking the license from your application's main Activity ###
Once you've implemented a Policy for managing access to your application, the next step is to add a license check to your application, which initiates a query to the licensing server if needed and manages access to the application based on the license response. All of the work of adding the license check and handling the response takes place in your main Activity source file.

To add the license check and handle the response, you must:

  1. Add imports
  1. Implement ADKCallback as a private inner class
  1. Create a Handler for posting from ADKCallback to the UI thread
  1. Instantiate UpdateChecker and ADKCallback
  1. Call check() to initiate the update check
  1. Embed your [AppKey](AppKey.md) for licensing
  1. Call your UpdateChecker's onDestroy() method to close connections.

The sections below describe these tasks.

#### Add imports ####
First, open the class file of the application's main Activity or the [Application object](http://developer.android.com/reference/android/app/Application.html) and import UpdateChecker and ADKCallback from the [ADK](ADK.md) Jar.

```java

import com.leadbulb.adk.UpdateChecker;
import com.leadbulb.adk.ADKCallback;
```

#### Implement ADKCallback as a private inner class ####