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
application/components/activity/ActivityView.js
...
import Conversation from '../messages/Conversation';
...
  case 'Profile':
    return (
    <Profile
      {...this.props}
      {...route}
      navigator={navigator}
    />
  );
  case 'Conversation':
    return (
      <Conversation
        {...this.props}
        {...route}
        navigator={navigator}
      />
  );
...
```

Then in `Event.js`:
```javascript
...
  <View style={styles.infoContainer}>
    <Text style={styles.h2}>Going <Text style={styles.h4}>{event.going.length}</Text></Text>
    {eventMembers.map((member, idx) => (
      <TouchableOpacity
        key={idx}
        onPress={() => {
          if (user.id !== currentUser.id){
            navigator.push({
              name: 'Profile',
              user: member
            })
          }
        }}
        style={styles.memberContainer}>
        <Image source={{uri: member.avatar}} style={styles.avatar}/>
        <View style={styles.memberInfo}>
          <Text style={styles.h5}>{member.firstName} {member.lastName}</Text>
        </View>
      </TouchableOpacity>
    ))}
  </View>
...
```

```javascript
application/components/profile/Profile.js
import _ from 'underscore';
import moment from 'moment';
import Icon from 'react-native-vector-icons/Ionicons';
import React, { Component } from 'react';
import {
  ScrollView,
  StyleSheet,
  Text,
  View,
  Image,
  TouchableOpacity,
  Dimensions,
} from 'react-native';

import Colors from '../../styles/colors';
import NavigationBar from 'react-native-navbar';
import {API, DEV} from '../../config';
import LeftButton from '../accounts/LeftButton';

let { width: deviceWidth, height: deviceHeight } = Dimensions.get('window');

const EmptyGroupBox = () => (
  <View style={styles.groupsContainer}>
    <View style={styles.groupImage}>
      <View style={[styles.group, {backgroundColor: Colors.inactive,}]} />
    </View>
  </View>
);

const GroupBoxes = ({ groups }) => (
  <View style={{justifyContent: 'center', flexDirection: 'row', flexWrap: 'wrap'}}>
    {groups.map((group, idx) => {
      if (!group) { return <EmptyGroupBox key={idx}/>}
      return (
        <TouchableOpacity key={idx} style={styles.groupsContainer}>
          <Image source={{uri: group.image}} style={styles.groupImage}>
            <View style={[styles.group, {backgroundColor: group.color,}]} >
              <Text style={styles.groupText}>{group.name}</Text>
            </View>
          </Image>
        </TouchableOpacity>
      )
    })}
  </View>
)

class Profile extends React.Component{
  constructor(props){
    super(props);
    this.sendMessage = this.sendMessage.bind(this);
    this.state = {
      groups: []
    }
  }
  componentDidMount(){
    let { user } = this.props;
    let query = {
      members: {
        $elemMatch: {userId: user.id}
      }
    };
    fetch(`${API}/groups?${JSON.stringify(query)}`)
    .then(response => response.json())
    .then(groups => this.setState({ groups }))
    .catch(err => console.log('ERR:', err))
    .done();
  }
  sendMessage(){
    let { user, navigator } = this.props;
    navigator.push({
      name: 'Conversation',
      user
    })
  }
  render(){
    let {user, currentUser, navigator} = this.props;
    let {groups} = this.state;
    if (groups.length % 2 === 1) { groups.push(null) }
    let titleConfig = {title: `${user ? user.firstName : 'User'}'s Profile`, tintColor: 'white'};

    return (
      <View style={styles.container}>
        <NavigationBar
          tintColor={Colors.brandPrimary}
          title={titleConfig}
          leftButton={<LeftButton handlePress={() => navigator.pop()}/> }
        />
        <ScrollView style={styles.profileContainer}>
          <View style={{height: 120, alignItems: 'center'}}>
            <Image source={{uri: user.avatar}} style={styles.avatar}/>
          </View>
          <Text style={styles.username}>{user.firstName} {user.lastName}</Text>
          <Text style={styles.location}>{user.location.city.long_name}, {user.location.state.long_name}</Text>
          <TouchableOpacity style={styles.newMessageContainer} onPress={this.sendMessage}>
            <Icon name="ios-chatboxes" size={40} style={styles.chatBubble} color={Colors.brandPrimary}/>
            <Text style={styles.sendMessageText}>Send a Message</Text>
          </TouchableOpacity>
          <View style={styles.break}></View>
          <Text style={styles.technologies}>Technologies</Text>
          <Text style={styles.technologyList}>{user.technologies.join(', ')}</Text>
          <Text style={styles.technologies}>Assemblies</Text>
          <View style={{flex: 1}}>
            <GroupBoxes groups={this.state.groups} />
          </View>
        </ScrollView>
      </View>
    )
  }
}

