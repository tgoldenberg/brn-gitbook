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

From here, if you build the app (without changing any of the lines in your AppDelegate.m file, it should build a version on your iPhone. Shaking the phone will bring up the developer menu.

### Why is my app much slower?

If you're wondering why your app runs slower on your actual iPhone than in the Simulator, it could be because of unnecessary **console.log** statements. That is why we adding a configuration file that turns off **console.log** statements when in mode **DEV=false**. 

Also, the **Navigator** transitions could be slow if you are not making good use of the **InteractionsManager**. We show how to use this to your advantage in order to maintain smooth scene transitions in the tutorial.

### How to run your app with a bundle

Although not very well documented, React Native provides a **bundle** command in its command line tool. This command expects the following arguments
* **--entry-file** the file where your app starts (**index.ios.js**)
* **--platform** the platform compiling for (**ios** or **android**)
* **--dev** development mode (**true** or **false**)
* **--bundle-output** where you want to store the final bundle (**ios/main.jsbundle**)
* **--assets-dest** where you would like the images, etc., store (**ios**)

While there are other arguments the **react-native bundle** command accepts, these are the main ones we will be utilizing.

Now run the command with arguments we specified. The command should look like this:

```
react-native bundle --entry-file index.ios.js --platform ios --dev false --bundle-output ios/main.jsbundle --assets-dest ios
```

![bundle](/images/chapter-15/bundle-1.png)

You should see output showing the app being bundled, and then a notification that it was successful. After this, we have to configure our **AppDelegate.m** file to run the bundle instead of the local Node instance. 

You will want to comment out the previous definition of **jsCodeLocation** and replace it with the following:

```
NSURL *jsCodeLocation;
  jsCodeLocation = [[NSBundle mainBundle] URLForResource:@"main" withExtension:@"jsbundle"];

//  jsCodeLocation = [[RCTBundleURLProvider sharedSettings] jsBundleURLForBundleRoot:@"index.ios" fallbackResource:nil];

```

This tells Xcode that we want to run our app from the **main.jsbundle** file instead of the local file directory. The app should run exactly the same, except it will say **Loading from pre-bundled file** when the app is refreshed. Notice that it doesn't depend on any local Node processes as well!

Once you successfully bundle in this way and install on your device, you can test the app anywhere you go. This is also the biggest step to deployment, because you have verified that the bundle works properly. Everything else if just submitting your bundle and distributing it to other users!

## App Icons

Apple has specific size requirements for app icons and launch images, which can be quite confusing. Luckily, there are some online tools that make it easier. If you have a high-resolution image that you wish to use for your icon or launch screen, you can visit [www.makeappicon.com](www.makeappicon.com) and download a folder with all the different sizes. 
![app-icon](/images/chapter-15/make-app-icon-1.png)

Notice that if your app lacks app icon images, you will not be able to upload it to the App Store. While this process can be confusing, Xcode will give warning on the left panel if your images are the wrong size.

For editing image sizes, it is good to have one image in a very large size, and then make copies, editing the size in a program like **Sketch** or even in your Mac's **Preview** application. For a list of required image sizes, there are [resources](https://developer.apple.com/library/content/qa/qa1686/_index.html) on Apple's Q&A forum. 

## Setting up the Launch Screen

While setting up a launch screen isn't absolutely essential to uploading your app, it is recommended. From the start, React Native provides us with a basic launch screen, but you may want to follow the instructions in Apple's developer [documentation](https://developer.apple.com/ios/human-interface-guidelines/graphics/launch-screen/) to add your own custom launch screen images. 

Basically, you will want to add a **new iOS Launch Image** from the **Images.xcassets** directory, and fill it with images of the proper sizes. Then you will want to **General > App Icons and Launch Images** and add your Launch Screen folder to the **Launch Images Source** option. You will also want to remove the default **LaunchImage.xib** file provided by React Native and clean/rebuild your build to test the changes. 

![launch image](/images/chapter-15/launch-image-1.png)
![launch image](/images/chapter-15/launch-image-2.png)
![launch image](/images/chapter-15/launch-image-3.png)
# Deploying Your App to TestFlight

From here, it gets relatively easy. Please do not skip to this step until you have done all of the above. The reasons for this:
* if you do not have an app icon you cannot upload your app
* if you have not properly bundled and successfully ran your app from a bundle you may be wasting lots of time (and may only realize the error after waiting a week to hear your submission response).

Now that we've got all that taken care of, here are some things we will have to be aware of when uploading our app

1. We will need to download the **Application Loader** application. This makes uploading our binaries much easier and reliable, at least in my opinion.
2. We will need to make sure our app is linked to the **App ID** we created earlier and that our Apple ID account is correctly linked
3. We will need to manage the version numbers of each upload. Uploading an app with the same version number will cause an error and will fail.

That said, let's get started!

#### Linking to our App ID

In **General > Identity**, there are three fields we need to ensure are correct. The first is **Bundle Identifier**. This should be exactly the same as the app ID that we created in the itunes developer portal. The second is the **Team** field. This should be your Apple developer account (you may have to add it here, by clicking **add**). Last is the **Version**. As mentioned earlier, this adhere to **semver** rules, meaning **Number dot number dot number**. You want to bump up the last number after each upload, and change the major numbers as needed.

![deploying](/images/chapter-15/deploying-1.png)

It's also useful to modify the settings right below this, under **General > Deployment Info**. Here you can set your **Deployment Target** (the earliest iOS you will support), **Device Orientation** (usually useful to only have **Portrait** on, and **Status Bar Style** (useful to have as **Light** if shown over a dark navigation bar, as in our case).

### Creating a Build

Once we have ensured that our app is linked to the proper app ID and Apple ID, we can create a build. To do this, first select **Generic iOS Device** under the top device selection button. Then under the **Product** top tab, select **Archive**.

This will then create a build, and show the build in a separate window. Here, we will need to do a few things:

* Validate the build
* Export the build
* Upload via the Application Loader

First let's validate the build, by clicking **Validate**. This will first ask us to select an Apple account. Then it will show a screen that asks us to **Validate**. Here, it is recommeded to unclick the checkbox that says **Include bitcode**. This is a small modification that sometimes speeds up the upload time.

Now click **Validate**, and you should get a success message.

![validate](/images/chapter-15/validate-1.png)

After that is done, you should click the **Export** button, and basically follow the same steps as the **Validate** process. Make sure you choose the default, to upload for iOS deployment. When this is over, it will save a file to your Desktop with the app name and timestamp (ex. **assembliesTutorial 2016-09-17 12-50-03**). Inside this folder will be your build to submit (ex. **assembliesTutorial.ipa**). 

![build](/images/chapter-15/app-build-1.png)

From here, you should open the **Application Loader** application. Select **Deliver Your App**, and then choose the **.ipa** file you just created. The build information should then display, and you can select **Next** to upload it. Next it should take some time to upload the app, and then (hopefully) deliver a success message.

![application loader](/images/chapter-15/application-loader-success-1.png)

#### What If My Upload Fails?

If your build failed to upload, you will usually receive an error message. The most common one is that there is already a build with the same versioning (as warned above). If your build fails for some unknown reason, you are probably best bumping the **Version** number up and creating a new build to upload.

### Accessing Your Build on iTunes Connect

From the time that your app successfully uploads from **Application Loader**, it will usually take about 15 minutes for it to be available on iTunes Connect. Access your account on [www.itunesconnect.apple.com](www.itunesconnect.apple.com). From here, go to **My Apps**, and click the **+** icon to add a new app. Add your app ID as **Bundle ID** and **SKU** to get started.

From here, the main options are **App Information** (basic info, description about your app), **Pricing and Availability** (in which countries it is available, etc.), and **Prepare for Submission**. The last option is where we will eventually add our description, screenshots, app icon, and build. 

For now, go to the **TestFlight** option above. First you will want to select **Internal Testing**. Add your internal testers (your team), and select a version to test. As mentioned, you may have to wait a while for your build to become ready. Once selected, you can click **Save** and your internal testers will get a notification from Apple.

From the email notification, you can access a special code and enter the code in the **TestFlight** iOS app. This will give you access to download the build and run it on your device.

### External Testers

Once you've tested the app internally, the next step is to submit a build for external testers. This step has a little more scrutiny, and usually requires 1-2 days for approval. You will have to specify what your users should be testing for and provide contact information. Once submitted, you will be notified when the build is approved, and the same process as internal testing will take place.

### Benefits of TestFlight

TestFlight is a great tool once you get to know it. It enables you to get feedback from up to 2,000 users before unleashing your new app to the public. The internal testing tool is a great way to share updates across a dev team and actually test the app on multiple devices (something that is important for design purposes, among other reasons). We recommend that you at least go through internal testing before submitting a build to the App Store. Always better to get feedback from friends and family before getting bad reviews from paying customers.

# Deploying Your App to the App Store

This part should be pretty easy if you've followed the previous steps. We have already created a successful build and tested it on our own device, as well as sharing the build with our team and friends. Now it's time to submit that build to the App Store!

There are several things we will need to do this.

* A privacy url - you can host a static page on Github pages for this and follow other similar privacy pages
* screenshots - we can get those with the command **cmd + s** on the Xcode Simulator. Make sure to take 3-4 relevant screenshots on iPhone 6s Simulator.
* large app icon - just use the same app icon we used before but with the specified measurements
* submit a rating for the app - this is straightforward, just check the selections for sensitive content

Once you submit your app, the status will change to **Waiting for Review**. About 5-7 days later, the status will change to either **Approved** or will fail, in which case you should investigate any errors that are given.

### Congratulations!

If you've made it this far, congrats! The App Store deployment process is a painful process, requiring many steps that can be confusing, from creating the proper type of account to loading your final bundle. We hope that this overview may have saved you some precious time! Good luck!




