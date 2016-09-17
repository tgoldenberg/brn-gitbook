# Chapter 14: Deploying to the App Store

Great, so we've built all the features we want to build and refactored our code. Now we want to show it to our friends! How can we do that? 

For that, we need to do several things. Our ultimate goal is to have our app available on the App Store. But for that to happen, there are some steps we should take.

1. We need to create an Apple Developer account                   [ ]
2. We need to test on an actual device                            [ ]  
3. We need to add an app icon and splash image                    [ ]
4. We need to share the app with beta testers through TestFlight  [ ]
5. We need to deploy the app to the App Store and get it approved [ ]

# First Steps

One of the best tutorials on creating an Apple Developer account is originally by Gustavo Ambrozio, since edited by Tony Dahbura. The full post is [here](https://www.raywenderlich.com/127936/submit-an-app-part-1). 

Basically, if you want to actually publish apps on the App Store, you have to cough up the $99 to be enrolled in the Apple Developer Program. Now, if you just want to test a device on your phone, you can get the free Apple Developer Account, but we'll be assuming here that you're getting the paid account.

Make sure you have some time to do all this, preferably an entire weekend. There are a lot of hoops to jump through, from setting up your Apple Developer account, to preparing your device and setting up the necessary certificates.

The basic first steps are these:

1. Create a developer account and join the Apple Developer Program
2. Prepare the certificates in Xcode to run your device

## Creating a Developer Account

A lot of this information can be found in the detailed blog post linked above, but we'll be doing a quick overview of the steps. Basically, you have a couple options when creating a developer account. You can:

1. Use your personal Apple ID and make it an individual account
2. Create a new Apple ID and register as a business account

In my own personal case, I registered with my personal Apple ID as an individual. The advantage to this is that it is easy to setup. The disadvantage is that instead of your company's name, your full name will be listed as the app creator. Somehow the aesthetic of that is less pleasing. Also, if you are creating an app with partners, you may want to register as a business account, for tax and distribution purposes. Both approaches are definitely doable. Just make sure you weigh the pros and cons before deciding.

Either way, the tutorial linked to above gives step-by-step instruction on setting up the account. You will have to link your bank account and sign tax agreements if you plan on charging for your app. 

## Registering your App and Device

After you've created the Apple Developer account and paid to join the Developer Program, it's time to provision your device and add some signing certificates. This is Apple's way of ensuring that only Apple-approved apps are available on the App Store. Again, Tony explains step-by-step how to create a certificate on your Mac and include your device in the list of approved devices. Both actions take place in the **Certificates, Identifiers & Profiles** section of the Apple Developer platform ([www.developer.apple.com](www.developer.apple.com)).

To add a certificate to your machine, choose the **Certificates** tab on the left side, choosing **All**. There will be a plus button in the top right corner, and you'll have the option to create a developer or production certificate. Follow the complete instructions in the blog post for getting this set up on your Mac.

![certificates](/images/chapter-15/certificates-1.png)

After installing the device certificate, you will want to add your device as well. Tony covers this as well -- you will want to go to the **Devices** section on the left and follow the instructions for adding your device. 

![device](/images/chapter-15/device-1.png)

## Creating an App ID

The last thing to do before running the app on your device is to create an app ID for your app. To do this we go to the **Identifiers** tab on the left and click the plus button to add a new app ID. When giving an app ID, it will ask for a description of the app, and give the option of using a wildcard ID or an explicit one. It is best to give the explicit one. Apple expects the ID to be ing the form of com.companyName.appName.

The last thing you'll want to do in this portal is to set up what Apple calls your Provisioning Profile. This tab is also on the left side menu, and Tony shows how to easily set this up.

![app-id](/images/chapter-15/app-id-1.png)

Now we can check off one of our boxes!

1. We need to create an Apple Developer account                   [x]
2. We need to test on an actual device                            [ ]  
3. We need to add an app icon and splash image                    [ ]
4. We need to share the app with beta testers through TestFlight  [ ]
5. We need to deploy the app to the App Store and get it approved [ ]


# Testing on Your Device

Now that we've set up an Apple Developer account and handled the necessary logistics in the [www.developer.apple.com](www.developer.apple.com) portal, we can head over to XCode to set ourselves up to test our app locally on our own device. Most of the changes to our actual code take place in a single file: AppDelegate.m

![xcode](/images/chapter-15/xcode-1.png)

Now this is **really** important to understand - the line that defines **jsCodeLocation** determines whether your app will be run from a local Node instance or from a bundled file. The default is to run from a local Node instance. This is the line: 
```objective-c
jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index.ios" fallbackResource:nil];
```

Now, there are 2 ways to run on a local device. The first is to run the local Node instance on our phone, the second is to actually download a bundled file to run on the phone. There are advantages to both:

#### Running a local Node instance

This is much easier in development. You are able to access the Chrome Devtools debugger in this way on your actual phone, which can be quite helpful. Also much easier to make changes and see them after a refresh.

#### Running a bundled file

This is cool because it actually downloads the app onto your phone, and it doesn't depend on anything else. This means that once you have done this, you can continue to test your app on the commute to work, for example. While running a local Node instance can only be run when your phone is connected to your computer, the bundled file can be run anywhere.

### How to run your app on your phone locally

To run the app locally, you will want to connect your iPhone to your Mac. Once it is connected, you should be able to select your device in the top device menu. 

![choose device](/images/chapter-15/choose-device-1.png)

## App Icons

Apple has specific size requirements for app icons and launch images, which can be quite confusing. Luckily, there are some online tools that make it easier. If you have a high-resolution image that you wish to use for your icon or launch screen, you can visit www.makeappicon.com and download a folder with all the different sizes. 
![app-icon](/images/chapter-15/make-app-icon-1.png)


## Setting up the App Icon

## Setting up the Launch Screen


# Deploying Your App to TestFlight

# Deploying Your App to the App Store
