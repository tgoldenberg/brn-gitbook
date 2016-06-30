Mobile apps are comprised of many different parts - navigation, UI components, animations, integrating an external API, etc. We'll be looking first at navigation, which is how we tie different parts of the app together.


## Navigator Drama - Which Should I Use?

Since React Native is a budding technology, it is not always as opinionated as other frameworks. For example, we're given three different options for setting up navigation - `NavigatorIOS`,  `Navigator`, and `NavigationExperimental`.

Let's look at the pros and cons of these three options for routing to see which best suits our needs.

### NavigatorIOS
`NavigatorIOS` offers great performance because the animation displayed as we navigate throughout our app is handled outside of the main JavaScript thread. It features a simple layout and API to support pushing and popping routes off of the stack of views the user accumulates while browsing. `NavigatorIOS` is able to achieve this by very thinly wrapping the native iOS navigation stack in a Javascript component. The downside of this superficial implementation is that it is highly opinionated, tied very closely to iOS default navigation patterns, and therefore less configurable. It can be appropriate for simple apps or when customization isn't needed. It also is not actively maintained by the core React Native team at Facebook.

### Navigator
`Navigator`, on the other hand, is highly customizable. It has options for different sliding and fading transitions, and is completely neutral in regards to UI. The main downside is that the animation runs on the JavaScript thread, and so, this can cause performance lags. In order to keep our app running smoothly, we'll need to add some tweaks to the navigation code.

### NavigationExperimental

`NavigationExperimental` was introduced to offer a single-state approach to navigation. This is because many development teams use the library `redux` to manage their application state. However, learning `redux` with React and React Native can be overwhelming, so we will be sticking with `Navigator`.

Even though `Navigator` and `NavigatorExperimental` are flexible for complex apps, `NavigatorIOS` can be very useful in simple applications. In order to get a feel for NavigatorIOS, let's implement it in a simple two-route app. If you have experience with `NavigatorIOS` or are not interested in seeing what it has to offer, you can skip to the next commit.

## 3.1 Using NavigatorIOS - a Simple Example

Now we're ready to start writing some components! First, let's set up our file directory structure. Create a folder at the root level called `application`, and within that, a folder called `components`. There we will create two `.js` files, `Landing.js` and `Dashboard.js`.

Let's build our Landing component -

```javascript
import React, { Component } from 'react';

import {
  StyleSheet,
  Text,
  TouchableOpacity,
  View
} from 'react-native';

import Dashboard from './Dashboard';

export default class Landing extends Component{
  render(){
    return (
      <View style={styles.container}>
        <Text style={styles.h1}>This is Landing</Text>
        <TouchableOpacity onPress={() => {
          this.props.navigator.push({
            title: 'Dashboard',
            component: Dashboard,
          });
        }}>
          <Text>Go to Dashboard</Text>
        </TouchableOpacity>
      </View>
    );
  }
};

let styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  h1: {
    fontSize: 22,
    fontWeight: 'bold',
    padding: 20,
  },
});
```

And our Dashboard component (with the same styles)-

```javascript
import React, { Component } from 'react';

import {
  StyleSheet,
  Text,
  TouchableOpacity,
  View,
} from 'react-native';

import Landing from './Landing';

export default class Dashboard extends Component{
  render(){
    return (
      <View style={styles.container}>
        <Text style={styles.h1}>This is Dashboard</Text>
        <TouchableOpacity onPress={() => {
          this.props.navigator.push({
            title: 'Landing',
            component: Landing,
          });
        }}>
          <Text >Go to Landing</Text>
        </TouchableOpacity>
      </View>
    );
  }
};

let styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  h1: {
    fontSize: 22,
    fontWeight: 'bold',
    padding: 20,
  },
});
```

Whoa, that was a lot of code! Let's take a minute and look at what we did piece by piece.

We're using Javascript's ES2015 syntax throughout to wire up our views. We start by importing the dependencies of each of our components, which will always include the `import React from 'react-native;'` statement. We also pull in any components we directly reference, like `./Dashboard` or `./Landing` in this case.

