---
layout: post
title: Failed to install WatchKit App, error&#58; Application Verification Failed 
---

![WatchKit Error](http://i.imgur.com/utV57Ji.png)

WatchKit hasn't been around for long and as is to be expected documentation is a scarce resource. Which has been both an issue and a blessing in disguise forcing me to dig deeper than usual and learn something I otherwise wouldn't.

I had an issue earlier with packaging an iOS + WatchKit bundle into an `.ipa` using `xcrun` instead of Xcode's Archive and Export functionality. Chances are you'll come across this issue. Installing your WatchKit app on your Apple Watch device fails with the following error message:

```
'Failed to install WatchKit App, error: 
Application Verification Failed'. 
```

You've got no stack trace and no console logs... so this could be one of many different things.

In my case this specific project involved a build pipeline that Archived, Code Signed and Exported the app automatically using  a few fancy build scripts. This seems to be a common process allowing teams to have a deeper control over their Continuous Integration setup.

####xcode-build
To accomplish this `xcode-build` is used to compile the Xcode project into an output `.app` file. You cannot distribute this `.app` file as it does not contain any Provisioning Profiles or Developer Certificates.

####xcrun
Once an Xcode project is compiled into an `.app` file, `xcrun` is used to package it into an `.ipa` file. This is the file which includes your `.app` as well as Provisioning Profiles and Developer Certificates and is the package to be installed on a user's iOS device. 

##WatchKitSupport/WK Required
Until now I did not realise that an `.ipa` package has an inherent internal structure that must be adhered to at all times:

```
/Payload/
/Payload/Application.app
/WatchKitSupport/WK
```

####/Payload/
Contains your `.app` file, which itself contains all your iOS applications assets, .xibs, .plists.

####/Payload/MyApp.app/Plugins
Contains a `MyApp_WatchKitExtension.appex` file, which itself contains all your WatchKit Extension resources.

####/Payload/MyApp.app/Plugins/MyApp_WatchKitExtension.appex/
Contains a `MyApp_WatchKitApp.app` file, which itself contains all your WatchKit App (not Extension) related files such as Storyboards, Assets and everything living on the Watch device itself as opposed to the Extension. It also contains the Watch Extension executable.

####/WatchKitSupport/
Undocumented. Apps supporting a WatchKit App require this WatchKitSupport directory and within it a `WK` binary file.

##xcrun vs Xcode
Knowing this I compared the output of xcrun with the output of Xcode's Archive and Export functionality. 

####Xcode
```
/Payload/
/Payload/Application.app
/WatchKitSupport/WK
```

####xcrun
```
/Payload/
/Payload/Application.app
```

Then I discovered this forum post by an Apple Engineer in the [Developer Forums](https://devforums.apple.com/message/1119973#1119973), inadvertently describing the inability for xcrun to support packaging an `.ipa` for WatchKit support. His workaround and [Kassem Wridan's](http://www.matrixprojects.net/p/watchkit-command-line-builds) workaround seem to be the only solutions to this issue.

{% gist phillfarrugia/5b0c892cf4b625c90bf7 %}

This is definitely not a permanent solution to the problem but is a perfect opportunity for me to [file a radar](https://openradar.appspot.com/radar?id=5021668984487936). If you're reading this you should do the same in the hopes this will one day be fixed.

If you have any questions, or corrections feel free to [let me know](./about/).


