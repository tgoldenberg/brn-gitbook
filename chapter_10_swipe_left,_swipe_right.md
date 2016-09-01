
Now when a user completes the new event form and submits, they should be directed back to the **Group** page. From here, we will want to fetch the events related to that group on `componentDidMount` and then render them in the **events** section. Let's modify **Group.js**.

```javascript
/* application/components/groups/Group.js */

class EventList extends Component{
  render(){
    return (
      <View>
        {this.props.events.map((event, idx) => (
          <Text key={idx}>{event.name}</Text>
        ))}
      </View>
    )
  }
};

/* ... */

class Group extends Component{
  constructor(){
    super();
    this.goBack = this.goBack.bind(this);
    this.visitProfile = this.visitProfile.bind(this);
    this.visitCreateEvent = this.visitCreateEvent.bind(this);
    this.openActionSheet = this.openActionSheet.bind(this);
    this.state = {
      events    : [],
      ready     : false,
      users     : [],
    }
  }
  /* ... */
  componentDidMount(){
    this._loadEvents();
  }
  _loadEvents(){
    let query = {
      groupId: this.props.group.id,
      end: { $gt: new Date().valueOf() },
      $limit: 10,
      $sort: { start: -1 }
    };
    fetch(`${API}/events?${JSON.stringify(query)}`)
    .then(response => response.json())
    .then(events => this._loadUsers(events))
    .catch(err => {})
    .done();
  }
/* ... */
    <Text style={styles.h2}>Events</Text>
    <EventList 
      {...this.props}
      {...this.state}
    />
/* ... */
```

We should now see the names of the events that were created in a simple **Text** component.

![group](/images/chapter-9/group-with-event-1.png)

Let's spruce up our `EventList` component a bit.

```javascript
/* application/components/groups/Group.js */

/* ... */ 
import { rowHasChanged } from '../../utilities';
/* ... */
class EventList extends Component{
  constructor(){
    super();
    this._renderRow = this._renderRow.bind(this);
  }
  _renderRow(event, sectionID, rowID){
    let { currentUser, events, group } = this.props;
    let isGoing = find(event.going, (id) => isEqual(id, currentUser.id));
    return (
      <View style={styles.eventContainer}>
        <TouchableOpacity
          style={globals.flex}
          onPress={() => this.props.visitEvent(event)}
        >
          <Text style={globals.h5}>
            {event.name}
          </Text>
          <Text style={styles.h4}>
            {moment(event.start).format('dddd, MMM Do')}
          </Text>
          <Text style={styles.h4}>
            {event.going.length} Going
          </Text>
        </TouchableOpacity>
        <View style={[globals.flexRow, globals.pa1]}>
          <Text style={[globals.primaryText, styles.h4, globals.ph1]}>
            {isGoing ? "You're Going" : "Want to go?"}
          </Text>
          <Icon
            name={ isGoing ? "ios-checkmark" : "ios-add" }
            size={30}
            color={Colors.brandPrimary}
          />
        </View>
      </View>
    )
  }
  dataSource(){
    return (
      new ListView.DataSource({ rowHasChanged })
        .cloneWithRows(this.props.events)
    );
  }
  render(){
    if (! this.props.events.length ){ 
      return (
        <Text style={[globals.h5, globals.mh2]}>
          No events scheduled
        </Text>
      )
    }
    return (
      <ListView
        enableEmptySections={true}
        dataSource={this.dataSource()}
        renderRow={this._renderRow}
        scrollEnabled={false}
        style={globals.flex}
      />
    )
  }
};

/* ... */
```

Let's review:
- We are using the **ListView** component to render our events. We initialize a **dataSource** with the events that are passed as props. We then check to see if the user is already attending the event, and if so, we show a success message. If not, we show a "want to join?" message. 

Now is a good time for a commit.

