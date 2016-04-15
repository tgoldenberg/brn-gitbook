# Chapter 2: Getting Started

## Installing React Native

### Basic Requirements

The steps that follow assume you’ve met React Native’s basic requirements, which are:

1. You’re using OS X, since it is (and will definitely continue to be) required for any iOS development.

2. You have [Homebrew](http://brew.sh/) installed (a good idea no matter what). Also run ```brew update && brew upgrade``` to be safe.

3. Node.js, 4.0+ (React Native runs on Node)

4. ```brew install watchman```

5. ```brew install flow``` (optional)

We’re paraphrasing the requirements listed in the official React native [documentation](https://facebook.github.io/react-native/docs/getting-started.html). It’s a good idea to check the details there before getting started.

#### For iOS:

We'll begin our app on iOS since the development workflow is a bit easier. Make sure you have the latest version of [Xcode](https://developer.apple.com/xcode/download/) installed. When Apple rolls out new major releases of Xcode, breaking changes are often introduced. Do yourself a favor and start out with the latest to avoid serious pain later.


## Creating your First Project

Here’s where things get really exciting - we’re going to install React Native on our system and get our first project up and running on iOS.

### Install the React Native Command Line Interface (CLI)

This should be pretty familiar if you’ve used anything like the Amazon or Express.js CLIs:

```npm install -g react-native-cli```

This will install all of the React Native and React packages you need, in addition to installing all of the command line helpers you’ll use for building, running, and building your apps. You only need to do this once per system.

The `-g` flag tells `npm` to install the CLI in our system's global scope, that way we'll be able to access any `react-native-cli` commands from any folder.

### Create a React Native Project

So far, so good. Now let’s get a project up and running. it’s good practice to keep all of your coding projects in one place, like **~/Source**. Navigate to your working directory and create a new project:

```cd ~/Source```

```react-native init assemblies```

In this guide we’ll be building a community app for developers similar to Meetup called "Assemblies," but you can name your project whatever you like.

Now navigate to your new project’s directory:

`cd assembliesTutorial`

## Basic Structure of a React Native Project

Before we test out our new project, let’s explain a little bit about how React Native projects are structured. This will be enormously helpful to understand later, especially when we start working on  both the iOS and Android versions of a project simultaneously.

You should see something like this:
```

android          
index.android.js
index.ios.js     
ios              
node_modules     
package.json
```

`package.json` - works like it does in any other Node project - it’s a Javascript object containing all of the npm package dependencies your project has, as well as basic metadata and any scripts that should be included. Right now it’s pretty unimpressive:

```
{

  "name": "Assemblies",
  "version": "0.0.1",
  "private": true,
    "scripts": {
      "start": "react-native start"
  },
    "dependencies": {
      "react-native": "^0.23.1
    }
}
```

RIght now we only have one package dependency, ```react-native```, at version 0.23.1. It’s important to note this version number at the outset. React Native is incredible, but bleeding edge technology and the codebase is changing constantly. Whenever you’re debugging issues, especially with third-party libraries, conflicts created by differing ```react-native``` version dependencies are often the cause. Try and update at regular intervals, testing in a new branch, so you don’t get too far behind, but play with release candidates at your own risk.

```node_modules```  - contains all of the `npm` packages, React Native core or otherwise, our project will use. 

```index.android.js``` and ```index.ios.js``` - think of these like your `index.html` file in a web project - it’s the starting point for your application, one is for Android, and the other is for iOS.

```android``` and ```ios``` - these folders contain all of the native code the Android and iOS platforms require to run and bundle apps. We’ll introduce different aspects of each directory as we build our project

## Running our Project on iOS


To check out our new project on iOS, simply start Xcode from your "Applications" folder and open the ```assemblies.xcodeproj``` file in your project’s `ios` folder. 

Assuming you have iOS Simulator devices installed, you can simply go to Project > Run in the menu or hit `⌘-R` on your keyboard. The iOS Simulator will open and if everything went according to plan, you should see this:

![alt text](/images/chapter-2-getting-started/beginning-react-native-project-screen-on-ios.png "Beginning React Native project screen on iOS")

You’ll also notice Xcode has opened a Terminal window:

![alt text](/images/chapter-2-getting-started/react-native-packager-terminal-window.png "React native packager Terminal window")

This window holds the React Native transformer’s process, which will interpret all of our Javascript code into native code Xcode can understand. 


## Adding Packages

One of the best things about building apps in React Native is you get access to npm, currently the largest open source library on the planet. We’ll use a lot of different npm packages through this guide for a variety of purposes. 

Installing a new package works just like it does in any other Node project. Make sure you’re in your project’s root folder and run:

```npm install moment --save```

Here we’re installing the incredible Moment.js package which we’ll be using to format dates easily in our project. Adding the ```--save``` flag is essential, as ```npm install``` only installs the package locally - ```--save``` will add the package as a dependency in ```packages.json``` so it will get bundled with our app. 

Opening up `packages.json`, we can see that moment has been added:

```
{

  "name": "Assembly",
  "version": "0.0.1",
  "private": true,
    "scripts": {
      "start": "react-native start"
    },
      "dependencies": {
        "moment": "^2.10.6",
        "react-native": "^0.23.1
      }
}
```



If your prefer not to use the CLI, you can also edit ```packages.json``` directly to accomplish the same thing. 

## Setting Up a Git Repo

Now it’s time to set up a Git repo to version control our project and make undoing mistakes or testing new feature ideas super easy. From your project’s root, run:

```git init```

If you’ve made it this far and understood the instructions, you probably already know hot to use Git. If that’s not the case, or if you just want to improve your Git-fu, these resources will help:

[Atlassian GIt Tutorials](https://www.atlassian.com/git/tutorials/) - far easier to understand than the official Git docs

[Git on Codeacademy](https://www.codecademy.com/learn/learn-git) - everything you’ve come to expect from Codeacademy, straightforward and intuitive

Let's make a commit now

***
<img src='github-logo.png' style='width: 40px;' /> [Commit 1](https://github.com/buildreactnative/assemblies-tutorial/tree/ch-3.0)
***

