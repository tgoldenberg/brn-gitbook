## Chapter 3: The Beginnings of an App

Mobile apps are comprised of many different parts - navigation, UI components, animations, integrating an external API, etc. We'll be looking first at navigation, which is how we tie different parts of the app together.


### Navigator Drama - Which Should I Use?

Since React Native is a budding technology, it is not always as opinionated as other frameworks. For example, we're given three different options for setting up navigation - **NavigatorIOS**,  **Navigator**, and **NavigationExperimental**. Since NavigationExperimental is as it is described, experimental, and NavigatorIOS is no longer maintained by Facebook, we'll be using [**Navigator**](https://facebook.github.io/react-native/docs/navigator.html) for this tutorial. We will, however, look more at NavigationExperimental with [**Redux**](https://github.com/reactjs/redux) in the later chapters of the guide.


### Why Navigator?
**Navigator**, unlike **NavigatorIOS**, is highly customizable. It has options for different sliding and fading transitions, and is completely neutral in regards to UI. The only downside is that navigation animations run on the JavaScript thread, and this can cause performance lags. We'll show how to tweak the navigation code in order to keep our app running smoothly.


### A Simple Example

Now we're ready to start writing some components! First, let's set up our file directory structure. Create a folder at the root level called **application**, and within that, a folder called **components** and a folder called **styles**. Within **styles** create the two files  `index.js` and `colors.js`, and within **components** create the files `Dashboard.js` and `Landing.js`. Your folder tree should look something like this.

```
android
application
  - components
    - Dashboard.js
    - Landing.js
  - styles
    - colors.js
    - index.js
```

While we will have a chapter later on how to style and design your app in React Native, it will make the code samples much simpler if we can simply import our styles from a single file. This also makes our design code more modular. 

Copy/paste the contents at [this gist](https://gist.github.com/tgoldenberg/5e2f7be3d3f4f97302e2d7545063b3c9) and paste them in `application/styles/index.js`.

Now copy/paste the contents at [this gist](https://gist.github.com/tgoldenberg/b024fd60ad6fab148bdcd2b039eac5c9) and past them in `application/styles/colors.js`.

Before we build our first component, a warning that we will be using [**ES6**](http://es6-features.org/) syntax. If you're unfamiliar with using features such as `import`, destructuring, the spread operator, and the fat arrow function syntax, we recommend first completing the ES6 chapter in the appendix. Here are some other resources to get you up to speed: 

- [Tutorial by Mantra](https://tutor.mantrajs.com/say-hello-to-ES2015/introduction)
- [learnharmony.org](http://learnharmony.org/)
- [Video by Sencha](https://www.youtube.com/watch?v=Z7yS28I5ci4)

Since **Navigator** is not opinionated in terms of UI, we will need to `npm install` the following packages:
  - [**react-native-navbar**](https://github.com/react-native-community/react-native-navbar) by @react-native-community
  - [**react-native-vector-icons**](https://github.com/oblador/react-native-vector-icons) by @oblador

```
npm install --save react-native-vector-icons react-native-navbar
```

We'll also need to link the icon package to XCode with [**rnpm**](https://github.com/rnpm/rnpm), a package manager built specifically for React Native. First install `rnpm` globally

```
npm install -g rnpm
```
Then link the packages with the command `rnpm link`. You should get a success message.

![rnpm](/images/chapter-3/rnpm-1.png)

Once the packages are linked we can build our Landing component.

```javascript
/* application/components/Landings.js */

import React, { Component } from 'react';
import {
  View,
  Text,
  TouchableOpacity
 } from 'react-native';

import Icon from 'react-native-vector-icons/Ionicons';
import NavigationBar from 'react-native-navbar';
import Colors from '../styles/colors';
import { globals } from '../styles';

class Landing extends Component{
  constructor(){
    super();
    this.visitDashboard = this.visitDashboard.bind(this);
  }
  visitDashboard(){
    this.props.navigator.push({
      name: 'Dashboard'
    });
  }
  render(){
    let titleConfig = { title: 'Landing', tintColor: 'white' };
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          title={titleConfig}
          tintColor={Colors.brandPrimary}
        />
        <View style={globals.flexCenter}>
          <Text style={globals.h2}>
            This is the Landing Page
          </Text>
          <TouchableOpacity onPress={this.visitDashboard}>
            <Text>Go to the Dashboard</Text>
          </TouchableOpacity>
        </View>
      </View>
    )
  }
}

export default Landing;

```

Let's go over what's going on in this component.

- We import the necessary components from React and React Native for building our component.
- We import our global styles object, as well as the `NavigationBar` and `Icon` components from our recently installed npm packages.
- We bind our class methods in the `constructor` function. We need to bind them so that the value of `this` that corresponds to the class instance itself persists.
- We define our `visitLanding` method, which routes the `Navigator` to the `Landing` component.
- We render our content with the `render` method.

To view our content now, we can simply import the component in `index.ios.js` and render it.

```javascript
index.ios.js

import React, { Component } from 'react';
import {
  AppRegistry,
  Navigator
} from 'react-native';

import Landing from './application/components/Landing';

class assemblies extends Component {
  render() {
    return (
      <Landing />
    );
  }
}

AppRegistry.registerComponent('assemblies', () => assemblies);

```

You should see something similar to this in the Simulator:

![navigator](/images/chapter-3/navigator-1.png)

### Adding Routing

We can't go to the **Dashboard** component just yet. When we press the **Go to the Dashboard** button, we get an error `Cannot read property 'push' of undefined`.

This means that we haven't defined our **Navigator** yet. Let's add that in `index.ios.js` and then reload.

```javascript
/* index.ios.js */
import React, { Component } from 'react';
import {
  AppRegistry,
  Navigator
} from 'react-native';

import Landing from './application/components/Landing';
import { globals } from './application/styles';

class assemblies extends Component {
  render() {
    return (
      <Navigator
        style={globals.flex}
        initialRoute={{ name: 'Landing' }}
        renderScene={(route, navigator) => {
          switch(route.name){
            case 'Landing':
              return (
                <Landing navigator={navigator}/>
            );
          }
        }}
      />
    );
  }
}

AppRegistry.registerComponent('assemblies', () => assemblies);

```

Alright, what's going on here?

- We render our **Navigator**, which currently has 3 properties - **style**, **initialRoute**, and **renderScene**.
- `initialRoute` is exactly what is sounds like - the first component we want to render when our app starts. We define this route with an object with the name of our first route.
- `renderScene` is a function which expects a component to be returned. We render the appropriate component depending on the name of the route. To achieve this, we use JavaScript's [**switch/case**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/switch) syntax.

With all that, we should now see the same component. However, we get a different result when we try to press the same button. This time, a blank screen appears. This is because we haven't defined our `Dashboard.js` component, nor included a route for it in the **Navigator**. Let's modify our **Navigator** first.

### Pushing and Popping Routes

```javascript
/* index.ios.js */

import React, { Component } from 'react';
import {
  AppRegistry,
  Navigator
} from 'react-native';

import Landing from './application/components/Landing';
import Dashboard from './application/components/Dashboard';
import { globals } from './application/styles';

class assemblies extends Component {
  render() {
    return (
      <Navigator
        style={globals.flex}
        initialRoute={{ name: 'Landing' }}
        renderScene={(route, navigator) => {
          switch(route.name){
            case 'Landing':
              return (
                <Landing navigator={navigator}/>
            );
            case 'Dashboard':
              return (
                <Dashboard navigator={navigator}/>
            );
          }
        }}
      />
    );
  }
}

AppRegistry.registerComponent('assemblies', () => assemblies);

```

Next let's define our `Dashboard` component.

```javascript
/* application/components/Dashboard.js */

import React, { Component } from 'react';
import {
  View,
  Text,
  TouchableOpacity
} from 'react-native';

import Icon from 'react-native-vector-icons/Ionicons';
import NavigationBar from 'react-native-navbar';
import Colors from '../styles/colors';
import { globals } from '../styles';

const BackButton = ({ handlePress }) => (
  <TouchableOpacity onPress={handlePress} style={globals.pa1}>
    <Icon name='ios-arrow-back' size={25} color='white' />
  </TouchableOpacity>
);

class Dashboard extends Component{
  constructor(){
    super();
    this.goBack = this.goBack.bind(this);
    this.visitLanding = this.visitLanding.bind(this);
  }
  visitLanding(){
    this.props.navigator.push({
      name: 'Landing'
    });
  }
  goBack(){
    this.props.navigator.pop();
  }
  render(){
    let titleConfig = {title: 'Dashboard', tintColor: 'white'};
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          title={titleConfig}
          tintColor={Colors.brandPrimary}
          leftButton={<BackButton handlePress={this.goBack}/>}
        />
        <View style={globals.flexCenter}>
          <Text style={globals.h2}>
            This is the Dashbaord
          </Text>
          <TouchableOpacity onPress={this.visitLanding}>
            <Text>
              Go to the Landing Page
            </Text>
          </TouchableOpacity>
        </View>
      </View>
    )
  }
}

export default Dashboard;

```

Notice a few things:
- We add a `leftButton` property to the navbar. This is the back icon that renders in the top left part of our screen. We define our `BackButton` component at the top of the file, using React's [**functional stateless component**](https://medium.com/@housecor/react-stateless-functional-components-nine-wins-you-might-have-overlooked-997b0d933dbc#.c8er29tt7) syntax.
- When we press the **Go to the Dashboard** button, we pass in a new object to our **Navigator** with the name of our new component (in our case, 'Dashboard'). Our **Navigator** understands this and delivers us to the correct component.

Now you should be able to go back and forth between the **Landing** and **Dashboard** screens. If you have any errors, try reloading in XCode with the `cmd + r` command. It may be that the fonts from our `react-native-vector-icons` package haven't yet loaded.

![navigator](/images/chapter-3/navigator-2.png)

### Overview of Components Used

Let's talk a bit about the components we have been using and the approach we are taking.

Think of `View` like `<div>` in the world of React for Web - it's the basic container for our component and something the packager can bundle and send to the view.

`TouchableOpacity` is a handy component provided by React Native that lets you easily create that "touchable" feel on any elements users will be tapping or pressing. Elements wrapped in `<TouchableOpacity>` will automatically modulate their opacity values when touched. This may seem like a small thing, but it's incredibly noticeable when it's missing, as your elements just sort of respond eventually without any apparent reaction to your touch. Make your interface feel responsive to input is a big part of Apple's [Human Interface Guidelines](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/index.html?utm_source=twitterfeed&utm_medium=twitter).

As we mock up our view, you can see this is just fairly familiar JSX syntax, pretty much XML with a few tweaks.

### JavaScript Styles

If you examing the `globals` style object in `application/styles/index.js`, you'll see that we are declaring our styles with Javascript in camel case, referencing keys of a styles object we declare and define below. Generally, these styles work pretty closely to CSS, and use flexbox to define containers and layouts. If you're not too familiar with how flexbox works (perhaps you've spent too much time making things backwards compatible for Internet Explorer), we highlight recommend you check out the excellent guide over at [CSS Tricks](https://css-tricks.com/snippets/css/a-guide-to-flexbox/).

*Note: One of the first 'gotchas' to avoid when working with flexbox is making sure you set `flex: 1` on your top-level components so they actually fill the view, otherwise you'll be very confused and frustrated when your app looks completely blank*

Don't worry, we'll cover flexbox and styling in React Native in far more detail as we go. We've provided all the styles you need for this tutorial so you can get familiar with the basics before having to worry too much about styling.

### Isn't that nice?

Take a minute and play with the views, switching back and forth between them. Right out of the gate you can see how much smoother things are than any hybrid app you've ever tried.

As you can see, the nice thing about **Navigator** is that we can customize how our screen looks at any given time. We can create a navbar with `react-native-navbar` and customize it with icons, or we can set up navigation in a different way. It's worth looking at the different options before deciding what's right for your app.

Okay, now it's time for another commit! Congrats on having delved into navigation with React Native. The **Navigator** API has many more options, some of which we will use in the tutorial. Please check out the [docs](https://facebook.github.io/react-native/docs/navigator.html) for specific API information.

Note: If you are interested in using **NavigationExperimental**, we plan to add a chapter in the appendix that deals with this, along with adding Redux state management to your applications. Honestly, the transition from **Navigator** to **NavigationExperimental** should feel very easy.

***
![GitHub logo](/images/github-logo.png)
[Commit 2](https://github.com/buildreactnative/assemblies-tutorial/tree/630635abda872078ea89937952f91f7cb95e7617) - "Create basic navigation with Navigator"
***

### Fleshing out the App

One important thing to understand about an open source project like React Native is the motivations of its sponsors. Facebook uses React Native currently in two apps - the Ads Manager app for iOS and Android, and partially in the Groups app. If we look at these apps, we can see where React Native's strengths are, and we should be looking to leverage them. This is why we decided to use tab bar navigation in Assemblies. Facebook uses this type of navigation in both apps and more apps are following suit. In this chapter, we will implement a simple TabBar navigation.

The interesting thing about tab bar navigation is that to accomplish it, you often need each tab to have its own **Navigator**. So we end up with **Navigators** inside of **Navigators**... It actually creates a nice effect and makes our app easy to get around.

I've gone ahead and wired up a few tabs. In all, there will be five tabs, but for now, I'll start with a profile view, a dashboard view, and a messages view.

### Using Fake Data to Prototype

Now that we're about to build out the app, there are a couple of things we should clarify:

*Why are we starting out with fake data?*

This is something we are strong believers in. Fake it and then make it. The wireframing process helps to funnel the idea of the product into a visual representation. A developer's job is to translate that into an actual product. It's very easy to focus on the programming problems like integration with a backend system and server, scalability, and so on. However, most often, the best thing to do at this point is to make a fake product. This helps get the UI components of the product in place so that you can think of the data integration later. This is the process we use and find it works extremely well. That said, different things work for different people, so sue the method that works for you on your own projects.

### Should We Be Using Redux?

*Will you be using an architecture like Flux or Redux to manage state between components?*

We won't be using either in this tutorial. We personally love Redux but even its creator has said that you shouldn't use it until you've felt the pain without it. This app could be a good candidate for Redux state management. However, we are also able to create a good product without it. So while the production version of Assemblies may incorporate Redux, the tutorial itself won't touch on the topic. We address how to structure a React Native app with `redux` in the Appendix section of the tutorial.

### Rounding out our Landing Page

Now we're going to fill in our landing page. Later, this will link to user registration and sign in, but for now we'll have it go directly to the dashboard. Let's place an image as the screen background using the `Dimensions` module. Then we'll use the `TouchableOpacity` component as a button that leads to the dashboard. You can download the image assets from the [open-source repository](https://github.com/buildreactnative/assemblies/tree/master/application/assets/images), or use the URL we provide below. Create a folder under **application** called **assets**, and an **images** folder in that. That is where we'll store our images for the tutorial.

```javascript
/* application/components/Landing.js */
import Icon from 'react-native-vector-icons/MaterialIcons';
import React, { Component } from 'react';
import {
  Text,
  TouchableOpacity,
  Image,
  View
} from 'react-native';

import Colors from '../styles/colors';
import { landingStyles } from '../styles';
import { globals } from '../styles';

const BackgroundImage = 'https://s3-us-west-2.amazonaws.com/assembliesapp/welcome%402x.png';
const Logo = 'https://s3-us-west-2.amazonaws.com/assembliesapp/logo.png';
const styles = landingStyles;

class Landing extends Component{
  constructor(){
    super();
    this.visitDashboard = this.visitDashboard.bind(this);
  }
  visitDashboard(){
    this.props.navigator.push({ name: 'Dashboard' })
  }
  render(){
    return (
      <View style={styles.container}>
        <View style={styles.container}>
          <Image
            style={styles.backgroundImage}
            source={{ uri: BackgroundImage }}
          />
        </View>
        <View style={globals.flexCenter}>
          <Image
            style={styles.logo}
            source={{ uri: Logo }}
          />
          <Text style={[globals.lightText, globals.h2, globals.mb2]}>
            assemblies
          </Text>
          <Text style={[globals.lightText, globals.h4]}>
            Where Developers Connect
          </Text>
        </View>
        <TouchableOpacity
          style={globals.button}
          onPress={this.visitDashboard}
        >
          <Icon name='person' size={36} color='white' />
          <Text style={globals.buttonText}>
            Go to Dashboard
          </Text>
        </TouchableOpacity>
      </View>
    );
  }
};

export default Landing;


```

![landing](/images/chapter-3/landing-1.png)

If you were to download the images and places them under `application/assets/images`, here's how your **Image** elements might look.

```javascript

<Image 
  style={styles.backgroundImage} 
  source={require('../assets/images/welcome.png')}
/>
</View>
<View style={globals.flexCenter}>
  <Image 
    style={styles.logo} 
    source={require('../assets/images/logo.png')}
  />
  <Text style={[globals.lightText, globals.h2, globals.mb2]}>
    assemblies
  </Text>
  <Text style={[globals.lightText, globals.h4]}>
    Where Developers Connect
  </Text>
</View>
...
```

As for the styles, some stuff should be pretty self-explanatory for those familiar with styling with CSS on the web. You'll see that we use some absolute positioning as well here, but the majority of components typically use flexbox for positioning.

Time for another commit to close out this chapter on setting up navigation:

***
![GitHub logo](/images/github-logo.png)
[Commit 3](https://github.com/buildreactnative/assemblies-tutorial/tree/ea505fc5f296a38f723ae05fedec3d46b5c0a584) - "Flesh out Landing screen"
***
