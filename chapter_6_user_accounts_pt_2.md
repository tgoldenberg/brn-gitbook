# Chapter 6: User Accounts Pt. 2

At the end of the last chapter, we had created a user through Deployd, logged into the dashboard and fleshed out the Profile view with real data. However, there are two important things that we neglected related to user accounts. These are logging out and registering an account. 

## 6.1 Logging Out

Since registering is much trickier, let’s first implement logging out. Our API has an endpoint for logout - `/users/logout`. When the user initiates logout, we want to call this endpoint and then redirect the user to the Landing page.

Let’s edit our ProfileView and Dashboard to reflect these changes. We will reference a `prop` of `logout` in the ProfileView and define the method in the parent `Dashboard`.

```javascript
application/components/profile/ProfileView.js
…

render() {
  let { currentUser, logout } = this.props;
  console.log('CURRENT USER', currentUser);
  return (
…
<TouchableOpacity style={styles.logoutButton} onPress={logout}>
  <Text style={styles.logoutText}>Logout</Text>
</TouchableOpacity>
…

```

```javascript
application/components/Dashboard.js
…
import { DEV, API } from ‘./config’;
…
constructor(props){
  super(props);
  this.logout = this.logout.bind(this);
  this.state = {
    selectedTab: 'Activity',
  };
}
logout(){
  fetch(`${API}/users/logout`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    }
  })
  .then(response => response.json())
  .then(data => {
    this.props.navigator.popToTop();
  })
  .catch(err => {})
  .done();
}
…
<ProfileView currentUser={currentUser} logout={this.logout}/>
…
```

Let’s commit at this point.

## 6.2 Registration

Now we’re left with the most complex part of user accounts – registration. This doesn’t have to be complicated; we could just ask for our users email and a password. After all, that’s enough information to create a user with Deployd. However, we want more information about our users. We want to know what city they live in, so we can suggest nearby meetups. We want to know what technologies they are interested in, for similar reasons. We want their first and last name, since many “assemblies” require a person’s real name to be admitted to the venue location. Finally, we want an avatar for our users so that our users are able to know each other a little better.

We could, of course, just make a single view with all of these inputs available. However, that wouldn’t be a very good user experience. Long forms are off-putting to potential users, and we want our registration process to be relatively smooth. That’s why we take a step approach for user registration. 

In step 1, the user will enter their email, first and last name, and password – all absolutely necessary information. In step 2, we will ask the user for their interests, location, and a photo to use as their avatar. All of these steps, except for the user’s location, are optional. If the user doesn’t select an avatar, we can use a default image. If the user doesn’t specify their interests, we can ask them for them later. 

Let’s flesh out step 1 of our user registration process.

```javascript
application/components/accounts/Register.js

import React, { Component } from 'react';
import {
  Text,
  View,
  ScrollView,
  TextInput,
  TouchableOpacity,
  Dimensions
} from 'react-native';
import Icon from 'react-native-vector-icons/Ionicons';
import NavigationBar from 'react-native-navbar';
import Colors from '../../styles/colors';
import Globals from '../../styles/globals';
import {GooglePlacesAutocomplete} from 'react-native-google-places-autocomplete';
import {DEV} from '../../config';

const { width: deviceWidth, height: deviceHeight } = Dimensions.get('window');

const LeftBtnConfig = ({ navigator }) => {
  return (
    <TouchableOpacity style={Globals.backButton} onPress={()=>{
      navigator.pop();
    }}>
      <Icon name="ios-arrow-back" size={25} color="#ccc" />
    </TouchableOpacity>
  );
};


class Register extends Component{
  constructor(){
    super();
    this.state = {
      email: '',
      password: '',
      firstName: '',
      lastName: '',
      location: null
    }
  }
  render(){
    let { navigator } = this.props;
    let titleConfig = { title: 'Create Account', tintColor: 'white' };
    return (
      <View style={styles.container}>
        <NavigationBar title={titleConfig} tintColor={Colors.brandPrimary} leftButton={<LeftBtnConfig navigator={navigator}/>} />
        <ScrollView>
        </ScrollView>
      </View>
    )
  }
}

let styles = {
  container: {
    flex: 1,
    backgroundColor: 'white',
  }
}

export default Register;
```

Here we introduce a new `npm` package, `react-native-google-places-autocomplete`. To avoid errors, we have to install it. Remember, `npm install --save react-native-google-places-autocomplete`, and then `rnpm link` to make sure the new package is linked to our iOS and Android code.

Also, you may notice that we refactor the left button of our navigation to a stateless functional component. These are components that have no local state (i.e., no `constructor(){}` method). These components render faster than state-ful components, so we'll try to use them more frequently. If you'll notice the ` ({ navigator }) ` syntax is a way of destructuring our props, and reduces the amount of code.

Now we can add in some of the form content.

```javascript
...


...
```



