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


```

We also have to add the `updateTab` method in our `Dashboard.js` file.
```javascript
...
export default class Dashboard extends Component{
  constructor(props){
    super(props);
    this.logout = this.logout.bind(this);
    this.updateTab = this.updateTab.bind(this);
    this.state = {
      selectedTab: 'Activity',
    };
  }
  updateTab(type){
    switch(type){
      case 'Message':
        this.setState({ selectedTab: 'Messages'});
        break;
      case 'Event':
        this.setState({ selectedTab: 'Groups'});
        break;
    }
  }
  ...
  
render() {
  let { selectedTab } = this.state;
  let { currentUser } = this.props;
  return (
    <TabBarIOS style={styles.outerContainer}>
      <TabBarItemIOS
        title='Activity'
        selected={ selectedTab == 'Activity' }
        iconName='ios-pulse'
        onPress={() => this.setState({ selectedTab: 'Activity' })}
      >
        <ActivityView {...this.props} updateTab={this.updateTab}/>
      </TabBarItemIOS>
...
```

Now when our user clicks on a new message notifications, they'll be directed to the `Mesages` tab, and when they receive a new event notification, they'll be directed to the `Groups` tab. Also, when they click on the next event, we will navigate to the `Event` screen.

![activity](Screen Shot 2016-07-19 at 8.58.22 AM.png)
![activity event](Screen Shot 2016-07-19 at 9.25.24 AM.png)

