# Chapter 11: Building a Notification System
### Building on the shoulders of giants

All the work we have done so far has led us to this point. We now have some pretty rich data to use for the rest of our app. Here is what we have so far:

-	User city and location, full name, avatar, and technologies
-	Which groups the user has joined, as well as the events for those groups
-	Which groups share the same technologies or location as the user
-	Which events the user has RSVP’d for

With this data, we can start to construct our `Calendar` and `Activity` Views. In our `Calendar` view, we want to show a list of events, both those that the user has RSVP’d for and those that are recommended based on the user’s preferences and location. 

In our `Activity View`, we want to show the next event that the user has RSVP’d for, along with a map of its location. We also want to display any notification messages for the user. 

We haven’t yet considered how we will implement notifications. We will actually need a separate collection for these, with the following fields: 

```
message: String
createdAt: Number
type: String
data: Object
participants: Array
``` 

After each new message and event is created, we should send out a notification to the message recipient and the group members. Let's add this logic in Deployd with a callback after an event is created and after a message is created. Here's what that looks like.

```javascript
console.log('MESSAGE CREATED', this);
var text = this.text;
var senderId = this.senderId;
var recipientId = this.recipientId;
var createdAt = this.createdAt;

dpd.users.get({ id: recipientId })
.then(function(user){
    dpd.users.get({ id: senderId })
    .then(function(currentUser){
      console.log('SENDER', user.firstName, currentUser.firstName);
      dpd.notifications.post({
        type: 'Message',
        participants: [{userId: recipientId, seen: false}],
        message: 'New message from ' + currentUser.firstName,
        createdAt: new Date().valueOf(),
        data: {
            user: currentUser
        }
      })  
    })
})

dpd.conversations.get({
    $or: [
        {user1Id: senderId, user2Id: recipientId},
        {user2Id: senderId, user1Id: recipientId}
    ]
})
.then(function(data){
    console.log('DATA', data);
    if (data.length) {
        dpd.conversations.put(data[0].id, {
            lastMessageText: text,
            lastMessageDate: createdAt
        });
    } else {
        dpd.conversations.post({
            user1Id: senderId,
            user2Id: recipientId,
            lastMessageText: text,
            lastMessageDate: createdAt
        });
    }
});
...
```

Now create a new message and see that a notification gets created.
![notification 1](Screen Shot 2016-07-18 at 8.43.37 PM.png)

Now we can do the same thing with new events. Let's create a hook on the `POST` event of the `events` collection.

```javascript
var evt = this;
var groupId = evt.groupId;

dpd.groups.get({id: groupId})
.then(function(group){
    console.log('New event for group', group.name);
    let members = group.members;
    dpd.notifications.post({
        type: 'Event',
        message: 'Your group ' + group.name + ' has a new event',
        createdAt: new Date().valueOf(),
        participants: members.map(function(m){
            return {
                seen: false,
                userId: m.userId
            }
        }),
        data: {
          group: group,
          event: evt
        }
    })
});

```
![deployd event](Screen Shot 2016-07-18 at 8.52.19 PM.png)
![new event](Screen Shot 2016-07-18 at 8.53.43 PM.png)

![notification](Screen Shot 2016-07-18 at 8.51.20 PM.png)

Now if we create another event, we will see a new notification has been created. Now it should be straightforward to render these in our `ActivityView`. We also want to load the next event that a user is signed up for, or if there is none, the next event nearby.

Let's edit `ActivityView.js`.

```javascript
import React, { Component } from 'react';
import { Navigator } from 'react-native';
import { extend } from 'underscore';

import Activity from './Activity';
import Conversation from '../messages/Conversation';
import Event from '../groups/Event';
import { API, DEV } from '../../config';
import { globals } from '../../styles';

class ActivityView extends Component{
  constructor(){
    super();
    this.state = {
      nextEvents      : [],
      notifications   : [],
    }
  }
  componentDidMount(){
    this._loadNotifications();
  }
  _loadNotifications(){
    let query = {
      participants: {
        $elemMatch: {
          userId: this.props.currentUser.id
        }
      }
    };
    fetch(`${API}/notifications?${JSON.stringify(query)}`)
    .then(response => response.json())
    .then(notifications => this._loadNextEvents(notifications))
    .catch(err => {})
    .done();
  }
  _loadNextEvents(notifications){
    this.setState({ notifications });
    let dateQuery = { end: { $gt: new Date().valueOf() }};
    let query = {
      $or: [
        extend(dateQuery, { going: { $elemMatch: { $eq: this.props.currentUser.id }}}),
        extend(dateQuery, { 'location.city.long_name': this.props.currentUser.location.city.long_name })
      ],
      $limit: 1,
      $sort: { createdAt: 1 }
    };
    fetch(`${API}/events?${JSON.stringify(query)}`)
    .then(response => response.json())
    .then(nextEvents => this.setState({ nextEvents }))
    .catch(err => {})
    .done();
  }
  render(){
    return (
      <Navigator
        style={globals.flex}
        initialRoute={{ name: 'Activity' }}
        renderScene={(route, navigator) => {
          switch(route.name){
            case 'Activity':
              return (
                <Activity
                  {...this.props}
                  {...route}
                  {...this.state}
                  navigator={navigator}
                />
            );
            case 'Event':
              return (
                <Event
                  {...this.props}
                  {...route}
                  {...this.state}
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
          }
        }}
      />
    )
  }
}

export default ActivityView;

```

