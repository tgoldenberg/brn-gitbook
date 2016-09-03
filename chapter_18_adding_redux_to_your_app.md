# Chapter 18: Adding Redux to your App

Redux, according to its Github README, is a "predictable state container for JavaScript apps." In other words, it's a way to create a single source of truth for your client-side state. This can very useful for complex applications that share data across different views. 

#### Setting Things Straight

However, do not think that everything application **needs** Redux. This is not true. Redux is an excellent pattern for when state is shared across many components. It is also a reliable way to test user flows and locating bottlenecks and bugs. There are situations where you probably shouldn't use Redux, though, and definitely not to start when using React or React Native. Here are some tweets by Dan Abramov, creator of Redux.

* ["Don’t use Redux unless you *tried* local component state and were dissatisfied."](https://twitter.com/dan_abramov/status/725089243836588032)
* ["There is so much FUD about local state. “setState is bad”, “no-set-state”, “root of all evil”, “use functional components”. State is fine."](https://twitter.com/dan_abramov/status/725089775783391232)
* ["Beginners take Redux principles (“use single state tree”) and think they’re universally good advice for any React app"](https://twitter.com/dan_abramov/status/760891641657954305)

And from Ryan Florence, author of `react-router`:

* ["why do so many think React means React + Webpack + Redux + CSS Modules + Hot Loading + Server Rendering + ..."](https://twitter.com/ryanflorence/status/724934112822329344)

#### How does Redux work?

Let's examine how Redux works. If you think of your application as a tree of components, then state flows downwards from higher to lower components. But what happens when sibling components need to be aware of a change? Then a lower component has to pass a callback to the higher component which then shares the state with the sibling. This happens in **Assemblies** with our user state. We initiate the initial user data in our **index.ios.js** file and pass it to the child components. However, in **profile.js**, for example, we initiate a change in the user object -- the user updates their profile. This change then needs to be passed up to the parent component via a callback (**updateUser**), and then the change is shared with the rest of the application.

What's the way around this? Redux introduces a client data store that holds application state, and that individual components can "connect" to. That means that we don't have to pass `currentUser={user}` as **props** in all of our components. Any component that needs access to the user object can "connect" to the Redux store.

![redux](https://css-tricks.com/wp-content/uploads/2016/03/redux-article-3-02.svg)

This means that our components can theoretically become "lighter", carrying less props. It also means that sharing data across components is seamless. Finally, holding state in a single store makes logging changes and identifying bugs much easier (more on this later). 

Instead of explaining Redux in abstract terms, let's dig in right away in the codebase and start implementing, learning more as we go along. Since the user data is shared across all of our components, it makes sense to incorporate user data in our Redux store to start. 

#### Getting Started

To start, let's install the dependencies we'll be needing to implement Redux in our React Native app. 

```
npm install --save redux react-redux redux-thunk
```

* Now we'll need to move the contents of **index.ios.js** to the file **application/containers/AppContainer.js**

* Next, replace **index.ios.js** with this code:

```javascript
import React, { Component } from 'react';
import { AppRegistry } from 'react-native';
import { createStore, applyMiddleware } from 'redux';
import { Provider } from 'react-redux';
import thunk from 'redux-thunk';

import reducers from './application/reducers/index';
import AppContainer from './application/containers/AppContainer';

const createStoreWithMiddleware = applyMiddleware(thunk)(createStore);
const store = createStoreWithMiddleware(reducers);

class assembliesTutorial extends Component{
  render(){
    return (
      <Provider store={store} >
        <AppContainer />
      </Provider>
    )
  }
}

AppRegistry.registerComponent('assembliesTutorial', () => assembliesTutorial);
```

Your file **application/containers/AppContainer.js** should look like this (since we're not using **AppRegistry** for this component anymore.

```javascript
import React, { Component } from 'react';
import {
  ActivityIndicator,
  View,
  Navigator,
  AsyncStorage
} from 'react-native';

import Landing from '../components/Landing';
import Register from '../components/accounts/Register';
import RegisterConfirmation from '../components/accounts/RegisterConfirmation';
import Login from '../components/accounts/Login';
import Dashboard from '../components/Dashboard';
import Loading from '../components/shared/Loading';
import { Headers } from '../fixtures';
import { extend } from 'underscore';
import { API, DEV } from '../config';
import { globals } from '../styles';

export default class AppContainer extends Component {
  constructor(){
    super();
    this.logout = this.logout.bind(this);
    this.updateUser = this.updateUser.bind(this);
    this.state = {
      user          : null,
      ready         : false,
      initialRoute  : 'Landing',
    }
  }
  componentDidMount(){
    this._loadLoginCredentials()
  }
  async _loadLoginCredentials(){
    try {
      let sid = await AsyncStorage.getItem('sid');
      console.log('SID', sid);
      if (sid){
        this.fetchUser(sid);
      } else {
        this.ready();
      }
    } catch (err) {
      this.ready(err);
    }
  }
  ready(err){
    this.setState({ ready: true });
  }
  fetchUser(sid){
    fetch(`${API}/users/me`, { headers: extend(Headers, { 'Set-Cookie': `sid=${sid}`})})
    .then(response => response.json())
    .then(user => this.setState({ user, ready: true, initialRoute: 'Dashboard' }))
    .catch(err => this.ready(err))
    .done();
  }
  logout(){
    this.nav.push({ name: 'Landing' })
  }
  updateUser(user){
    this.setState({ user });
  }
  render() {
    if ( ! this.state.ready ) { return <Loading /> }
    return (
      <Navigator
        style={globals.flex}
        ref={(el) => this.nav = el }
        initialRoute={{ name: this.state.initialRoute }}
        renderScene={(route, navigator) => {
          switch(route.name){
            case 'Landing':
              return (
                <Landing navigator={navigator}/>
            );
            case 'Dashboard':
              return (
                <Dashboard
                  updateUser={this.updateUser}
                  navigator={navigator}
                  logout={this.logout}
                  user={this.state.user}
                />
            );
            case 'Register':
              return (
                <Register navigator={navigator}/>
            );
            case 'RegisterConfirmation':
              return (
                <RegisterConfirmation
                  {...route}
                  updateUser={this.updateUser}
                  navigator={navigator}
                />
            );
            case 'Login':
              return (
                <Login
                  navigator={navigator}
                  updateUser={this.updateUser}
                />
            );
          }
        }}
      />
    );
  }
}
```

Finally, create the folders **application/actions**, **application/constants**, **application/reducers**, and create the file **application/reducers/index.js** with these contents:

```javascript
import { combineReducers } from 'redux';

const accounts = (state={}, action) => {
  switch(action.type){
    default:
      return state;
  }
}

const appReducers = combineReducers({
  accounts
});

export default appReducers;
```

Now run your app, and everything should work exactly as it did before!

#### What did we do?

Redux requires us to link our top-level component with the Redux store. We do this in **index.ios.js**, by first creating a store ```const store = createStoreWithMiddleware(reducers)```, and then connecting that store to a **Provider** component. We then place our content (our app), as a child to the **Provider** component.

To create a store, you must provide the **createStoreWithMiddleware** function with an object that contains your reducers. A **reducer** is a function that takes in an action and returns a new state. We use the **combineReducers** function to enable us to have multiple reducers. Right now, our reducers object contains a single function which returns an empty object no matter what.

```
const accounts = (state={}, action) => {
  switch(action.type){
    default:
      return state;
  }
}
```

The **combineReducers** function simply allows us to have several of these state objects, and combine them into a single object. The use of **thunk** is for sending asyncronous actions to our store, a common need in complex apps, but not absolutely necessary.

### Building Our Store

The next step is identifying which aspects of our application state that we want to keep in our store. To start, we can take the local state in our **AppContainer.js** component. Let's make a file **application/reducers/accounts.js** and use it to store our initial state.

```javascript
/* application/reducers/accounts.js */

const initialState = {
  user          : null,
  ready         : false,
  initialRoute  : 'Landing',
};

const accounts = (state=initialState, action) => {
  switch(action.type){
    default:
      return state;
  }
};

export default accounts;
```

And we can **import** this into our main **reducers/index.js** file:
```javascript
import accounts from './accounts';
import { combineReducers } from 'redux';

const appReducers = combineReducers({
  accounts
});

export default appReducers;
```

Notice how we replaced our original **accounts** function with our new one.