let styles = StyleSheet.create({
  groupsContainer: {
    flexDirection: 'row'
  },
  profileContainer: {
    flex: 1,
    flexDirection: 'column',
    paddingTop: 15,
  },
  backButton: {
    position: 'absolute',
    left: 20,
  },
  backButtonText: {
    color: 'white',
  },
  avatar: {
    width: 120,
    height: 120,
    borderRadius: 60,
    padding: 20,
  },
  username: {
    textAlign: 'center',
    fontSize: 24,
    fontWeight: '300',
    padding: 8,
  },
  group: {
    opacity: 0.9,
    flex: 1,
    padding: 15,
    height: (deviceWidth / 2) - 25,
    width: (deviceWidth / 2) - 25,
  },
  groupsContainer: {
    flexDirection: 'row',
    paddingHorizontal: 5,
  },
  groupImage: {
    height: (deviceWidth / 2) - 25,
    width: (deviceWidth / 2) - 25,
    opacity: 0.8,
    margin: 5,
  },
  location: {
    textAlign: 'center',
    fontSize: 19,
    fontWeight: '300',
  },
  newMessageContainer:{
    flexDirection: 'row',
    justifyContent: 'center',
    alignItems: 'center',
  },
  groupText: {
    color: 'white',
    fontSize: 20,
    position: 'absolute',
    fontWeight: '500',
  },
  assemblies:{

  },
  break:{
    marginHorizontal: 10,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
    height: 1,
    width: deviceWidth - 20,
  },
  chatBubble:{
    padding: 10,
  },
  sendMessageText: {
    fontSize: 18,
    padding: 5,
  },
  technologies:{
    textAlign: 'left',
    fontSize: 22,
    paddingHorizontal: 20,
    paddingBottom: 4,
    paddingTop: 20,
  },
  technologyList:{
    textAlign: 'left',
    fontSize: 18,
    fontWeight: 'bold',
    color: Colors.brandPrimary,
    paddingHorizontal: 20,
    paddingVertical: 4,
  },

  inputBox: {
    height: 60,
    backgroundColor: '#F3EFEF',
    flexDirection: 'row',
    marginBottom: 50,
  },

  container: {
    flex: 1,
    backgroundColor: 'white'
  },
  header: {
    height: 70,
    backgroundColor: Colors.brandPrimary,
    justifyContent: 'center',
    alignItems: 'center',
    flexDirection: 'row',
    padding: 20,
  },
  headerText: {
    color: 'white',
    fontSize: 22,
  },
})

export default Profile;


```

Now we get directed to a user's profile when we press on another user. From there we can start a conversation. Notice that we prevent this from happening if the other user is us! Now all we have to do is do the same in `CalendarView` and `GroupsView`.

```javascript
application/components/groups/GroupsView.js
...
import Conversation from '../messages/Conversation';
import Profile from '../profile/Profile';
...
case 'Profile':
  return (
    <Profile
      {...this.props}
      {...route}
      navigator={navigator}
    />
);
case 'Conversation':
  return (
    <Conversation
      {...this.props}
      {...route}
      navigator={navigator}
    />
);
...

```
and the same for `application/components/calendar/CalendarView.js`.

We can also add the same functionality for the members that are rendered as part of `Group.js`.

```javascript
{group.members.map((member, idx) => {
  if (DEV) {console.log('MEMBER', member)}
  let user = find(users, (u => u.id === member.userId))
  let isOwner = member.role === 'owner';
  let isAdmin = member.role === 'admin';
  let status = isOwner ? 'owner' : isAdmin ? 'admin' : 'member'
  if ( ! user ) { return; }
  return (
    <TouchableOpacity key={idx} style={styles.memberContainer} onPress={() => {
        this.props.navigator.push({
          name: 'Profile',
          user: user
        })
      }}>
      <Image source={{uri: user.avatar}} style={styles.avatar}/>
      <View style={styles.memberInfo}>
        <Text style={styles.h5}>{user.firstName} {user.lastName}</Text>
        <Text style={styles.h4}>{status}</Text>
      </View>
    </TouchableOpacity>
  )
})}

```

## 12.3 Refactoring and Preparation for Deployment

There are a couple of things we should do before immediately pushing this up to the App Store. Here's a checklist of some important things to check.

- Have you run the app on a local device? (See the appendix for more details on doing this)
- Have you either removed or prevented all `console.log` statements from running? (This will really slow down the performance of your app in production if there are logs)
- Have you checked the font sizes, layout, etc. on different screen widths and heights? (This is why it's 
