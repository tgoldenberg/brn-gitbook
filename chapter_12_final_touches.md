# Chapter 12: Final Touches

Let's review what we've accomplished before and what our app can do. 

- User authentication (login, register, logout, edit profile)
- TabBar navigation 
- Ability to create groups
- Ability to schedule and RSVP to events
- Ability to message users and view messages by conversation
- Notification system for new events and messages

To finish up, let's make a user's avatar clickable, that leads to a `Profile` component. First, which tabs have user avatars? Well, any tab that has an event or group also has a list of user avatars. So we should create a component `Profile` and route to it in our `ActivityView`, `CalendarView`, `MessagesView`, and `GroupsView`.

First let's design our `Profile` component in `application/profile/Profile.js`.

```javascript
import Icon from 'react-native-vector-icons/Ionicons';
import NavigationBar from 'react-native-navbar';
import React, { Component } from 'react';
import { ScrollView, Text, View, Image, TouchableOpacity } from 'react-native';

import Colors from '../../styles/colors';
import BackButton from '../shared/BackButton';
import {API, DEV} from '../../config';
import { globals, groupsStyles, profileStyles } from '../../styles';
import { EmptyGroupBox } from '../groups/Groups';

const styles = profileStyles;

function formatGroups(groups){
  if (groups.length % 2 === 1 ){
    return groups.concat(null);
  } else {
    return groups;
  }
};

const GroupBoxes = ({ groups}) => {
  return (
    <View style={groupsStyles.boxContainer}>
      {groups.map((group, idx) => {
        if (!group) { return <EmptyGroupBox key={idx}/>}
        return (
          <View key={idx} style={globals.flexRow}>
            <Image source={{uri: group.image}} style={groupsStyles.groupImage}>
              <View style={[groupsStyles.groupBackground, {backgroundColor: group.color,}]} >
                <Text style={groupsStyles.groupText}>{group.name}</Text>
              </View>
            </Image>
          </View>
        );
      })}
    </View>
  );
}

class Profile extends React.Component{
  constructor(props){
    super(props);
    this.goBack = this.goBack.bind(this);
    this.sendMessage = this.sendMessage.bind(this);
    this.state = {
      groups: []
    }
  }
  componentDidMount(){
    let query = {
      members: {
        $elemMatch: {userId: this.props.user.id}
      }
    };
    fetch(`${API}/groups?${JSON.stringify(query)}`)
    .then(response => response.json())
    .then(groups => this.setState({ groups }))
    .catch(err => console.log('ERR:', err))
    .done();
  }
  sendMessage(){
    this.props.navigator.push({
      name: 'Conversation',
      user: this.props.user
    });
  }
  goBack(){
    this.props.navigator.pop();
  }
  render(){
    let { user } = this.props;
    let title = `${user ? user.firstName : "User"}'s Profile`;
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          tintColor={Colors.brandPrimary}
          title={{title: title, tintColor: 'white'}}
          leftButton={<BackButton handlePress={this.goBack}/> }
        />
        <ScrollView style={[globals.flex, globals.mt1]}>
          <View style={styles.avatarContainer}>
            <Image source={{uri: user.avatar}} style={styles.avatar}/>
          </View>
          <Text style={[globals.h4, globals.centerText]}>{user.firstName} {user.lastName}</Text>
          <Text style={[globals.h5, globals.centerText]}>
            {user.location.city.long_name}, {user.location.state.long_name}
          </Text>
          <TouchableOpacity style={[globals.row, globals.mv2]} onPress={this.sendMessage}>
            <Icon name="ios-chatboxes" size={40} style={globals.mr1} color={Colors.brandPrimary}/>
            <Text style={globals.h5}>Send a Message</Text>
          </TouchableOpacity>
          <View style={globals.lightDivider}></View>
          <View style={globals.ph1}>
            <Text style={[globals.h4, globals.ma1]}>Technologies</Text>
            <Text style={[globals.h5, globals.primaryText, globals.ma1]}>{user.technologies.join(', ')}</Text>
            <Text style={[globals.h4, globals.ma1]}>Assemblies</Text>
          </View>

          <View style={globals.flex}>
            <GroupBoxes groups={this.state.groups} />
          </View>
        </ScrollView>
      </View>
    )
  }
};

export default Profile;
```

And then, to start, let's render this in our `ActivityView`.

```javascript
application/components/activity/ActivityView.js

...
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
```

Now when we click on a user avatar, we should be directed to their profile, with the option of sending a message (the `Conversation` route).

![profile](/images/chapter-13/profile-1.png)
![profile](/images/chapter-13/profile-2.png)

Now, we still have to implement this in our other routes. Simply import `Profile` and provide a route for it in each tab `Navigator`. For example, in `GroupsView.js`:

```javascript
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



### Refactoring and Preparation for Deployment

There are a couple of things we should do before immediately pushing this up to the App Store. Here's a checklist of some important things to check.

- Have you run the app on a local device? (See the appendix for more details on doing this)
- Have you either removed or prevented all `console.log` statements from running? (This will really slow down the performance of your app in production if there are logs)
- Have you checked the font sizes, layout, etc. on different screen widths and heights? (This is why it's a good idea to develop on a smaller screen size, like the iPhone 4s)
- Have you refactored the styles? (Many times there is duplication of styles which can be refactored)

Once we have gone through this checklist, we can start preparing our app for deployment. For iOS, this means creating a build, testing locally on your device, and then submitting to TestFlight. After testing with TestFlight you can then submit to the AppStore. For Android, this means creating a bundle, testing locally on the device, and then submitting to the Google Play store. We will cover these topics in the appendix.

We hope you enjoyed building `Assemblies` with us. We hope that you learned new things about mobile app development, ranging from app design to API construction to user experience. Please contact us and let us know how `Assemblies` has helped you in your own project. We'd be glad to feature products that were influenced by it!