[Commit](https://github.com/buildreactnative/assemblies-tutorial/tree/f84c739f60c28a5d4355a4e7193755da1354af23) - "Render created events on Group screen"

![group](/images/chapter-9/group-with-event-2.png)


## Joining an event

Once we have created an event, we want our users to be able to RSVP for them, or cancel their reservation. Now is a good opportunity to use a swipe-to-join functionality. When our user swipes left on the event, we want to show them the option to either join or leave the event. We can use the package `react-native-swipeout` for this. Add this line to your **package.json** file and then `npm install`.

```json
    "react-native-swipeout": "dancormier/react-native-swipeout#829b7c01ba969cd70509944bc210289b759d48a6",
```

```javascript
/* application/components/groups/Group.js */
/* ... */
import Swipeout from 'react-native-swipeout';
/* ... */
class EventList extends Component{
  constructor(){
    super();
    this._renderRow = this._renderRow.bind(this);
  }
  getButtons(isGoing, event, currentUser){
    if (isGoing){
      return [{
        text: 'Cancel',
        type: 'delete',
        onPress: () => { this.props.cancelRSVP(event, currentUser) }
      }];
    } else {
      return [{
        text: 'RSVP',
        type: 'primary',
        onPress: () => { this.props.joinEvent(event, currentUser) }
      }];
    }
  }
  _renderRow(event, sectionID, rowID){
    let { currentUser, events, group } = this.props;
    let isGoing = find(event.going, (id) => isEqual(id, currentUser.id));
    let right = this.getButtons(isGoing, event, currentUser);
    return (
      <Swipeout backgroundColor='white' rowID={rowID} right={right}>
        <View style={styles.eventContainer}>
          <TouchableOpacity
            style={globals.flex}
            onPress={() => this.props.visitEvent(event)}
          >
            <Text style={globals.h5}>
              {event.name}
            </Text>
            <Text style={styles.h4}>
              {moment(event.start).format('dddd, MMM Do')}
            </Text>
            <Text style={styles.h4}>
              {event.going.length} Going
            </Text>
          </TouchableOpacity>
          <View style={[globals.flexRow, globals.pa1]}>
            <Text style={[globals.primaryText, styles.h4, globals.ph1]}>
              {isGoing ? "You're Going" : "Want to go?"}
            </Text>
            <Icon
              name={ isGoing ? "ios-checkmark" : "ios-add" }
              size={30}
              color={Colors.brandPrimary}
            />
          </View>
        </View>
      </Swipeout>
    )
  }
/* ... */
```

Now we should be able to reveal either a "cancel" or "join" button when you swipe left on each event row.

![swipeout](/images/chapter-10/swipeout-1.png)


In the **Swipeout** component we are referencing three actions, `this.props.cancelRSVP`, `this.props.visitEvent` and `this.props.joinEvent`. We should now define them in our **Group** component.

Here are the methods we need to add to **Group**:

```javascript
/* application/components/groups/Group.js */
class Group extends Component{
  constructor(){
    super();
    this.openActionSheet = this.openActionSheet.bind(this);
    this.cancelRSVP = this.cancelRSVP.bind(this);
    this.joinEvent = this.joinEvent.bind(this);
    this.goBack = this.goBack.bind(this);
    this.visitProfile = this.visitProfile.bind(this);
    this.visitEvent = this.visitEvent.bind(this);
    this.visitCreateEvent = this.visitCreateEvent.bind(this);
    this.updateEvents = this.updateEvents.bind(this);
    this.state = {
      events    : [],
      ready     : false,
      users     : [],
    }
  }
  joinEvent(event, currentUser){
    let { events } = this.state;
    let updatedEvent = {
      ...event,
      going: [ ...event.going, currentUser.id ]
    };
    let index = findIndex(this.state.events, ({ id }) => isEqual(id, event.id));
    let updatedEvents = [
      ...events.slice(0, index),
      updatedEvent,
      ...events.slice(index + 1)
    ];
    this.setState({ events: updatedEvents })
    this.updateEventGoing(event);
  }
  cancelRSVP(event, currentUser){
    let { events } = this.state;
    let updatedEvent = {
      ...event,
      going: event.going.filter((userId) => ! isEqual(userId, currentUser.id))
    };
    let index = findIndex(this.state.events, ({ id }) => isEqual(id, event.id));
    let updatedEvents = [
      ...events.slice(0, index),
      updatedEvent,
      ...events.slice(index + 1)
    ];
    this.setState({ events: updatedEvents })
    this.updateEventGoing(event);
  }
  visitEvent(event){
    this.props.navigator.push({
      name: 'Event',
      group: this.props.group,
      updateEvents: this.updateEvents,
      event,
    })
  }
  updateEvents(event){
    let { events } = this.state;
    let idx = findIndex(this.state.events, ({ id }) => isEqual(id, event.id));
    let updatedEvents = [
      ...events.slice(0, idx),
      event,
      ...events.slice(idx + 1)
    ];
    this.setState({ events: updatedEvents })
  }
  updateEventGoing(event){
    fetch(`${API}/events/${event.id}`, {
      method: 'PUT',
      headers: Headers,
      body: JSON.stringify({
        going: event.going
      })
    })
    .then(response => response.json())
    .then(data => {})
    .catch(err => {})
    .done();
  }
```
And then we add the methods in our `render` method:

```javascript
<EventList
  {...this.state}
  {...this.props}
  visitEvent={this.visitEvent}
  joinEvent={this.joinEvent}
  cancelRSVP={this.cancelRSVP}
/>
```
Let's go over what we just added:
- If a user is already RSVP'd to an event, we show the "cancel" button in **Swipeout**, otherwise the "RSVP" button. Each of these methods change the values of **this.state.events**, and then invoke the method `updateEventGoing`. This method then updates the database. 
- The process of updating the views before updating the database is sometimes called "optimistic rendering." This provides a much smoother and faster user experience.
- Our `visitEvent` method routes us to the individual event. We haven't yet added this route or the **Event** component so it renders an empty screen.

Take a moment to test out the swipe-left functionality. So that you can change from RSVP to cancel, and back again.

![swipeout](/images/chapter-10/swipeout-2.png)
![swipeout](/images/chapter-10/swipeout-3.png)


Let's take a moment to make a commit.

[Commit](https://github.com/buildreactnative/assemblies-tutorial/tree/a9f32830bd66902ea75aed7014a5c59aafde179f) - "Add join/leave functionality to events"

### Rendering an Event Screen

Now that we can join and leave a group, we should enable our user to view an individual event and its relevant information. Let’s create a new route, ‘Event’, and direct to it when the user presses on the event section..

```javascript
/* application/components/groups/GroupsView.js */
/* ... */
import Event from './Event';
/* ... */
case 'Event':
  return (
    <Event 
      {...this.props}
      {...route}
      navigator={navigator}
    />
);
/* ... */
```

```javascript
/* application/components/groups/Event.js */
import React, { Component } from 'react';
import {
  View,
  Text,
  ScrollView,
  Image,
  MapView,
  TouchableOpacity
} from 'react-native';

import moment from 'moment';
import Icon from 'react-native-vector-icons/Ionicons';
import NavigationBar from 'react-native-navbar';
import Headers from '../../fixtures';
import { find, uniq, contains } from 'underscore';
import Colors from '../../styles/colors';
import BackButton from '../shared/BackButton';
import { globals, groupsStyles } from '../../styles';
import { API, DEV } from '../../config';
const styles = groupsStyles;

const EmptyMap = ({ going, ready }) => (
  <View>
    <View style={[globals.map, globals.inactive, globals.pa1]}/>
    <View style={styles.bottomPanel}>
      <Text style={[globals.h5, globals.primaryText]}>
        { ready ? `${going.length} going` : '' }
      </Text>
    </View>
  </View>
);

const EventMap = ({ location, going, ready }) => {
  if ( ! location || typeof location != 'object' ) {
    return <EmptyMap going={going} ready={ready}/>
  }
  const mapRegion = {
      latitude        : location.lat,
      longitude       : location.lng,
      latitudeDelta   : 0.01,
      longitudeDelta  : 0.01
    }
  return (
    <View style={globals.inactive}>
      <MapView
        style={globals.map}
        region={mapRegion}
        annotations={[{
          latitude: mapRegion.latitude,
          longitude: mapRegion.longitude
        }]}
      />
      <View style={[styles.bottomPanel, globals.inactive, globals.pa1]}>
        <Text style={[globals.h5, globals.primaryText]}>
          {location.formattedAddress}
        </Text>
      </View>
    </View>
  )
}

const JoinControls = ({ hasJoined, joinEvent }) => (
  <View style={[styles.joinButtonContainer, globals.mv1]}>
    <TouchableOpacity
      onPress={() => { if (!hasJoined) joinEvent() }}
      style={styles.joinButton}
    >
      <Text style={styles.joinButtonText}>
        { hasJoined ? 'Joined' : 'Join'}
      </Text>
      <Icon
        name={hasJoined ? "ios-checkmark" : "ios-add"}
        size={30}
        color='white'
      />
    </TouchableOpacity>
  </View>
)
class Event extends Component{
  constructor(){
    super();
    this.goBack = this.goBack.bind(this);
    this.joinEvent = this.joinEvent.bind(this);
    this.state = {
      ready         : false,
      eventMembers  : []
    };
  }
  _loadUsers(){
    let query = { id: { $in: this.props.event.going } };
    fetch(`${API}/users?${JSON.stringify(query)}`)
    .then(response => response.json())
    .then(eventMembers => this.setState({ eventMembers }))
    .catch(err => {})
    .done();
  }
  componentDidMount(){
    this._loadUsers();
  }
  joinEvent(){
    let { event, currentUser, updateEvents } = this.props;
    let going = [ ...event.going, currentUser.id ];
    let users = [ ...this.state.eventMembers, currentUser ];
    this.setState({ eventMembers: users });
    fetch(`${API}/events/${event.id}`, {
      method: 'PUT',
      headers: Headers,
      body: JSON.stringify({going: going})
    })
    .then(response => response.json())
    .then(data => updateEvents(data))
    .catch(err => console.log('UPDATE EVENT ERROR: ', err))
    .done();
  }
  goBack(){
    this.props.navigator.pop();
  }
  visitProfile(user){
    if ( user.id === this.props.currentUser.id ) {
      return;
    }
    this.props.navigator.push({
      name: 'Profile',
      user
    })
  }
  render(){
    let { event, group, currentUser, navigator } = this.props;
    let { ready, eventMembers } = this.state;
    let hasJoined = contains(event.going, currentUser.id);
    let justJoined = contains(eventMembers.map(m => m.id), currentUser.id);
    let titleConfig = { title: event.name, tintColor: 'white' };
    return (
      <View style={styles.flexContainer}>
        <NavigationBar
          title={titleConfig}
          tintColor={Colors.brandPrimary}
          leftButton={<BackButton handlePress={this.goBack}/>}
        />
        <ScrollView
          style={globals.flexContainer}
          contentInset={{ bottom: 49 }}
        >
          <EventMap
            location={event.location}
            going={event.going}
            ready={ready}
          />
          <View style={styles.infoContainer}>
            <Text style={styles.h2}>
              Summary
            </Text>
            <Text style={[styles.h4, globals.mh2]}>
              {event.description ? event.description : 'N/A'}
            </Text>
          </View>
          <View style={globals.lightDivider} />
          <View style={styles.infoContainer}>
            <Text style={styles.h2}>
              Date
            </Text>
            <Text style={[styles.h4, globals.mh2]}>
              {moment(event.start).format('dddd, MMM Do, h:mm')} till {moment(event.end).format('dddd, MMM Do, h:mm')}
            </Text>
          </View>
          <View style={globals.lightDivider} />
          { !hasJoined && <JoinControls hasJoined={justJoined} joinEvent={this.joinEvent} /> }
          <View style={styles.infoContainer}>
            <Text style={styles.h2}>
              Going <Text style={styles.h4}>{eventMembers.length}</Text>
            </Text>
            {eventMembers.map((member, idx) => (
              <TouchableOpacity
                key={idx}
                onPress={() => this.visitProfile(member)}
                style={globals.flexRow}
              >
                <Image
                  source={{uri: member.avatar}}
                  style={globals.avatar}
                />
                <View style={globals.textContainer}>
                  <Text style={globals.h5}>
                    {member.firstName} {member.lastName}
                  </Text>
                </View>
              </TouchableOpacity>
            ))}
          </View>
          <View style={styles.break} />
        </ScrollView>
      </View>
    )
  }
}

export default Event;

```

Now we should be able to press an event row and route to an individual event screen.

![event](/images/chapter-10/event-1.png)

## Wrapping Up

There are a lot of enhancements we can make, but we essentially have created the ability for our users to create, manage, and join groups, as well as to create and join events nearby. Here are some ideas for further enhancements / features:

-	More control over managing user roles within a group (ability to nominate another user as an ‘admin’, adding the ability to delete a group, etc.)
-	Ability to write comments in an event
-	Ability to contact group members and write private messages to them

These are all things that are possible, and indeed, have been built into the production version of Assemblies! But for now, we’ll be moving on to building out our Calendar and Activity Views. 

Congrats on building out a pretty complex user interface! We have considered a lot of user interactions and data changes, so great job on completing this part!


