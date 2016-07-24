## What is React Native?

[React Native](https://facebook.github.io/react-native/) is a framework developed by Facebook that allows developers to create native apps for iOS and Android using Javascript and [React](https://facebook.github.io/react/).

React Native is the second major implementation of React, which Facebook originally created for developing web interfaces. React is a platform for building UIs, i.e. it's the "V" ("View") in the M-V-C ("Model"-"View"-"Controller") approach to application development. Think of React Native as the same thing as React, except here our target device is iOS or Android, and not the browser.


## How it Works

React allows developers to build UIs using a consistent API that then is interpreted by the library for rendering into browser DOM components. Similarly, React Native provides a thin API that wraps native components in declarative Javascript. Basically, you define your UI using simple components like ```<View />``` and ```<Image />``` then React Native translates them into native code that renders their native representations on iOS or Android.


While the above describes how most developers will experience the technology and what it does, React Native is doing a lot more under the hood to make sure your apps feel completely native.

For one, it keeps the Javascript computational thread separate from the "main" thread your phone uses to run your app. This has several benefits, the most noticeable of which is that the phone never has to wait for Javascript functions to run when the user is interacting with the UI. This makes animations in React Native just as smooth as anything built in Swift or Java, since the phone can't tell the difference. React Native also batches Javascript declaration for performance, so the phone receives commands from the Javascript thread only as often as necessary.


## How does React Native compare to Cordova?

It's easy to assume that React Native is just a better version of [Cordova](https://cordova.apache.org/), an engine developed by Apache that enables developers to write apps in HTML, CSS, and Javascript (also called "hybrid" applications).

Nothing could be further from the truth.

When a phone runs a Cordova app, it's really just running an instance of the phone's web browser in full-screen mode. This is why Cordova apps are written similarly to web applications, since that's essentially what they are. This is also why the transitions and animations in Cordova never feel native, because in fact they aren't. Instead of using any native rendering capabilities of the device, Cordova apps use CSS and Javascript to render their UIs.

Cordova apps also suffer from another major drawback - they are single-threaded. This means that all parts of the application - rendering of colors and layouts, transitions and animations, API calls, state changes, etc. must happen in order, one at a time. This frequently leads to a degraded user experience, especially when data-fetching and transitions need to happen at the same time.

Desktop browsers are able to get away with this because they're generally running on devices with more powerful CPUs and graphics processors than smartphones or tablets, but even they don't compare with the buttery smooth experiences delivered by native apps.

React Native offers the best of both worlds by preserving the native approach to rendering while offering the web-friendly convenience of Javascript APIs and declarative styles. You get all of the performance of a native application without sacrificing the enjoyable experience of developing using Javascript components.

You can watch Tom Occhino of Facebook giving the first public demo of React Native at React.js Conf 2015 [here](https://www.youtube.com/watch?v=KVZ-P-ZI6W4). His talk gives a deeper demonstration of how React Native changes the way we build apps.

## Who is this Guide For?

This guide is targeted primarily at web developers/hybrid app developers who want to build great apps on iOS and Android without compromising user experience.

Everyone is welcome and can derive benefits from *Build React Native*, but it's probably most useful if you:
- Are confident writing Javascript, ideally ES015/ES6 syntax
- Have no problem building web pages using HTML and CSS
- Are familiar with basic responsive web design approaches and principles, since we'll still need to accommodate slightly different device sizes

If you're new to ES6 JavaScript, no worries! We've provided a chapter in the appendix to get you up to speed.

## Who's behind this guide?

This guide is written by Tom Goldenberg and Nicholas Alan Brown, co-founders of [React Native NYC](http://www.meetup.com/React-Native-NYC/), a Meetup in New York dedicated to the technology. We're nearly at 700 members and are one of the fastest growing tech Meetups in the region. We're excited to share what we've learned using the technology ourselves and from working with Meetup members one on one. Here's a little more about each of us:

**Tom Goldenberg** is a web and mobile engineer at [Agolo](http://www.agolo.com/), graduate of [Dev Bootcamp](http://devbootcamp.com/), and Sanskrit enthusiast. He cut his teeth with Ruby on Rails, but has been working almost exclusively with Javascript since it started to get awesome. He currently lives in Long Beach, NY.

**Nicholas Alan Brown** is a product manager at [Huge](http://www.hugeinc.com/), freelance developer, and startup advisor. While a confident developer, his real passion is user experience and product strategy, and he includes app design tips and tricks throughout the guide.

Ready to learn more? Let's get started!
