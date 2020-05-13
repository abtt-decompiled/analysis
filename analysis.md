# ABTraceTogether Analysis
>  Patrick King, Calgary area Web Developer
>  
>  [@king4nosehill](https://twitter.com/king4nosehill/) / patrick.f.king@gmail.com

Updated Tuesday May 12, 2020


## Introduction

COVID contact tracing apps are proliferating around the world, and Canada is no exception. In countries like Australia and the UK, vigorous debates have emerged about the functionality and privacy implications of these apps. Certainly the current moment is unprecedented: never before have so many states offered apps to their citizens with the capability to track the user's close contacts, and share this information with the government. 

My impression is that Alberta and Canada's share of this debate has been lacking, and I believe it's vital to scrutinize these tracking apps' implementation, privacy implications, potential effectiveness, and their surrounding clarity of communications.

I am a Calgary area freelance web developer with an interest in politics. Mobile development and security research are not my usual areas of work, but the unprecedented haste behind apps like ABTraceTogether, and the potentially enormous consequences for our privacy, health, and rights, prompted me to take a closer look. The perspectives and opinions shared here are my own, and are not related to any of my employers.

My goal with this project is to create a single point of reference for the recently released ABTraceTogether contact tracing application, operated by the provincial government of Alberta, Canada.

Are you interested in helping out? Do you have development (especially mobile and Bluetooth development), IT, or information security skills? Feel free to drop me a line on Twitter or by email.

### [Media Coverage and Other Resources](media.md)

## ABTraceTogether Background

Friday May 1, the government of Alberta [launched](https://twitter.com/YourAlberta/status/1256345052898627585) [ABTraceTogether](https://www.alberta.ca/ab-trace-together.aspx), a contact tracing app for Android and iOS mobile phones. The app is derived from the [BlueTrace/OpenTrace](https://bluetrace.io/) open source reference apps created by the government of Singapore for use with their own contact tracing program. Since OpenTrace was made available on April 9, governments and health authorities around the world have created their own mobile apps derived from this [codebase](https://github.com/OpenTrace-community).

The app works by transmitting messages via the phone's Bluetooth radio as the user goes about their day, and by listening for transmissions from other phones running the app. Unsurprisingly, the release of mobile apps by governments promising to protect the user's health in exchange for tracking the user's meetings raises concerns on many fronts.


## How BlueTrace Apps Work

Here's how apps built on BlueTrace technology are intended to work:

- When the user runs the app for the first time, they are prompted to register their phone number with the health authority, and are sent a code by text message to be entered into the app to complete setup.
- Periodically, the app will download temporary IDs (tempIDs) from the health authority server. These IDs contain encrypted information identifying the user, but are unreadable to anyone but the health authority. The user (and anyone else who might obtain a tempID) cannot read them. Some implementations download one batch of tempIDs daily and cycle through them, some download a new tempID as needed every few hours.
- During the day, the app will try to connect to other instances of the app on other nearby phones via Bluetooth. If a connection succeeds, the apps exchange some data: their current tempIDs, their phone models, and the measured strength of the connection (called an RSSI, a received signal strength indication).
- The data resulting from this exchange is stored in a database on the phone. It is only transmitted to the health authority if and when the user takes action to do so (after being notified of a positive COVID-19 test).
- This local database of exchanges is not stored on the phone forever. Each day, exchange records which are older than a defined limit (usually older than 21 days) are deleted.
- The tempID used for exchanges is rotated regularly by the app. In theory, the fact that each tempID is only transmitted for a short period should make it difficult to track any one user over time. Some implementations rotate the tempID every 15 minutes, some rotate every few hours.
- If a user tests positive for COVID-19 and they have the app, the health authority will ask them to upload their database of tempID exchanges. Doing so is voluntary in Alberta, but sharing data is mandatory in some jurisdictions. These contacts would form the basis of discussion with a contact tracer, combined with the user's own memory of recent travels, to determine which contacts may have been exposed.
- All tempIDs are recorded on the health authority's server with their user's phone number. Once the contact tracer decides that a contact might have been exposed and should be followed up with, their tempID is used to look up their phone number, and that person can be called.

A few other notes about how I expect ABTraceTogether to behave:

- The app should not access the phone's GPS features at any time.
- The app should only upload the database of contacts on the user's request.
- The app should share limited, if any, other data with the health authority. It's common for commercial apps to come with analytics packages, which can and do share everything they can access about the user with the app's operator. This sort of activity should not be observed here.

The [BlueTrace whitepaper](https://bluetrace.io/static/bluetrace_whitepaper-938063656596c104632def383eb33b3c.pdf) covers these mechanisms in a bit more detail. Note that OpenTrace and its derived apps are a separate initiative from the [Apple/Google partnership](https://www.apple.com/covid19/contacttracing/) to add Bluetooth based exposure notification features to their mobile operating systems. There are many similarities in approach between the two projects, and in the near future many tracing apps will be updated to use these new features, but the Apple/Google work has not been released yet.

## Source Code and Dissassembled Code

Apps begin life as a folder full of text files written in a programming language, the *source code*. The source has two major jobs to do: it has to describe the operation of the app in perfect detail, and it also has to be readable and understandable to human maintainers into the future. Many developers and corporations choose to release the source code behind their apps, making them *open source*. With source code in hand, a developer can verify that the app works as expected, compile the source into a fresh copy of the app, and create variants of the app for their own purposes.

However, even when the source code is not available, it's often possible to examine the inner workings of an app by decompiling the app's files, producing *decompiled code*. This code is typically missing many of the details added to the source specifically to make it readable and understandable, but with patience a great deal can be learned from decompiled code. When the source code is available, a developer can try to check that the app in an app store matches the source code, by comparing the app's decompiled code to the source code.

At this writing, the government of Alberta has pledged to release the source code for ABTraceTogether soon, and has not done so yet. But, ABTraceTogether is a modified version of OpenTrace, which does have published source code. This examination is based on the decompiled code of ABTraceTogether, comparing it with OpenTrace's source code.

## Issues Investigated (ABTraceTogether Android app)

In the issues I've looked at below, there are many links to code repositories for [decompiled ABTraceTogether code](https://github.com/abtt-decompiled/abtracetogether_1.0.1.apk_disassembled) and for the [OpenTrace Android](https://github.com/abtt-decompiled/opentrace-android) app.


### The Firebase to MobileFirst switch

The OpenTrace reference app uses Firebase, a Google-owned cloud communication and database technology, to handle tempIDs and user phone number registration. The Firebase client is completely removed from ABTraceTogether, and [in its place](https://github.com/abtt-decompiled/abtracetogether_1.0.1.apk_disassembled/blob/c996e20b180a9dfbfb3addd10511af76ce8ae1ed/sources%5Cca%5Calbertahealthservices%5Ccontacttracing%5Capi%5Cauth%5CAuthRequests.java#L3) is an IBM platform called [MobileFirst](https://mobilefirstplatform.ibmcloud.com/) (formerly branded as WorkLight). The server component of MobileFirst can be installed on premises, and I assume that a major reason for the Firebase to MobileFirst switch is so that the app's servers can be operated by AHS or the government of Alberta, and not by a public cloud provider.

### MobileFirst Analytics

MobileFirst has an integrated [analytics module](https://mobilefirstplatform.ibmcloud.com/tutorials/en/foundation/8.0/analytics/). This analytics system can collect data in [four categories](http://mobilefirstplatform.ibmcloud.com/tutorials/en/foundation/8.0/analytics/analytics-api/), including app lifecycle events, network usage, user information, and custom events. The app itself only enables one of these categories: [lifecycle events](https://github.com/abtt-decompiled/abtracetogether_1.0.1.apk_disassembled/blob/c996e20b180a9dfbfb3addd10511af76ce8ae1ed/sources%5Cca%5Calbertahealthservices%5Ccontacttracing%5CTracerApp.java#L103), which is documented as including "app usage rates, usage duration, app crash rates." 

This data collection should be minimal, and is consistent with the [FAQ](https://www.alberta.ca/ab-trace-together-faq.aspx) and [privacy policy](https://www.alberta.ca/ab-trace-together-privacy.aspx). Worth noting though: additional analytics capabilities could be enabled in an update by adding one or two lines of code.


### Android 8+ Required

OpenTrace is designed with the relatively low version requirement of [Android 5.1 (API level 22)](https://github.com/abtt-decompiled/opentrace-android/blob/master/app/build.gradle#L36). ABTraceTogether has a much higher requirement of [Android 8 (API level 26)](https://github.com/abtt-decompiled/abtracetogether_1.0.1.apk_disassembled/blob/c996e20b180a9dfbfb3addd10511af76ce8ae1ed/resources%5CAndroidManifest.xml#L3). It's not immediately clear to me why this was necessary, as the major dependency switch to MobileFirst appears to be compatible with Android versions [back to Android 6 (API level 23)](https://mobilefirstplatform.ibmcloud.com/blog/2019/09/04/mobilefirst-android-Q/), which is the oldest target version that Google will still allow into the Play store. Numerous [reviews](https://play.google.com/store/apps/details?id=ca.albertahealthservices.contacttracing&hl=en_US&showAllReviews=true) on the Play store reference how relatively new devices are not supported by the app. Given the stated goal of wide adoption of the app, making technology choices that exclude Android 6 and 7, which make up [36% of Android phones](https://www.androidauthority.com/android-version-distribution-748439/), seems like a mistake.

### Logging features

ABTraceTogether comes with two distinct logging systems.

The first system is a centralized [logging module](https://github.com/abtt-decompiled/abtracetogether_1.0.0.apk_disassembled/blob/master/sources%5Cca%5Calbertahealthservices%5Ccontacttracing%5Clogging%5CCentralLog.java), which is essentially identical to the [logger in OpenTrace](https://github.com/abtt-decompiled/opentrace-android/blob/7e0cdd8f52f4560c4ed9bb37d55a0f820d1cb0ae/app/src/main/java/io/bluetrace/opentrace/logging/CentralLog.kt). This tool appears to be intended as an aid to development only, and this [logging](https://github.com/abtt-decompiled/abtracetogether_1.0.0.apk_disassembled/blob/d81b0ee1490911104ab36056d220be7fe3a19e1c/sources%5Cca%5Calbertahealthservices%5Ccontacttracing%5Clogging%5CCentralLog.java#L19) is [disabled](https://github.com/abtt-decompiled/opentrace-android/blob/7e0cdd8f52f4560c4ed9bb37d55a0f820d1cb0ae/app/src/main/java/io/bluetrace/opentrace/logging/CentralLog.kt#L19) for general distribution versions of the app. A search of the codebase turns up hundreds of logged events, covering everything from low level Bluetooth events to user interactions to scheduled app activities (scanning, purging old data, etc). Worth noting, that since this logging infrastructure is already in place, all of the events which are logged could easily be stored or transmitted by an updated version of the app with minimal code changes.

The second system uses [MobileFirst's logger](https://github.com/abtt-decompiled/abtracetogether_1.0.1.apk_disassembled/blob/master/sources%5Cca%5Calbertahealthservices%5Ccontacttracing%5Clogging%5CWFLog.java). This system is configured to [send logged events](https://mobilefirstplatform.ibmcloud.com/tutorials/en/foundation/8.0/application-development/client-side-log-collection/android/) to the server by default, and is used in ABTraceTogether to report application errors related to communication with the server (including authentication failures, and tempID request failures).

- Logger initialization in [TracerApp](https://github.com/abtt-decompiled/abtracetogether_1.0.1.apk_disassembled/blob/c996e20b180a9dfbfb3addd10511af76ce8ae1ed/sources%5Cca%5Calbertahealthservices%5Ccontacttracing%5CTracerApp.java#L104)
- Logger convenience class in [WFLog](https://github.com/abtt-decompiled/abtracetogether_1.0.1.apk_disassembled/blob/master/sources%5Cca%5Calbertahealthservices%5Ccontacttracing%5Clogging%5CWFLog.java)
- Uses
  * Login check error reporting in [BluetoothMonitoringService](https://github.com/abtt-decompiled/abtracetogether_1.0.1.apk_disassembled/blob/c996e20b180a9dfbfb3addd10511af76ce8ae1ed/sources%5Cca%5Calbertahealthservices%5Ccontacttracing%5Cservices%5CBluetoothMonitoringService.java#L947)
  * Errors requesting new tempIDs in [TempIDManager](https://github.com/abtt-decompiled/abtracetogether_1.0.1.apk_disassembled/blob/master/sources%5Cca%5Calbertahealthservices%5Ccontacttracing%5Cidmanager%5CTempIDManager$getTemporaryIDs$1.java)
  * Network errors in [Request](https://github.com/abtt-decompiled/abtracetogether_1.0.1.apk_disassembled/blob/c996e20b180a9dfbfb3addd10511af76ce8ae1ed/sources%5Cca%5Calbertahealthservices%5Ccontacttracing%5Capi%5CRequest%24runRequest%242%24listener%241.java#L62)
  * Authentication error in [AuthRequest](https://github.com/abtt-decompiled/abtracetogether_1.0.1.apk_disassembled/blob/c996e20b180a9dfbfb3addd10511af76ce8ae1ed/sources%5Cca%5Calbertahealthservices%5Ccontacttracing%5Capi%5Cauth%5CAuthRequests%24obtainAccessToken%242%241.java#L36)



## App Updates

### 1.0.0 to 1.0.1

Version 1.0.1 of ABTraceTogether launched on the Play store on May 7, 2020. ([Diff](1.0.0_1.0.1.diff)) This version: 

- Adds a new user interface for error handling, possibly for if something goes wrong with the signup process
- Makes some small interface fixes
- Adds some clarifications to instructions for when a user should upload their data to AHS
- Adds a number of odd checks in user interface files, for interface elements which should probably never go missing

There are no major functionality changes, all of the updates appear to be oriented toward improving app reliability.



## Future Topics

I've identified a list of topics to dig into in more detail, time permitting.

- Bug no tempID rotation if tempID fetch does not succeed
- 21 day deletion window (changes to system time can delete)
- Device model name in Bluetooth exchanges
- Tempid fetch timing (Batched or continuous?)
- Tempid duration (15m? 2h?)
- Old TempID reuse
- MobileFirst server location (US CLOUD act implications)
- Android fine location permission + automatic updates
- Breaks Bluetooth MAC address randomization
- False positives: rooms and cars
- False negatives: brief contacts
- Potential to enable surveillance: malls/shopping centres + at home
- Bluetooth RCE vulnerabilities
- Android Rooms database, root vulnerability, autoupdate vulnerability
- Record deletion policy, post upload
- Detailing the 'collects less information than similar apps' claim
- Issues with tempIDs from a central authority
- Issues with the central authority having the phone number
- Difficulties with an eventual Apple/Google upgrade (and confusion in communication on this point)
- Sunset clause

## Changes

- Tuesday May 12: add sections for logging, 1.0.1 changes, Android 8, MobileFirst and analytics, new introduction
- Monday May 11: first release

































