# Chapter 12: Final Touches

Let's review what we've accomplished before and what our app can do. 

- User authentication (login, register, logout, edit profile)
- TabBar navigation 
- Ability to create groups
- Ability to schedule and RSVP to events
- Ability to message users and view messages by conversation
- Notification system for new events and messages

What else can we do to improve the quality of our app? For one, we can make logging in a little easier. Our users shouldn't have to login everytime they use the app (that might be more appropriate for a financial app, for example). 

To achieve this, we will store the users session id in the `AsyncStorage` module and login when the user opens the app. If no account has been created, the app will run as usual.

```javascript
index.ios.js
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 * @flow
 */

import React, { Component } from 'react';
import {
  AppRegistry,
  Navigator,
  StyleSheet,
  AsyncStorage
} from 'react-native';

import Landing from './application/components/Landing';
import Dashboard from './application/components/Dashboard';
import Register from './application/components/accounts/Register';
import Login from './application/components/accounts/Login';
import RegisterConfirm from './application/components/accounts/RegisterConfirm';
import Loading from './application/components/utilities/Loading';
import { API, DEV } from './application/config';

class assemblies extends Component {
  constructor(){
    super();
    this.updateUser = this.updateUser.bind(this);
    this.state = {
      user: null,
      ready: false,
      initialRoute: 'Landing'
    }
  }
  async _loadLoginCredentials(){
    try {
      let sid = await AsyncStorage.getItem('sid');
      if (sid){
        fetch(`${API}/users/me`, {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'Set-Cookie': `sid=${sid}`
          }
        })
        .then(response => response.json())
        .then(data => { this.setState({ user: data, ready: true, initialRoute: 'Dashboard' }) })
        .catch(err => { if (DEV) { console.log('Connection error ', err); } })
        .done();
      } else {
        this.setState({ ready: true })
      }
    } catch (err) {
      this.setState({ ready: true })
    }
  }
  componentDidMount(){
    this._loadLoginCredentials();
  }
  updateUser(user){
    this.setState({ user: user });
  }
  render() {
    let { user, ready, initialRoute } = this.state;
    if (! ready ) { return <Loading /> }
    return (
      <Navigator
        ref='nav'
        initialRoute={{name: initialRoute, index: 0}}
        renderScene={(route, navigator) => {
        switch(route.name){
          case 'Landing':
            return <Landing navigator={navigator} />
            break;
          case 'Dashboard':
            return (
              <Dashboard
                navigator={navigator}
                currentUser={user}
                updateUser={this.updateUser}
              />
            );
          case 'Register':
            return <Register navigator={navigator} />
          case 'Login':
            return (
              <Login
                navigator={navigator}
                updateUser={this.updateUser}
              />
            );
          case 'RegisterConfirm':
            return (
              <RegisterConfirm
                {...route}
                navigator={navigator}
                updateUser={this.updateUser}
              />
            );
          }
        }}
        configureScene={() => Navigator.SceneConfigs.PushFromRight}
      />
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF',
  },
  welcome: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
  instructions: {
    textAlign: 'center',
    color: '#333333',
    marginBottom: 5,
  },
});

AppRegistry.registerComponent('assemblies', () => assemblies);

application/components/accounts/Login.js
...
loginUser(){
  if (DEV) { console.log('Logging in...'); }
  let { email, password } = this.state;
  let { updateUser } = this.props;
  fetch(`${API}/users/login`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      username: email,
      password: password
    })
  })
  .then(response => response.json())
  .then(data => {
    if (data.status === 401){
      this.setState({ errorMsg: 'Email or password was incorrect.' });
    } else {
      console.log('DATA', data);
      AsyncStorage.setItem('sid', data.id);
      fetch(`${API}/users/me`, {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
          'Set-Cookie': `sid=${data.id}`
        }
      })
      .then(response => response.json())
      .then(data => {
        if (DEV){ console.log('Logged in user: ', data); }
        updateUser(data);
        this.props.navigator.push({
          name: 'Dashboard'
        })
      })
      .catch(err => {
        if (DEV) { console.log('Connection error ', err); }
        this.setState({ errorMsg: 'Connection error.' });
      })
      .done();
    }
  })
  .catch(err => {
    if (DEV) { console.log('Connection error ', err); }
    this.setState({ errorMsg: 'Connection error.' });
  })
  .done();
}
...
```

And voila! Our user is automatically logged in when they open the app. Now, the wrong way to do this would be to store the user's password in the `AsyncStorage`, as this would be insecure.

## 12.2 More Robust Tabs

Currently, each of our tab routes are fairly limited in what they can do. For example, how do we message a user from the `Event` page? The `Event` page appears in 4 of our 5 routes. To enable this, we need to add the `Conversation` route in each of these routes.

Let's start with the `ActivityView`.

```javascript


```