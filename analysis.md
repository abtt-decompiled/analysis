# ABTraceTogether Analysis
>  Patrick King, Calgary area Web Developer
>  
>  [@king4nosehill](https://twitter.com/king4nosehill/) / patrick.f.king@gmail.com

Updated May 11 2020

## Background

Friday May 1, the government of Alberta [launched](https://twitter.com/YourAlberta/status/1256345052898627585) [ABTraceTogether](https://www.alberta.ca/ab-trace-together.aspx), a contact tracing app for Android and iOS mobile phones. The app is derived from the [BlueTrace/OpenTrace](https://bluetrace.io/) open source reference apps created by the government of Singapore for use with their own contact tracing program. Since OpenTrace was made available on April 9, governments and health authorities around the world have created their own mobile apps derived from this [codebase](https://github.com/OpenTrace-community).

The app works by transmitting messages via the phone's Bluetooth radio as the user goes about their day, and by listening for transmissions from other phones running the app. Unsurprisingly, the release of mobile apps by governments promising to protect the user's health in exchange for tracking the user's meetings raises concerns on many fronts.

I am a Calgary area freelance web developer with an interest in politics. Mobile development and security research are not my usual areas of work, but the unprecedented haste behind apps like ABTraceTogether, and the potentially enormous consequences for our privacy, health, and rights, prompted me to take a close look at this app. The perspectives and opinions shared here are my own, and are not related to any of my employers.

### [Media Coverage and Other Resources](media.md)


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

## Issues Investigated

There are a lot of issues and concerns with ABTraceTogether, ranging from concerns about its technical implementation and phone security, the possibility of increased data collection, concerns about enabling surveillance of users by third parties, and questions about whether Bluetooth based contact can even be effective or useful.

### 1. Disabled logging features



## Future Topics

I've identified a long list of topics to dig into in more detail, time permitting.

- Bug no tempID rotation if tempID fetch does not succeed
- The Firebase to MobileFirst switch
- IBM MobileFirst Analytics
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
- Version requirement: why Android 8?
- Android Rooms database, root vulnerability, autoupdate vulnerability
- Record deletion policy, post upload
- Detailing the 'collects less information than similar apps' claim
- Issues with tempIDs from a central authority
- Issues with the central authority having the phone number
- Changes from 1.0.0 to 1.0.1
- Difficulties with an eventual Apple/Google upgrade (and confusion in communication on this point)

## Changes

- Monday May 11, first release

































