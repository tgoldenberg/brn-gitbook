**Redux**, according to its [Github README](https://github.com/reactjs/redux), is a "predictable state container for JavaScript apps." In other words, it's a way to create a single source of truth for your client-side state. This can be very useful for complex applications that share data across different views. *Note: this content assumes you have built the app according to the previous chapters and are familiar with ES6 syntax.*

#### Setting Things Straight

Don't think that every application **needs** Redux. This is not true. Redux is an excellent pattern for when state is shared across many components. It is also a great tool for monitoring user flows and identifying bugs. However, there are situations where you probably shouldn't use Redux, and that includes when you are starting out using React and React Native. Here are some tweets by [Dan Abramov](https://twitter.com/dan_abramov), creator of Redux.

* ["Don’t use Redux unless you *tried* local component state and were dissatisfied."](https://twitter.com/dan_abramov/status/725089243836588032)
* ["There is so much FUD about local state. “setState is bad”, “no-set-state”, “root of all evil”, “use functional components”. State is fine."](https://twitter.com/dan_abramov/status/725089775783391232)
* ["Beginners take Redux principles (“use single state tree”) and think they’re universally good advice for any React app"](https://twitter.com/dan_abramov/status/760891641657954305)

And from [Ryan Florence](https://twitter.com/ryanflorence), author of **React Router**:

* ["why do so many think React means React + Webpack + Redux + CSS Modules + Hot Loading + Server Rendering + ..."](https://twitter.com/ryanflorence/status/724934112822329344)

#### How does Redux work?

Let's examine how Redux works. If you think of your application as a tree of components, then state flows downwards from higher to lower components. But what happens when sibling components need to be aware of a change? Then a lower component has to pass a callback to the higher component which then shares the state with the sibling. This happens in **Assemblies** with our user state. We initialize our user data in the top-level **index.ios.js** file and pass it to the child components. However, in **profile.js**, for example, we initiate a change in the user object -- the user updates their profile. This change then needs to be passed up to the parent component via a callback (**updateUser**), and then the change is shared with the rest of the application.

What's the way around this? Redux introduces a client data store that holds application state, which individual components can "connect" to. That means that we don't have to pass `updateUser={this.props.updateUser}` as **props** in all of our components. Any component that needs read or write access to the **user** object can "connect" to the Redux store.

![redux](https://css-tricks.com/wp-content/uploads/2016/03/redux-article-3-02.svg)

This means that our components can theoretically become "lighter" (carrying less props), and that sharing data across components is seamless. Holding state in a single store also makes logging changes and identifying bugs much easier (more on this later). 

Instead of explaining Redux in abstract terms, let's dig right into the codebase and start using it, learning as we go along. Since the **user** data is shared across all of our components, it makes sense to incorporate user data in our Redux store to start. 

#### Getting Started

To start, let's install the dependencies we'll be needing to implement Redux in our React Native app. 

```
npm install --save redux react-redux redux-thunk
```

Now we'll need to move the contents of **index.ios.js** to the file **application/containers/AppContainer.js**, and replace **index.ios.js** with this code:

```javascript
/* index.ios.js */
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

Redux requires us to link our top-level component with the Redux store. We do this in **index.ios.js**, by first creating a store,

```javascript
const store = createStoreWithMiddleware(reducers)
```
and then connecting that store to a **Provider** component. We then place our content (our app), as a child to the **Provider** component.

To create a store, you must provide the **createStoreWithMiddleware** function with an object that contains your reducers. A **reducer** is a function that takes in an action (a JavaScript object) and returns a new state. We use the **combineReducers** function to enable us to have multiple reducers. Right now, our reducers object contains a single function which returns an empty object no matter what.

```
const accounts = (state={}, action) => {
  switch(action.type){
    default:
      return state;
  }
}
```

The **combineReducers** function simply allows us to have several of these state objects, and combine them into a single object. The use of **thunk** is for sending asyncronous actions to our store, a common need in complex apps, but not absolutely necessary (more on this later).

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

Notice how we replaced our original **accounts** function with our new one. Now, how do we access this state in our **AppContainer.js** component? Through the `connect` function that **react-redux** provides us. Let's modify **AppContainer.js** to be a connector to the Redux store, and let's move the content of **AppContainer.js** to **application/components/App.js**. We'll also change the local state of **App.js** to rely on props rather than on local state.

```javascript
/* application/components/App.js */
import React, { Component } from 'react';
import {
  ActivityIndicator,
  View,
  Navigator,
  AsyncStorage
} from 'react-native';
import { extend } from 'underscore';

import Dashboard from '../components/Dashboard';
import Landing from '../components/Landing';
import Loading from '../components/shared/Loading';
import Login from '../components/accounts/Login';
import Register from '../components/accounts/Register';
import RegisterConfirmation from '../components/accounts/RegisterConfirmation';
import { API, DEV } from '../config';
import { Headers } from '../fixtures';
import { globals } from '../styles';

export default class App extends Component {
  constructor(){
    super();
    this.logout = this.logout.bind(this);
  }
  componentDidMount(){
    this._loadLoginCredentials()
  }
  async _loadLoginCredentials(){
    try {
      let sid = await AsyncStorage.getItem('sid');
      if (sid){
        this.fetchUser(sid);
      } else {
        this.props.doneFetching();
      }
    } catch (err) {
      this.props.doneFetching(err);
    }
  }
  fetchUser(sid){
    fetch(`${API}/users/me`, { headers: extend(Headers, { 'Set-Cookie': `sid=${sid}`})})
    .then(response => response.json())
    .then(user => this.props.sendDashboard(user))
    .catch(err => this.props.doneFetching(err))
    .done();
  }
  logout(){
    this.nav.push({ name: 'Landing' })
  }
  render() {
    let { initialRoute, updateUser, user, ready } = this.props;
    if ( !ready ) { return <Loading /> }
    return (
      <Navigator
        style={globals.flex}
        ref={(el) => this.nav = el }
        initialRoute={{name: initialRoute}}
        renderScene={(route, navigator) => {
          switch(route.name){
            case 'Landing':
              return (
                <Landing navigator={navigator}/>
            );
            case 'Dashboard':
              return (
                <Dashboard
                  updateUser={updateUser}
                  navigator={navigator}
                  logout={this.logout}
                  user={user}
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
                  updateUser={updateUser}
                  navigator={navigator}
                />
            );
            case 'Login':
              return (
                <Login
                  navigator={navigator}
                  updateUser={updateUser}
                />
            );
          }
        }}
      />
    );
  }
}
```

Notice that **App.js** has no local state, and only relies on props. Let's add these props in our new **AppContainer.js**:

```javascript
import React from 'react';
import { connect } from 'react-redux';
import App from '../components/App';

export default connect(
  (state, ownProps) => ({
    ...state.accounts
  }),
  (dispatch) => ({
    doneFetching: () => {
      console.log('DONE FETCHING');
    },
    sendDashboard: (user) => {
      console.log('SEND DASHBOARD');
    },
    updateUser: (user) => {
      console.log('UPDATE USER');
    }
  })
)(App);

```

Now our app should work, except it only loads the initial state (a loading indicator). It doesn't change anything because we haven't added that yet! First we need to go over how our **connect** function works to connect our Redux store to our **App.js** component.

### Connecting to the Redux store

The **connect** function accepts two parameters and returns another function. This function then accepts a component as a parameter, and returns the component with the **props** passed down through the store.

The first parameter is a function that accepts the entire Redux store, and passes specific props to the component. Here we pass the **accounts** part of our store:
```
(state, ownProps) => ({
  ...state.accounts
}),
```
Notice that this function also takes an **ownProps** parameter. This is useful if the container is the child of another component, in which case you would pass the props from the parent to the connected component. 

The second parameter takes the **dispatch** function as a parameter, and returns more **props** that act as functions that can alter the Redux store. In our current example, these don't do anything other than console.log their existence.

### Dispatching Actions

To change the state of the Redux store, we'll need dispatch an **action**. An action is essentially an object with a **type** attribute, and any other data that it requires. Let's dispatch these actions in our container, by defining them in **application/actions/index.js** and **application/constants/index.js**.

```javascript
/* application/constants/index.js */
export const SEND_DASHBOARD = 'accounts/SEND_DASHBOARD';
export const DONE_FETCHING = 'accounts/DONE_FETCHING';
export const UPDATE_USER = 'accounts/UPDATE_USER';
```
```javascript
/* application/actions/index.js */
import * as actionTypes from '../constants';

export const sendDashboard = (user) => ({
  type: actionTypes.SEND_DASHBOARD,
  user
});

export const doneFetching = () => ({
  type: actionTypes.DONE_FETCHING
});

export const updateUser = (user) => ({
  type: actionTypes.UPDATE_USER,
  user
});
```
Now we can send these actions from **AppContainer.js**:

```javascript
/* application/containers/AppContainer.js */
import React from 'react';
import { connect } from 'react-redux';
import { updateUser, sendDashboard, doneFetching } from '../actions';
import App from '../components/App';

export default connect(
  (state, ownProps) => ({
    ...state.accounts
  }),
  (dispatch) => ({
    doneFetching: () => {
      dispatch(doneFetching());
    },
    sendDashboard: (user) => {
      dispatch(sendDashboard(user));
    },
    updateUser: (user) => {
      dispatch(updateUser(user));
    }
  })
)(App);

```
Let's also add a console.log in our **application/reducers/accounts.js** file:
```javascript

const initialState = {
  user          : null,
  ready         : false,
  initialRoute  : 'Landing',
};

const accounts = (state=initialState, action) => {
  console.log('ACTION', action);
  switch(action.type){
    default:
      return state;
  }
};

export default accounts;
```

Now let's see what happens when we run our app and debug remotely in our browser:

![redux](/images/chapter-19/redux-1.png)

Notice that our action has been recognized by the reducer. It has a **type**, as well as the field **user** with the data for the current users. Then why is our app still showing a loading indicator? Because we haven't changed our Redux store!

#### Modifying the Redux Store

There are some [best practices](http://redux.js.org/docs/introduction/ThreePrinciples.html) for using Redux, and one is to use pure functions, meaning that we don't mutate the Redux store state. Instead, we replace it with a new state. In our case, we want to replace the field **user** in the Redux store with our new value, as well as setting the **initialRoute** and **ready** fields. Here's what that looks like:

```javascript
/* application/reducers/accounts.js */
import * as actionTypes from '../constants';

const initialState = {
  user          : null,
  ready         : false,
  initialRoute  : 'Landing',
};

const accounts = (state=initialState, action) => {
  console.log('ACTION', action);
  switch(action.type){
    case actionTypes.SEND_DASHBOARD:
      return {
        ...state,
        user: action.user,
        ready: true,
        initialRoute: 'Dashboard'
      };
    case actionTypes.DONE_FETCHING:
      return {
        ...state,
        ready: true
      };
    case actionTypes.UPDATE_USER:
      return {
        ...state,
        user: action.user
      };
    default:
      return state;
  }
};

export default accounts;
```

Now we'll see that we can login to the app, refresh and then be automatically logged in!

### Reaping the Benefits

Now we don't have to pass an **updateUser** function to all children components. Wherever we need to update our user object in the Redux store, we simply connect that component to the store and dispatch the **updateStore** action. 

We can also throw our **async** actions into the dispatch action, since we are using **redux-thunk**. Here's how that would look:

```javascript 
/* application/components/App.js */
import React, { Component } from 'react';
import { Navigator } from 'react-native';

import Dashboard from '../components/Dashboard';
import Landing from '../components/Landing';
import Loading from '../components/shared/Loading';
import Login from '../components/accounts/Login';
import Register from '../components/accounts/Register';
import RegisterConfirmation from '../components/accounts/RegisterConfirmation';
import { globals } from '../styles';

export default class App extends Component {
  constructor(){
    super();
    this.logout = this.logout.bind(this);
  }
  componentDidMount(){
    this.props.loadCredentials();
  }
  logout(){
    this.nav.push({ name: 'Landing' })
  }
  render() {
    let { initialRoute, user, ready } = this.props;
    if ( !ready ) { return <Loading /> }
    return (
      <Navigator
        style={globals.flex}
        ref={(el) => this.nav = el }
        initialRoute={{name: initialRoute}}
        renderScene={(route, navigator) => {
          switch(route.name){
            case 'Landing':
              return (
                <Landing navigator={navigator}/>
            );
            case 'Dashboard':
              return (
                <Dashboard navigator={navigator} logout={this.logout} user={user}/>
            );
            case 'Register':
              return (
                <Register navigator={navigator}/>
            );
            case 'RegisterConfirmation':
              return (
                <RegisterConfirmation {...route} navigator={navigator}/>
            );
            case 'Login':
              return (
                <Login navigator={navigator} />
            );
          }
        }}
      />
    );
  }
}

```
```javascript
/* application/containers/AppContainer.js */
import React from 'react';
import { connect } from 'react-redux';
import { loadCredentials } from '../actions';
import App from '../components/App';

export default connect(
  (state, ownProps) => ({
    ...state.accounts
  }),
  (dispatch) => ({
    loadCredentials(){
      dispatch(loadCredentials())
    }
  })
)(App);

```

```javascript
/* application/actions/index.js */
import * as actionTypes from '../constants';
import { extend } from 'underscore';
import { API, DEV } from '../config';
import { Headers } from '../fixtures';
import { AsyncStorage } from 'react-native';

export const sendDashboard = (user) => ({
  type: actionTypes.SEND_DASHBOARD,
  user
});

export const doneFetching = () => ({
  type: actionTypes.DONE_FETCHING
});

export const updateUser = (user) => ({
  type: actionTypes.UPDATE_USER,
  user
});

export const fetchUser = (sid) => (
  (dispatch, getState) => {
    fetch(`${API}/users/me`, { headers: extend(Headers, { 'Set-Cookie': `sid=${sid}`})})
    .then(response => response.json())
    .then(user => dispatch(sendDashboard(user)))
    .catch(err => dispatch(doneFetching()))
    .done();
  }
);

export const loadCredentials = () => (
  async (dispatch, getState) => {
    try {
      let sid = await AsyncStorage.getItem('sid');
      if (sid) {
        dispatch(fetchUser(sid));
      } else {
        dispatch(doneFetching())
      }
    } catch (err) {
      dispatch(doneFetching())
    }
  }
)
```

Here, we are offloading a lot of the functionality to our **actions**. Because these are pure JavaScript functions untangled with our UI, it makes both the functionality and the components easier to reason about and test. What **redux-thunk** allows us to do is to dispatch asynchronous actions -- to fetch the user's session id in local storage, and then call our API to log in the user. **Thunk** achieves this by allowing our actions to return an asynchronous function. If this is overly confusing, then stick with the previous strategy of performing the asynchronous actions in the container or component. This is definitely a more nuanced point of using Redux.

Notice how simpler our rendering component is, and how much easier to it is to reason about it what it does? That is the beauty of **Redux**. We also are passing less **props** to the child components. This means that we have to **connect** the child components wherever we need to update our **user** object. This is mainly in **Login.js**, **Register.js**, and  **ProfileView.js**. Here's a brief look at what that would look like:

```javascript
/* application/containers/LoginContainer.js */
import React from 'react';
import { connect } from 'react-redux';
import { updateUser } from '../actions';
import Login from '../components/accounts/Login';

export default connect(
   (state, ownProps) => ({
    ...ownProps
  }),
  (dispatch) => ({
    updateUser(user){
      dispatch(updateUser(user))
    }
  })
)(Login);
```
```javascript
/* application/containers/RegisterConfirmationContainer.js */
import React from 'react';
import { connect } from 'react-redux';
import { updateUser } from '../actions';
import RegisterConfirmation from '../components/accounts/RegisterConfirmation';

export default connect(
  (state, ownProps) => ({
    ...ownProps
  }),
  (dispatch) => ({
    updateUser(user){
      dispatch(updateUser(user))
    }
  })
)(RegisterConfirmation);
```
```javascript
/* application/containers/ProfileContainer.js */
import React from 'react';
import { connect } from 'react-redux';
import { updateUser } from '../actions';
import ProfileView from '../components/profile/ProfileView';

export default connect(
  (state, ownProps) => ({
    ...ownProps
  }),
  (dispatch) => ({
    updateUser(user){
      dispatch(updateUser(user))
    }
  })
)(ProfileView);
```

```javascript
/* application/components/App.js */
import React, { Component } from 'react';
import {
  ActivityIndicator,
  Navigator,
} from 'react-native';

import Dashboard from '../components/Dashboard';
import Landing from '../components/Landing';
import Loading from '../components/shared/Loading';
import LoginContainer from '../containers/LoginContainer';
import Register from '../components/accounts/Register';
import RegisterConfirmationContainer from '../containers/RegisterConfirmationContainer';
import { globals } from '../styles';

export default class App extends Component {
  constructor(){
    super();
    this.logout = this.logout.bind(this);
  }
  componentDidMount(){
    this.props.loadCredentials();
  }
  logout(){
    this.nav.push({ name: 'Landing' })
  }
  render() {
    let { initialRoute, user, ready } = this.props;
    if ( !ready ) { return <Loading /> }
    return (
      <Navigator
        style={globals.flex}
        ref={(el) => this.nav = el }
        initialRoute={{name: initialRoute}}
        renderScene={(route, navigator) => {
          switch(route.name){
            case 'Landing':
              return (
                <Landing navigator={navigator}/>
            );
            case 'Dashboard':
              return (
                <Dashboard navigator={navigator} logout={this.logout} user={user}/>
            );
            case 'Register':
              return (
                <Register navigator={navigator}/>
            );
            case 'RegisterConfirmation':
              return (
                <RegisterConfirmationContainer {...route} navigator={navigator}/>
            );
            case 'Login':
              return (
                <LoginContainer navigator={navigator} />
            );
          }
        }}
      />
    );
  }
}
```
```javascript
/* application/components/Dashboard.js */
import React, { Component } from 'react';
import { TabBarIOS } from 'react-native';
import { TabBarItemIOS } from 'react-native-vector-icons/Ionicons';

import ActivityView from './activity/ActivityView';
import MessagesView from './messages/MessagesView';
import ProfileContainer from '../containers/ProfileContainer';
import GroupsView from './groups/GroupsView';
import CalendarView from './calendar/CalendarView';
import { Headers } from '../fixtures';
import { API, DEV } from '../config';

class Dashboard extends Component{
  constructor(){
    super();
    this.logout = this.logout.bind(this);
    this.state = {
      selectedTab: 'Activity'
    };
  }
  logout(){ /* end the current session and redirect to the Landing page */
    fetch(`${API}/users/logout`, { method: 'POST', headers: Headers })
    .then(response => response.json())
    .then(data => this.props.logout())
    .catch(err => {})
    .done();
  }
  render(){
    let { user } = this.props;
    return (
      <TabBarIOS>
        <TabBarItemIOS
          title='Activity'
          selected={this.state.selectedTab === 'Activity'}
          iconName='ios-pulse'
          onPress={() => this.setState({ selectedTab: 'Activity' })}
        >
          <ActivityView currentUser={user}/>
        </TabBarItemIOS>
        <TabBarItemIOS
          title='Groups'
          selected={ this.state.selectedTab == 'Groups' }
          iconName='ios-people'
          onPress={() => this.setState({ selectedTab: 'Groups' })}
        >
          <GroupsView currentUser={user}/>
        </TabBarItemIOS>
        <TabBarItemIOS
          title='Calendar'
          selected={ this.state.selectedTab == 'Calendar' }
          iconName='ios-calendar'
          onPress={() => this.setState({ selectedTab: 'Calendar' })}
        >
          <CalendarView currentUser={user}/>
        </TabBarItemIOS>
        <TabBarItemIOS
          title='Messages'
          selected={this.state.selectedTab === 'Messages'}
          iconName='ios-chatboxes'
          onPress={() => this.setState({ selectedTab: 'Messages' })}
        >
          <MessagesView currentUser={user}/>
        </TabBarItemIOS>
        <TabBarItemIOS
          title='Profile'
          selected={this.state.selectedTab === 'Profile'}
          iconName='ios-person'
          onPress={() => this.setState({ selectedTab: 'Profile' })}
        >
          <ProfileContainer currentUser={user} logout={this.logout} {...this.props}/>
        </TabBarItemIOS>
      </TabBarIOS>
    )
  }
}

export default Dashboard;

```

Now if you edit your user information or logout and login, everything works as it's supposed to. Notice that in our new containers, we pass the **ownProps** down, in order to inherit props from the higher-level components.

### Taking it Further

From all this, you should be able to see in what directions you can take Redux. You could even replace all local state with Redux store objects. However, that isn't always advisable, for several reasons.

Another cool thing about Redux is the ability to log every change in client state. Dan Abramov explains how to do so in the free video series [Building React Applications with Idiomatic Redux](https://egghead.io/courses/building-react-applications-with-idiomatic-redux). 

Finally, if you find your team embracing the Redux philosophy, you may want to explore the new experimental navigation module in React Native, **NavigationExperimental**. It was specifically designed to connect routing events to a Redux store, and works really well with that setup. For more info on **NavigationExperimental**, check out the [official documents](https://facebook.github.io/react-native/docs/navigation.html). These posts are also helpful:

* [First look: React Native Navigator Experimental](https://medium.com/@dabit3/first-look-react-native-navigator-experimental-9a7cf39a615b#.d6bradcn1)
* [Github repository with wiki](https://github.com/ericvicenti/navigation-rfc)
* [NavigationExperimental Premier: A Simple Recipe](http://www.reactnativediary.com/2016/06/23/navigation-examples-1.html)

For more tips on managing Redux applications, please look at the following resources: 

* [Getting Started with Redux](https://egghead.io/courses/getting-started-with-redux)
* [Building React Applications with Idiomatic Redux](https://egghead.io/courses/building-react-applications-with-idiomatic-redux)
* [Redux docs](http://redux.js.org/docs/introduction/)

It's always a good idea to start small -- go through the videos by Dan Abramov and try to replicate on your computer all of the examples. Don't commit to a specific strategy until you understand how to utilize the tools!

Good luck and happy coding!