Next, we declare any React Native components we'll need in our component. We'll always need `Component`, but you can find a list of available components in the [React Native docs](https://facebook.github.io/react-native/docs/getting-started.html).

Think of `View` like `<div>` in the world of React for Web - it's the basic container for our component and something the packager can bundle and send to the view.

`TouchableOpacity` is a handy component provided by React Native that lets you easily create that "touchable" feel on any elements users will be tapping or pressing. Elements wrapped in `<TouchableOpacity>` will automatically modulate their opacity values when touched. This may seem like a small thing, but it's incredibly noticeable when it's missing, as your elements just sort of respond eventually without any apparent reaction to your touch. Make your interface feel responsive to input is a big part of Apple's [Human Interface Guidelines](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/index.html?utm_source=twitterfeed&utm_medium=twitter).

As we mock up our view, you can see this is just fairly familiar JSX syntax, pretty much XML with a few tweaks.

Specifically, pay attention to how we style our components. We're declaring our styles with Javascript in camel case, referencing keys of a `styles` object we declare and define below. Generally, these styles work pretty closely to CSS, and use flexbox to define containers and layouts. If you're not too familiar with how flexbox works (perhaps you've spent too much time making things backwards compatible for Internet Explorer), we highlight recommend you check out the excellent guide over at [CSS Tricks](https://css-tricks.com/snippets/css/a-guide-to-flexbox/).

*Note: One of the first 'gotchas' to avoid when working with flexbox is making sure you set `flex: 1` on your top-level components so they actually fill the view, otherwise you'll be very confused and frustrated when your app looks completely blank*

Don't worry, we'll cover flexbox and styling in React Native in far more detail as we go.

Now that we have some components, let's connect them in our `index.ios.js` file -


```javascript
import React, { Component } from 'react';

import {
  AppRegistry,
  NavigatorIOS,
  StyleSheet,
} from 'react-native';

import Landing from './application/components/Landing';

class assembliesTutorial extends Component{
  render(){
    return (
      <NavigatorIOS
        style={{flex: 1}}
        barTintColor='#3A7BD2'
        titleTextColor='white'
        tintColor='white'
        shadowHidden={true}
        translucent={false}
        initialRoute={{
          component: Landing,
          title: 'Landing',
        }}
      />
    );
  }
};

AppRegistry.registerComponent('assembliesTutorial', () => assembliesTutorial);
```

This should look pretty familiar, as we're doing many of the same things as in our two previous components. Here we import our dependencies, as well as declare what we need from React Native. Here, `AppRegistry` is simply the global container that will house our app at the top level.

We set up `NavigatorIOS` by giving it it our initial route `Landing`, or the first view that will live at the bottom of our route stack. We then set a few basic options for styling, which you can find in the [React Native docs](https://facebook.github.io/react-native/docs/navigatorios.html#content).

Once we save our code, we should see our two-view app rendered in the iOS Simulator:

![Simple NavigatorIOS Example](/images/chapter-3-the-beginnings-of-an-app/simple-navigatorios-example.png "Simple NavigatorIOS Example")

Take a minute and play with the views, switching back and forth between them. Right out of the gate you can see how much smoother things are than any hybrid app you've ever tried.

If you app is going to be iOS-only (as the name implies, `NavigatorIOS` only works on iOS) and only really involves simple navigation through a few views, by all means, use NavigatorIOS.

We used `NavigatorIOS` for our first app ([Bhagavad Gita](https://itunes.apple.com/us/app/bhagavad-gita-app/id1065731220?mt=8)) partly because we didn't yet know how to handle the animations for Navigator, but also because it had limited navigation needs.

Not to worry - we'll make sure that you don't have that situation. Let's make a commit before moving to `Navigator`.

Let's commit that code now:

****
![GitHub logo](/images/github-logo.png "GitHub logo") 
[Commit 1](https://github.com/buildreactnative/assemblies-tutorial/tree/Ch3-0) - Simple NavigatorIOS example
****


## 3.2 Navigator - A World of Opportunity

Now we'll take our implementation with `NavigatorIOS` and switch it to `Navigator`. You'll notice some differences. For one, `Navigator` doesn't have any interface components out of the box. That's why we'll be using the `react-native-navbar` package by [@kureev](https://github.com/Kureev). We'll also want to install the `react-native-vector-icons` package by [@oblador](https://github.com/oblador) to use cool icons in our navbar. Let's also throw in `underscore` for use later in the tutorial. Type the following in the terminal

```npm install --save react-native-navbar react-native-vector-icons underscore```

This will install the packages to our `node_modules` folder. Now, one issue that React Native developers often face is linking npm libraries to Xcode. Most packages have instructions on how to do this the long way. Fortunately, there is a newer package, `rnpm`, which handles the linking process for us. Just install it with `npm install -g rnpm`, and then run `rnpm link` to link the libraries we installed.

Now we can swap out `NavigatorIOS` for `Navigator` in our `index.ios.js` file. In `Navigator`, we must provide an initial route and a `renderScene` function which acts as a `switch()` statement for all of our main routes. Let's set up the `Navigator` for our two previous components, `Dashboard` and `Landing`.

```javascript
...
import Dashboard from './application/components/Dashboard';


class assembliesTutorial extends Component{
  render(){
    console.log(Navigator.SceneConfigs)
    return (
      <Navigator
        initialRoute={{name: 'Landing', index: 0}}
        renderScene={(route, navigator) => {
          switch(route.name){
            case 'Landing':
              return <Landing navigator={navigator} />
              break;
            case 'Dashboard':
              return <Dashboard navigator={navigator} />
              break;
          }
        }}
        configureScene={() => Navigator.SceneConfigs.PushFromRight}
      />
    )
  }
}
...
```

Notice that the `configureScene` option defines which type of animation our navigation uses to transition between scenes. Feel free to experiment and try other configurations, such as `FloatFromLeft`, `HorizontalSwipeJump`, and `VerticalUpSwipeJump`.

Next we redesign our 2 screens so that they route to each other and also include our navbar with a back icon. Let's look at `Landing.js`

```javascript
...

import NavigationBar from 'react-native-navbar';

export default class Landing extends Component{
  render(){
    return (
      <View style={styles.outerContainer}>
        <NavigationBar
          title={{title: 'Landing', tintColor: 'white'}}
          tintColor='#3A7BD2'
        />
        <View style={styles.container}>
          <Text style={styles.h1}>This is the Landing</Text>
          <TouchableOpacity onPress={() => {
            this.props.navigator.push({
              name: 'Dashboard'
            });
          }}>
            <Text>Go to Dashboard</Text>
          </TouchableOpacity>
        </View>
      </View>
    );
  }
};

let styles = StyleSheet.create({
  outerContainer: {
    flex: 1,
    backgroundColor: 'white'
  },
 ...
```

That should give us our first screen with the navigation bar. If there are errors compiling, it may be that you did not re-build the app after the command `rnpm link`. If so, try pressing the "stop"  button on Xcode and restarting.

For the `Dashboard` component, we'll add an icon on the left of the navbar to `pop()` to the previous route.

```javascript
import React, {
  Component,
} from 'react';

import {
  StyleSheet,
  Text,
  TouchableOpacity,
  View,
} from 'react-native';

import NavigationBar from 'react-native-navbar';
import Icon from 'react-native-vector-icons/Ionicons';
import Landing from './Landing';

export default class Dashboard extends Component{
  _renderBackButton(){
    return (
      <TouchableOpacity
        onPress={() => this.props.navigator.pop()}
        style={styles.backBtn}>
        <Icon name='ios-arrow-back' size={25} color='white' />
      </TouchableOpacity>
    )
  }
  render(){
    return (
      <View style={styles.outerContainer}>
        <NavigationBar
          title={{title: 'Dashboard', tintColor: 'white'}}
          tintColor='#3A7BD2'
          leftButton={this._renderBackButton()}
        />
        <View style={styles.container}>
          <Text style={styles.h1}>This is Dashboard</Text>
          <TouchableOpacity onPress={() => {
              this.props.navigator.push({
                name: 'Landing'
              })
            }}>
            <Text>Go to Landing</Text>
          </TouchableOpacity>
        </View>
      </View>
    )
  }
};

let styles = StyleSheet.create({
  outerContainer: {
    flex: 1,
    backgroundColor: 'white'
  },
  backBtn: {
    paddingTop: 10,
    paddingHorizontal: 20,
  },
  ...
```

![Empty Navigator](/images/chapter-3-the-beginnings-of-an-app/empty-navigator.png "Empty Navigator")

As you can see, the nice thing about `Navigator` is that we can customize how our screen looks at any given time. We can create a navbar with `react-native-navbar` and customize it with icons, or we can set up navigation in a different way. It's worth looking at the different options before deciding what's right for your app.

Okay, now it's time for another commit! Congrats on having delved into navigation with React Native. The `Navigator` API has many more options, some of which we will use in the tutorial. Please check out the [docs](https://facebook.github.io/react-native/docs/navigator.html) for specific API information.

Note: If you are interested in using `NavigationExperimental`, please refer to the appendix. There we show how to structure an app with `NavigationExperimental`, including how to store state with `redux`. Honestly, the transition from `Navigator` to `NavigationExperimental` will feel very easy.

***
![GitHub logo](/images/github-logo.png "GitHub logo") 
[Commit 2](https://github.com/buildreactnative/assemblies-tutorial/tree/ch-3.2) - Create basic navigation with Navigator
***

## 3.3 Fleshing out the App

One important thing to understand about an open source project like React Native is the motivations of its sponsors. Facebook uses React Native currently in two apps - the Ads Manager app for iOS and Android, and partially in the Groups app. If we look at these apps, we can see where React Native's strengths are, and we should be looking to leverage them. This is why we decided to use TabBar navigation in Assemblies. Facebook uses this type of navigation in both apps and more apps are following suit. In this chapter, we will implement a simple TabBar navigation.

The interesting thing about TabBar navigation is that to accomplish it, you often need each tab to have its own `Navigator`. So we end up with `Navigator`s inside of `Navigator`s... It actually creates a nice effect and makes our app easy to get around.

I've gone ahead and wired up a few tabs. In all, there will be five tabs, but for now, I'll start with a Profile view, a Dashboard view, and a Messages view.

Now that we're about to build out the app, there are a couple of things I want to clarify:

*Why are we starting out with fake data?*

This is something were are strong believers in. Fake it and then make it. The wireframing process helps to funnel the idea of the product into a visual representation. A developer's job is to translate that into an actual product. It's very easy to focus on the programming problems like integration with a backend system and server, scalability, and so on. However, most often, the best thing to do at this point is to make a fake product. This helps get the UI components of the product in place so that you can think of the data integration later. This is the process I use and I find it works extremely well. That said, different things work for different people, so sue the method that works for you on your own projects.

*Will you be using an architecture like Flux or Redux to manage state between components?*

We won't be using either in this tutorial. We personally love Redux but even its creator has said that you shouldn't use it until you've felt the pain without it. This app would be a good candidate for an architecture using Redux. However, we were also able to create a good product without it. So while the production version of Assemblies may incorporate Redux, the tutorial itself won't touch on the topic. We address how to structure a React Native app with `redux` in the Appendix section of the tutorial.

### Rounding out our Landing Page

Now we're going to fill in our `Landing` page. Later, this will link to a `login/signup`, but for now we'll have it go directly to the `Dashboard`. Let's place an image as the screen background using the `Dimensions` module. Then let's use the `TouchableOpacity` component as a button that leads to our `Dashboard`. You can download the image assets from the [open-source repository](https://github.com/buildreactnative/assemblies/tree/master/application/assets/images). Create a folder under `application` called `assets`, and an `images` folder in that. That is where we'll store our images for the tutorial.

```javascript
import React, { Component } from 'react';

import {
  Dimensions,
  Image,
  StyleSheet,
  Text,
  TouchableOpacity,
  View
} from 'react-native';

import NavigationBar from 'react-native-navbar';
import Colors from '../styles/colors';

let { width: deviceWidth, height: deviceHeight } = Dimensions.get('window');

export default class Landing extends Component{
  render(){
    return (
        <View style={styles.container}>
          <View style={styles.backgroundHolder}>
            <Image style={styles.image} source={require('../assets/images/welcome.png')}/>
          </View>
          <View style={styles.logoContainer}>
            <Image style={styles.logo} source={require('../assets/images/logo.png')}/>
            <Text style={styles.title}>assemblies</Text>
            <Text style={styles.subTitle}>Where Developers Connect</Text>
          </View>
          <TouchableOpacity
            style={styles.button}
            onPress={() => {
              this.props.navigator.push({
                name: 'Dashboard'
              })
            }}
          >
            <Text style={styles.buttonText}>Go to Dashboard</Text>
          </TouchableOpacity>
        </View>
    )
  }
};

let styles = StyleSheet.create({
  backgroundHolder: {
    position: 'absolute',
    top: 0,
    right: 0,
    bottom: 0,
    left: 0,
  },
  button: {
    height: 80,
    position: 'absolute',
    bottom: 0,
    left: 0,
    right: 0,
    flexDirection: 'row',
    backgroundColor: Colors.brandPrimary,
    justifyContent: 'center',
    alignItems: 'center',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: '700',
  },
  container: {
    flex: 1,
    position: 'absolute',
    top: 0,
    right: 0,
    bottom: 0,
    left: 0,
  },
  image: {
    height: deviceHeight,
    width: deviceWidth,
  },
  logo: {
    height: 90,
    width: 90,
  },
  logoContainer: {
    flex: 1,
    backgroundColor: 'rgba(0, 0, 0, 0)',
    justifyContent: 'center',
    alignItems: 'center',
    marginBottom: 50,
  },
  subTitle: {
    color: 'white',
    fontSize: 20,
  },
  title: {
    color: 'white',
    fontSize: 28,
    fontWeight: '700',
    paddingBottom: 24,
  },
});

```

You'll notice that we reference `Colors` from a separate file now, and that some images are referenced. You can download the `logo.png` and `welcome.png` images by following the commit link below and place them in an `assets/images` folder under `application`. The `colors.js` file can be placed in `styles/` under `application`. So far it's just this:
```javascript
export default Colors = {
  brandPrimary: '#3A7BD2'
};
```

![Landing Screen](/images/chapter-3-the-beginnings-of-an-app/landing-screen.png "Landing Screen")

As for the styles, some stuff should be pretty self-explanatory for those familiar with styling with CSS on the web. You'll see that we use some absolute positioning as well here, but the majority of components typically use flexbox for positioning.

Time for another commit to close out this chapter on setting up navigation:

***
![GitHub logo](/images/github-logo.png "GitHub logo") 
[Commit 3](https://github.com/buildreactnative/assemblies-tutorial/tree/ch-3.3) - Flesh out landing page
***
