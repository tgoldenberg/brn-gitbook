# Chapter 3: The Beginnings of an App

## Create an App

1. We're going to start a new React Native project. First create a directory where you will store all folders related to the app -- `mkdir assemblies` - `cd assemblies`

2. After checking to make sure you have all the requirements (see Chapter 2), create your app with the command `react-native init assembliesTutorial`

You should see something like this: ![installing](Screen Shot 2016-04-07 at 7.34.18 PM.png)

3. Once the React Native packager is done installing, you will see a list of the packages installed by `NPM` (in a nested tree structure) and instructions on running the app. ![](Screen Shot 2016-04-07 at 7.37.19 PM.png)

3. Let's run our app! 

Just `cd` into the directory, and type the command `react-native run-ios` to open XCode and run the app on the simulator. After opening another terminal window to run the JS, your app should load on the iOS simulator. (It's a lot of logs to look at the first time, but don't worry). 

Alternatively, you can also open the `.xcodeproj` file in XCode with the command `open ios/assembliesTutorial.xcodeproj`, and then press the `run` button to launch it on the Simulator. I prefer this way, just because it's good to get acquainted with the XCode environment. 

4. Now is a good time for a `git` commit. As we go through the tutorial, we'll provide commits as snapshots of the app at different phases. It's always a good habit to commit early and often. Here's a link to some pointers on good `git` hygiene. 

**** 
<img src="github-logo.png" style="width: 60px;"/> [Commit 1]() - `Initialize React native app`
**** 


## Hot Loading - say what?!?

Thanks to the amazing team of contributors (including @gaeron) to `react-native`, recent versions enable hot loading. This makes development simply a joy -- style changes appear instantly without having to navigate to a particular component. So try it out. Open `index.ios.js` in your favorite text editor (I use `Atom`, `Sublime`, or `Vim`). First make sure LiveReload is enabled by opening up the options panel with `cmd D`. Now change the line `Welcome to React Native` to `Welcome to Assemblies`. Did you see how that changed right away? Amazing, I know :).

<img src="phone-01.png" style="height: 200px; margin: auto;"/>

## Navigator Drama - what should I use?

Next we will look at one of the most important parts of an app - navigation. This has been a topic of concern with React Native since the very beginning, when React Native came with both `NavigatorIOS` and `Navigator`. Facebook uses and maintains the `Navigator` component, but many people still prefer `NavigatorIOS` because it has excellent animation performance. Let's look at the pros and cons of these two options for routing. 

### NavigatorIOS

`NavigatorIOS` has great performance because the animation is handled outside of the main JavaScript thread. I features a simple layout and API to support pushing and popping routes off of the stack. Downsides are that it is highly opinionated and less configurable. It can be appropriate for simple apps or when customization isn't needed.

`Navigator`, on the other hand, is highly customizable. It has options for different sliding and fading transitions, and is neutral in regards to UI. Downsides are that the animation runs on the JavaScript thread, and so, this can cause performance lags. 

We will be using `Navigator` for this project, because it is more widely supported and seen in complex apps. However, to get a feel for NavigatorIOS, let's implement it in a simple two-route app. If you have experience with `NavigatorIOS` or are not interested in seeing what it has to offer, you can skip to the next commit. 

## Using NavigatorIOS - a small example

Now we're ready to start writing some components! First, let's setup our file directory structure. Create a folder in the root level called `application`, and within that, a folder called `components`. There we will create 2 `.js` files, `Landing.js` and `Dashboard.js`. 