Let's review:
- Notice how we import the `Event` and `Conversation` components. This is so we can directly route the user to a particular event or conversation from the notifications.
- In our `componentDidMount` method we fetch notifications from the database, based on the nested field of `userId` in the notification `participants` value. We then fetch the next event that the user is attending that is closest to the current date, or in absence, the next event by a group that the user is part of.
- The rest of the component should be pretty straightforward. It is nice that we're able to reuse `Event` and `Conversation` here.

Now that we've set up our routing, let's render the fetched notifications and next event in `Activity.js`

```javascript
import moment from 'moment';
import Icon from 'react-native-vector-icons/Ionicons';
import NavigationBar from 'react-native-navbar';
import React, { Component } from 'react';
import {
  ScrollView,
  View,
  Text,
  InteractionManager,
  TouchableOpacity,
  MapView,
} from 'react-native';

import Colors from '../../styles/colors';
import { globals, activityStyles } from '../../styles';

const styles = activityStyles;

const ActivityMap = ({ event, ready }) => {
  if (! ready || ! event) { /* render empty map if not ready or no event */
    return <View style={globals.map} />
  }
  const mapRegion = {
    latitude: event.location.lat,
    longitude: event.location.lng,
    latitudeDelta: 0.01,
    longitudeDelta: 0.01,
  };
  return (
    <MapView
      style={globals.map}
      region={mapRegion}
      annotations={[{latitude: mapRegion.latitude, longitude: mapRegion.longitude}]}
    />
  )
};

const Notification = ({ notification, handlePress }) => (
  <TouchableOpacity style={[globals.flexRow, globals.ph1]} onPress={() => handlePress(notification)}>
    <View style={globals.flex}>
      <View style={globals.flexRow}>
        <View style={styles.circle}/>
        <View style={[globals.textContainer, globals.pv1]}>
          <Text style={styles.h4}>New {notification.type}</Text>
        </View>
        <Text style={styles.h5}>{moment(new Date(new Date(notification.createdAt))).fromNow()}</Text>
      </View>
      <View style={globals.flex}>
        <Text style={styles.messageText}>{notification.message}</Text>
      </View>
    </View>
    <View>
      <Icon name='ios-arrow-forward' color='#777' size={25} />
    </View>
  </TouchableOpacity>
)

class Activity extends Component{
  constructor(){
    super();
    this.handleNotificationPress = this.handleNotificationPress.bind(this);
    this.visitEvent = this.visitEvent.bind(this);
    this.state = {
      ready: false
    }
  }
  componentDidMount(){
    InteractionManager.runAfterInteractions(() => {
      this.setState({ ready: true })
    });
  }
  visitEvent(){
    let event = this.props.nextEvents[0];
    this.props.navigator.push({
      name: 'Event',
      event,
    })
  }
  handleNotificationPress(notification){
    if (notification.type === 'Message'){
      this.props.navigator.push({
        name: 'Conversation',
        ...notification.data
      })
    } else if (notification.type === 'Event'){
      this.props.navigator.push({
        name: 'Event',
        ...notification.data
      })
    }
  }
  render() {
    let upcomingEvent = this.props.nextEvents.length ? this.props.nextEvents[0] : null;
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          title={{ title: 'Activity', tintColor: 'white' }}
          tintColor={Colors.brandPrimary}
        />
        <ScrollView>
          <TouchableOpacity onPress={this.visitEvent}>
            <View style={[globals.flexRow, globals.mb1]}>
              <Text style={styles.h4}>Next Assembly: </Text>
              <Text style={globals.primaryText}>{ upcomingEvent && upcomingEvent.name }</Text>
            </View>
            <Text style={[styles.dateText, globals.mb1]}>{upcomingEvent && moment(new Date(upcomingEvent.start)).format('dddd MMM Do, h:mm a')}</Text>
          </TouchableOpacity>
          <ActivityMap event={upcomingEvent} ready={this.state.ready} />
          <View>
            <Text style={[styles.h4, globals.mv1]}>Notifications</Text>
            <View style={globals.divider}/>
            <View style={globals.flex}>
              {this.props.notifications.map((n, idx) => (
                <View key={idx} style={globals.flex}>
                  <Notification notification={n} handlePress={this.handleNotificationPress}/>
                </View>
              ))}
              <View style={styles.emptySpace} />
            </View>
          </View>
        </ScrollView>
      </View>
    );
  }
};

export default Activity;


```

Let's go over this:
- We use a new module here, the `InteractionManager` library. Remember when we talked about tweaking the animation for `Navigator`? Well, this allows us to do it. We set a field `ready` to `false` in our initial state, and then set to `true` once the navigation animation is finished. This ensures that the JavaScript thread doesn't get block, which can cause animation frames to drop (which looks choppy). Thus, we get a super smooth transition, and then our map renders right after.
- Remember how we saved a `data` field in our notifications? That was so that we could route directly to `Event` and `Conversation` from our activity screen. Since we have all the data we need, our user is able to go directly to the conversation or event and either write a message or join the event.

Let's make a commit here. Our app is shaping into something actually quite nice!

![notifications](/images/chapter-11/notifications-1.png)
![notifications](/images/chapter-11/notifications-2.png)

[Commit 24](https://github.com/buildreactnative/assemblies-tutorial/tree/01a9fc1b4788b4c4c11e7f9c645363f4d12cd075) - "Render the Activity View"

