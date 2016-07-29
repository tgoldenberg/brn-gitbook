In the last chapter we left off with a nice prototype to build from. Now it’s time to flesh out the app with real dynamic data. This means we’ll be needing an **API** endpoint to send requests to.  The word **API** stands for *application programming interface* – it's the tool we use to interact with our data, via **HTTP** requests.

## Setting up Deployd

One of the first things we'll need from our API is the ability to manage user accounts. Users should be able to create an account, to login, logout, etc. While there are several good options for user management in frameworks such as [**Rails**](http://rubyonrails.org/) and [**Meteor**](https://www.meteor.com/), we will be using the Node package [**Deployd**](https://github.com/deployd/deployd).

**Deployd** abstracts the API creation process for us through an easy-to-use command line tool and intuitive interface. Since we want to focus on the mobile development aspects of React Native in this tutorial, we thought it would be an excellent tool to use. There are, of course, situations where a more configurable API would be required. But we found that **Deployd** meets our needs for the most part.

Getting the API started is easy. As per **Deployd**’s [documentation](http://docs.deployd.com/guides/), we make sure that the package is installed globally on our machine.

```
npm install –g deployd
```

Check to make sure that the package was installed by running. `dpd –version`. You should see something like this.

![deployd](/images/chapter-5/deployd-1.png)

Next, we can create our API with a simple command. `dpd create api`.
You should see instructions for opening and running the newly created API. If we `cd` into the created folder and type the command `dpd`, our API will be up and running.

![deployd](/images/chapter-5/deployd-2.png)

If you open your browser to `localhost:2403/dashboard`, you can see a visual representation of the API. We can use the dashboard to add **resources**, and it is easy to edit the fields of our resources as well.

We know that we need a **users** collection, so let’s start by creating that. By selecting the **Users Collection** option under *Resources*, we can see that **Deployd** creates a default collection with the fields **id**, **username**, and **password**. **Deployd** requires these three fields, even if you don’t plan on having a username for your users. One way around this is to designate the username field for the user’s email. The only criteria is that this field be unique for all users.

We can add other fields to our user collection. How about location? We will want to know where our users live, in order to suggest other assemblies. We will also want their first and last names, a URL for their avatar, and what technologies they are interested in, in order to filter the assemblies we suggest for them. Here are all the fields  we'll be adding to our `users` collection, along with their data type.

```JavaScript
username    String
id          String
password    String
firstName   String
lastName    String
location    Object
technologies Array
avatar      String
```

![User Fields](/images/chapter-5-user-accounts-part-1/user-fields.png "User Fields")


Now in our **Landing** component, let’s add two buttons in place of the previous **Go to dashboard** button - **Login** and **Signup**. We'll also add the methods `visitLogin` and `visitRegister`.
```JavaScript
/* application/components/Landing.js */

import React, { Component } from 'react';
import {
  Text,
  TouchableOpacity,
  Image,
  View
} from 'react-native';

import Icon from 'react-native-vector-icons/MaterialIcons';
import Colors from '../styles/colors';
import { landingStyles } from '../styles';
import { globals } from '../styles';
const BackgroundImage = 'https://s3-us-west-2.amazonaws.com/assembliesapp/welcome%402x.png';
const Logo = 'https://s3-us-west-2.amazonaws.com/assembliesapp/logo.png';
const styles = landingStyles;

class Landing extends Component{
  constructor(){
    super();
    this.visitLogin = this.visitLogin.bind(this);
    this.visitRegister = this.visitRegister.bind(this);
  }
  visitLogin(){
    this.props.navigator.push({ name: 'Login' })
  }
  visitRegister(){
    this.props.navigator.push({ name: 'Register' })
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
          style={[globals.button, globals.inactive, styles.loginButton]}
          onPress={this.visitLogin}
        >
          <Icon name='lock' size={36} color={Colors.brandPrimary} />
          <Text style={[globals.buttonText, globals.primaryText]}>
            Login
          </Text>
        </TouchableOpacity>
        <TouchableOpacity
          style={globals.button}
          onPress={this.visitRegister}
        >
          <Icon name='person' size={36} color='white' />
          <Text style={globals.buttonText}>
            Create an account
          </Text>
        </TouchableOpacity>
      </View>
    );
  }
};

export default Landing;

```

![landing](/images/chapter-5/landing-1.png)

Let's point out a few things:

- Notice how we use an array of styles for some elements. React Native makes it very easy to design modular styles, and part of that is support for arrays of styles. The last style in the array overrides any previous conflicting fields in the previous style objects.
- Also notice how our `render` method is very slim. We could easily include the full methods `visitLogin` and `visitRegister` in the code under render but this is less readable than factoring out these functions. Also remember that we have to bind them to the class in the **constructor** function.

Next, we have to create the routes for both **Register** and **Login** and create forms for our users to create an account and sign in.  In `index.ios.js` (depending on what you are building), add the following lines. Don’t forget to import the referenced files at the top of the file.
```javascript
/* index.ios.js */
…
import Register from './application/components/accounts/Register';
import Login from './application/components/accounts/Login';
…
case 'Register':
  return (
    <Register navigator={navigator} />
);
case 'Login':
  return (
    <Login navigator={navigator} />
);
…
```

Now let’s create our **Login** and **Register** components. To test out the routing, let’s just start with some minimal components.

```javascript
/* application/components/accounts/Register.js */

import React, { Component } from 'react';
import {
  View,
  Text
} from 'react-native';

import NavigationBar from 'react-native-navbar';
import Icon from 'react-native-vector-icons/Ionicons';
import BackButton from '../shared/BackButton';
import Colors from '../../styles/colors';
import { globals } from '../../styles';

class Register extends Component{
  constructor(){
    super();
    this.goBack = this.goBack.bind(this);
  }
  goBack(){
    this.props.navigator.pop();
  }
  render(){
    let titleConfig = { title: 'Register', tintColor: 'white' };
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          title={titleConfig}
          tintColor={Colors.brandPrimary}
          leftButton={<BackButton handlePress={this.goBack}/>}
        />
        <View style={globals.flexCenter}>
          <Text style={globals.h2}>
            Register
          </Text>
        </View>
      </View>
    )
  }
};

export default Register;

```

```JavaScript
/* application/components/accounts/Login.js */

import React, { Component } from 'react';
import {
  View,
  Text
} from 'react-native';

import NavigationBar from 'react-native-navbar';
import Icon from 'react-native-vector-icons/Ionicons';
import BackButton from '../shared/BackButton';
import Colors from '../../styles/colors';
import { globals } from '../../styles';

class Login extends Component{
  constructor(){
    super();
    this.goBack = this.goBack.bind(this);
  }
  goBack(){
    this.props.navigator.pop();
  }
  render(){
    let titleConfig = { title: 'Login', tintColor: 'white' };
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          title={titleConfig}
          tintColor={Colors.brandPrimary}
          leftButton={<BackButton handlePress={this.goBack}/>}
        />
        <View style={globals.flexCenter}>
          <Text style={globals.h2}>
            Login
          </Text>
        </View>
      </View>
    )
  }
};

export default Login;


```

You should see something like this screenshot when you click to either the `Login` or `Register` routes.

![login](/images/chapter-5/login-1.png)

Now is probably a good time to commit our changes.

***
[![GitHub logo](/images/github-logo.png "GitHub logo") Commit 1: "Add basic Login and Register routes"]()

### Building a Login and Register Form

Now that our routing works, let’s build a form for our users to fill out. For login, we’ll only need an email and password. For registration, however, we’ll need more data. Let’s start with the login form, since it’s a bit easier.

Now, there are several issues that we have to deal with when building forms in React Native. You may be used to building forms for the web, using the `<form>` element and maybe the `onSubmit` callback in React for the web.  In React Native, we’ll primarily be using components such as `<TextInput/>`, `<PickerIOS/>`, and so on. We’ll trigger our `handleSubmit` callback when the user presses the submit button. There are other issues as well with mobile forms, such as interference with the phone's keyboard. For now, let's just construct a simple form.

```javascript
/* application/components/accounts/Login.js */
import React, { Component } from 'react';
import {
  Text,
  View,
  ScrollView,
  TextInput,
  TouchableOpacity
} from 'react-native';
import { extend } from 'underscore';

import BackButton from '../shared/BackButton';
import Colors from '../../styles/colors';
import NavigationBar from 'react-native-navbar';
import { globals, formStyles } from '../../styles';

const styles = formStyles;

class Login extends Component{
  constructor(){
    super();
    this.loginUser = this.loginUser.bind(this);
    this.goBack = this.goBack.bind(this);
    this.changePassword = this.changePassword.bind(this);
    this.changeEmail = this.changeEmail.bind(this);
    this.state = {
      email           : '',
      password        : '',
      errorMsg        : '',
    };
  }
  loginUser(){
    /* TODO: login user with username and password */
  }
  goBack(){
    this.props.navigator.pop();
  }
  changeEmail(email){
    this.setState({ email })
  }
  changePassword(password){
    this.setState({ password })
  }
  render(){
    let titleConfig = { title: 'Login', tintColor: 'white' };
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          leftButton={<BackButton handlePress={this.goBack} />}
          title={titleConfig}
          tintColor={Colors.brandPrimary}
        />
        <ScrollView style={styles.container}>
          <Text style={styles.h3}>
            Login with your email and password.
          </Text>
          <Text style={styles.h4}>
            Email
          </Text>
          <View style={styles.formField}>
            <TextInput
              autoFocus={true}
              returnKeyType="next"
              onSubmitEditing={() => this.password.focus()}
              onChangeText={this.changeEmail}
              keyboardType="email-address"
              autoCapitalize="none"
              maxLength={140}
              placeholderTextColor={Colors.copyMedium}
              style={styles.input}
              placeholder="Your email address"
            />
          </View>
          <Text style={styles.h4}>Password</Text>
          <View style={styles.formField}>
            <TextInput
              ref={(el) => this.password = el }
              returnKeyType="next"
              onChangeText={this.changePassword}
              secureTextEntry={true}
              autoCapitalize="none"
              maxLength={140}
              placeholderTextColor={Colors.copyMedium}
              style={styles.input}
              placeholder="Your password"
            />
          </View>
          <View style={styles.errorContainer}>
            <Text style={styles.errorText}>
              {this.state.errorMsg}
            </Text>
          </View>
        </ScrollView>
        <TouchableOpacity style={styles.submitButton} onPress={this.loginUser}>
          <Text style={globals.largeButtonText}>Login</Text>
        </TouchableOpacity>
      </View>
    )
  }
}

export default Login;
```

![login](/images/chapter-5/login-2.png)

Let's go over a few things here:

- We store the values of **password** and **email** in our component state. When the user inputs a value into either the password field or email field, we update this value with the methods `changeEmail` and `changePassword`. 
- As before, we use a method `goBack` to go to the previous route
- We use the **TextInput** component to receive user information. Here are some of the relevant properties that we use to customize the inputs:
  - **autoFocus**: we set the first input to `true` since we want to make the process easier
  - **returnKeyType**: the text in the return key of the keyboard
  - **onSubmitEditing**: when the user hits **next**, we focus the next input to make the process faster
  - **keyboardType**: we can specify whether the input is for email, password, etc.
  - **secureTextEntry**: this masks the input for more security when using passwords, etc. 
- Also notice that we use a property called **ref**. We use this as a callback to later reference the element. In our password input, we use the code - 
``` 
ref={(el) => this.password = el }
``` 
  This is so that we can later call `this.password.focus()`. It is also possible to set **ref** with a string, such as `ref='password'`, and reference it as `this.refs.password`. However, the core React team recommends using a function. To learn more about the use **ref** check out the [docs](https://facebook.github.io/react/docs/more-about-refs.html).
- Finally, notice that a `loginUser` function is invoked when our button is pressed. Currently this function doesn't do anything. We will have to add our login functionality there.


Let’s make a commit here, before making the login calls to our API.

***
[![GitHub logo](/images/github-logo.png "GitHub logo") Commit 1: "Add basic Login component"]()

### Adding the API to Login

Now we’re ready to collect our user’s email and password and create a session with our API. In production, we would be pointing our requests to an API hosted on a server such as [DigitalOcean](https://www.digitalocean.com/), [AWS](https://aws.amazon.com/), or [Heroku](https://www.heroku.com/). While in development, it’s enough to point to our `localhost` with the port that is running Deployd. We can set a config file with our API endpoint set to `http://localhost:2403`, and when we’re ready for production, we can change the endpoint to the URL of the server that is hosting our API.

Let’s create a file **application/config/index.js** and add the following lines:

```javascript
export const DEV = true;
export const API = ‘http://localhost:2403’;
```

Now we can refer to our endpoint as `API` as long as we import from  the config file -- 
```
import { API, DEV } from '../../config';
```

Notice how we don't have to specify the **index.js** file, as it is automatically inferred. 

Now, let’s think about what we want to do.  With **Deployd**, we first have to call the **/users/login** endpoint with the username and password. Our API will return a session id (or cookie). From here, we will want to get all of the user’s information, so we will have to call to **Deployd** to fetch the user information with the session id. This would be **/users/me**, once we’re logged in.

If the user is not allowed to log in, we want to display a generic error message, to the effect that the email and password combination was invalid. Giving too much information in a login form can be a bad idea.

To make our API calls, we’ll be using **fetch**. React Native supports the **fetch** API, which was recently introduced into most browsers, and it gives us a simple way of sending HTTP requests. For more information about **fetch** and about promises in general, check out the [documentation](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API).

```JavaScript
/* application/components/accounts/Login.js */
...
loginUser(){
  fetch(`${API}/users/login`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      username: this.state.email,
      password: this.state.password
    })
  })
  .then(response => response.json())
  .then(data => {
    if (response.status === 401){
      this.setState({ errorMsg: 'Email or password was incorrect.' });
    } else {
      fetch(`${API}/users/me`, {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
          'Set-Cookie': `sid=${sid}`
        }
      })
      .then(response => response.json())
      .then(user => console.log('USER', user))
      .catch(err => {
        this.setState({ errorMsg: 'Connection error.'})
      })
      .done();
    }
  })
  .catch(err => {
    this.setState({ errorMsg: 'Connection error.'})
  })
  .done();
}
...
```
Let's go over this: 
- When we use **fetch**, it takes three arguments: **method**, **headers**, and **body**. The default method is **GET**, but **PUT**, **POST**, and **DELETE** are also supported. As for headers, the default is `{ 'Content-Type': 'application/json' }`. This specifies that we are dealing with **JSON** data. 
- So, first we send a **POST** request with our username and password information. This gives us a response with a **status** and **id** property. If the status is 401, it means our request failed, so we display an error message to the user.
- If the request was successful, we then make a **GET** request to retreive all of the user's information. We do this by sending a special header `Set-Cookie` which passes the session ID that was received from the first request.
- If the user information retrieval works, we `console.log` our **user** object. Otherwise, we set the error message to show that there was a connection error.

Wow, that's a lot! We can surely refactor this, since there is some duplication of code, and the method is very lengthy. One thing we can do is to refactor our `{ 'Content-Type': 'application/json' }` header to our **application/fixtures/index.js** file. In fact, we've already included **Headers** in the fixtures file so you can just import it before using. As for the login component, we'll also create separate class methods for setting the error message and for each separate **fetch** call. Here's our refactored version.

```javascript 
/* application/components/accounts/Login.js */
import { API, DEV } from '../../config';
import { Headers } from '../../fixtures';
...
loginUser(){
  if (DEV) { console.log('Logging in...'); }
  fetch(`${API}/users/login`, {
    method: 'POST',
    headers: Headers,
    body: JSON.stringify({
      username: this.state.email,
      password: this.state.password
    })
  })
  .then(response => response.json())
  .then(data => this.loginStatus(data))
  .catch(err => this.connectionError())
  .done();
}
loginStatus(response){
  if (response.status === 401){
    this.setState({ errorMsg: 'Email or password was incorrect.' });
  } else {
    this.fetchUserInfo(response.id)
  }
}
fetchUserInfo(sid){
  fetch(`${API}/users/me`, { headers: extend(Headers, { 'Set-Cookie': `sid=${sid}`}) })
  .then(response => response.json())
  .then(user => this.updateUserInfo(user))
  .catch(err => this.connectionError())
  .done();
}
updateUserInfo(user){
  if (DEV) { console.log('Logged in user:', user); }
  this.props.updateUser(user);
  this.props.navigator.push({ name: 'Dashboard' })
}
connectionError(){
  this.setState({ errorMsg: 'Connection error.'})
}
...

```

The only thing we have left to do is pass the new user information up to the main component in **index.ios.js**. We do this by making **user** a state object and defining a method `updateUser` that we pass as a property to **Login**.

```JavaScript
/* index.ios.js */
...
  constructor(){
    super();
    this.updateUser = this.updateUser.bind(this);
    this.state = {
      user: null
    };
  }
  updateUser(user){
    this.setState({ user: user });
  }
  ...
  case 'Dashboard':
    return (
      <Dashboard
        updateUser={this.updateUser}
        navigator={navigator}
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
  ...
```

Let's go over what's happening one more time:
- When our user submits their username and password, we call our API to start a new session.
- Once we receive the session ID, we call the API again to retreive our user information. We then pass this information to our main **Navigator** component.
- When the main **Navigator** receives the new user data, it sets the new data in its state, and then navigates to the Dashboard with our logged in user, that simple.
 
Now that we have our login functionality working, try logging into the app!

When you try logging in, you should see an error message like this. This is because we forgot to create a user! Luckily, Deployd makes this easy to get started.

![login](/images/chapter-5/login-3.png)

If we open up the dashboard for Deployd in our browser window (at `localhost:2403/dashboard`), we can actually manually insert a user. Open the **users** collection, and select the **data** tab. Now start typing, and you should see the fields fill up. When filling in values that expect arrays or objects, make sure to use double quotes and not single quotes. Here is what our screen looks like. Feel free to use the fields in our **currentUser** fixture for things like **location**, **technologies**, and **avatar**. 



Now when we login with the correct email and password, we should get a response with the user information, and we can then redirect to the dashboard. Voila! 

![user](/images/chapter-5/deployd-user-1.png)
![user](/images/chapter-5/deployd-user-2.png)
![user](/images/chapter-5/deployd-user-3.png)

Finally, notice that we add `if (DEV) { }` clauses to our console statements. This is because console logs can really slow down app performance in production. By setting `DEV` to false, we eliminate the need to remove individual console statements once we're ready to deploy our app.

### Real Profile View

Now that we have real user data, we can start to flesh out parts of our app. Our profile view, for example, should be pretty easy to update. We just have to ensure that the user data that is passed to the dashboard component gets passed to our profile view. Let's see what that looks like.

```JavaScript
/* application/components/profile/ProfileView.js */
...
// import { currentUser } from '../../fixtures';
...
 class ProfileView extends Component{
  render() {
    let { currentUser } = this.props;
    return (
...
```
```JavaScript
/* application/components/Dashboard.js */
  ...
class Dashboard extends Component{
  constructor(){
    super();
    this.state = {
      selectedTab: 'Activity'
    };
  }
  render(){
    let { user } = this.props;
  ...
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
      <ProfileView currentUser={user}/>
    </TabBarItemIOS>
  </TabBarIOS>
  ...
```

Now our profile view should have real dynamic data. If we log in as a different user, we'll see entirely different information. Pretty cool!

![profile](/images/chapter-5/profile-1.png)

Now that we've allowed our users to log in and fleshed out our profile view with real data, we still need to allow users to create their account, and logout. That will come next in Chapter 6.

`Git commit -am "Create user login"`

[Commit - "Create user login"]()
