# Chapter 9: Creating Events

### Joining groups

When we left off in chapter 8, we had added the ability to view groups, to create groups, and to view an individual group. We now need to add in the ability to join a group, as well as the ability to create and join events for a group.

Let's add this functionality in **GroupsView.js**.

```javascript
/* application/components/groups/GroupsView.js */
/* … */
class GroupsView extends Component{
  constructor(){
    super();
    this.addGroup = this.addGroup.bind(this);
    this.addUserToGroup = this.addUserToGroup.bind(this);
    this.state = {
      groups            : [],
      ready             : false,
      suggestedGroups   : [],
    }
  }
  ...
  addUserToGroup(group, currentUser){
    let { groups, suggestedGroups } = this.state;
    let member = {
      userId    : currentUser.id,
      role      : 'member',
      joinedAt  : new Date().valueOf(),
      confirmed : true
    };
    if (! find(group.members, ({ userId}) => isEqual(userId, currentUser.id))){
      group.members = [ ...group.members, member ];
      groups = [ ...groups, group ];
      suggestedGroups = suggestedGroups.filter(({ id }) => ! isEqual(id, group.id));
      this.setState({ groups, suggestedGroups })
      this.updateGroup(group);
    }
  }
  updateGroup(group){
    fetch(`${API}/groups/${group.id}`, {
      method: 'PUT',
      headers: Headers,
      body: JSON.stringify(group)
    })
    .then(response => response.json())
    .then(data => {})
    .catch(err => {})
    .done();
  }
  …
  case 'Group':
    return (
      <Group
        {...this.props}
        {...route}
        navigator={navigator}
        addUserToGroup={this.addUserToGroup}
      />
    )

```
![join group](/images/chapter-9/join-group-2.png)

![join group](/images/chapter-9/join-group-1.png)


Now when you click to join a group, our button should change its content to a success message, and the group should be added to our joined groups in the top level `Groups` component. Notice that we don’t just change the component state but also update our database through a **PUT** call to our Deployd server.

What about removing oneself from a group? For that, we can have an **ActionSheetIOS** component that gives us a list of actions, one of which can be to leave the group. Let's add an ellipses icon as the right button of our navigation bar, and have it open up the **ActionSheetIOS** component.

```javascript
/* application/components/groups/Group.js */
/* ... */
const OptionsButton = ({ openActionSheet }) => {
  return (
    <TouchableOpacity style={globals.pa1} onPress={openActionSheet}>
      <Icon name="ios-more" size={25} color="#ccc" />
    </TouchableOpacity>
  )
}
…
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
  openActionSheet(){
    let { group, currentUser } = this.props;
    let member = find(group.members, ({ userId }) => isEqual(userId, currentUser.id));
    let buttonActions = ['Unsubscribe', 'Cancel'];
    if (member && member.role === 'owner') {
      buttonActions.unshift('Create Event');
    }
    let options = {
      options: buttonActions,
      cancelButtonIndex: buttonActions.length-1
    };
    ActionSheetIOS.showActionSheetWithOptions(options, (buttonIndex) => {
      switch(buttonActions[buttonIndex]){
        case 'Unsubscribe':
          this.props.unsubscribeFromGroup(group, currentUser);
          break;
        case 'Create Event':
          this.visitCreateEvent(group);
          break;
        default:
          return;
      }
    });
  }
  visitCreateEvent(group){
    this.props.navigator.push({
      name: 'CreateEvent',
      group
    })
  }
/* ... */
<NavigationBar
  title={{title: group.name, tintColor: 'white'}}
  tintColor={Colors.brandPrimary}
  leftButton={<LeftButton navigator={navigator}/>}
  rightButton={<OptionsButton openActionSheet={this.openActionSheet}/>}
/>
/* ... */
```

We also have to define the method of `unsubscribeFromGroup` in our main `GroupsView.js` component.

```javascript
...
class GroupsView extends Component{
  constructor(){
    super();
    this.addGroup = this.addGroup.bind(this);
    this.unsubscribeFromGroup = this.unsubscribeFromGroup.bind(this);
    this.addUserToGroup = this.addUserToGroup.bind(this);
    this.state = {
      groups            : [],
      ready             : false,
      suggestedGroups   : [],
    }
  }
  ...
  unsubscribeFromGroup(group, currentUser){
    let { groups, suggestedGroups } = this.state;
    group.members = group.members.filter(({ userId }) => ! isEqual(userId, currentUser.id));
    groups = groups.filter(({ id }) => ! isEqual(id, group.id));
    suggestedGroups = [ ...suggestedGroups, group ];
    this.setState({ groups, suggestedGroups });
    this.updateGroup(group);
  }
  ...
  case 'Group':
    return (
      <Group
        {...this.props}
        {...this.state}
        {...route}
        addUserToGroup={this.addUserToGroup}
        navigator={navigator}
        unsubscribeFromGroup={this.unsubscribeFromGroup}
      />
  );
...
```

Let's review:
- We render a button as our right icon of our navigation bar. When it is pressed, we show the **ActionSheetIOS** by invoking the **showActionSheetWithOptions** function. We give two main options - either cancel the action sheet or unsubscribe from the group. We also give a 3rd option of creating an event if the user is an "owner" of the group. 
- When a user presses "unsubscribe", we filter the user out of the group's members and pass the group to the suggested groups array. We then update the state of both our groups and suggested groups. Last, we make changes to the database through an API call.

![group unsubscribe](/images/chapter-9/group-unsubscribe-1.png)
![group unsubscribe](/images/chapter-9/group-unsubscribe-2.png)
![group unsubscribe](/images/chapter-9/group-unsubscribe.png)

Let's make a commit here. 

[Commit](https://github.com/buildreactnative/assemblies-tutorial/tree/eadafaaa4dad011e4deda51874fc0f90d51aff19) - "Create join and unsubscribe functionality for groups"

### Creating Events

In the last part we added the ability to join or unsubscribe from a group. Next we want to give the group owners the ability to create and edit events. We can see that each member to a group has a specific role, currently either **member** or **owner**. We want users with **owner** privileges to be able to create and edit events, while users with **member** privileges have the ability to RSVP or cancel their reservation for an event.

We've already added the code that allows an "owner" to create an event on the **ActionSheetIOS**. However, it doesn't lead anywhere. Therefore, we have to create another route for **CreateEvent**.

Let's modify **GroupsView.js** and add a new file for **application/components/groups/CreateEvents.js**.

```javascript
/* application/components/groups/GroupsView.js */
/* ... */
import CreateEvent from './CreateEvent';
/* ... */
  case 'Create Event':
    return (
      <CreateEvent
        {...this.props}
        {...route}
        navigator={navigator}
      />
    )
/* ... */
```
```javascript
/* application/components/groups/CreateEvent.js */
import React, { Component } from 'react';
import { View, Text } from 'react-native';
import NavigationBar from 'react-native-navbar';
import Icon from 'react-native-vector-icons/Ionicons';

import BackButton from '../shared/BackButton';
import Colors from '../../styles/colors';
import { globals } from '../../styles';

class CreateEvent extends Component{
  constructor(){
    super();
    this.goBack = this.goBack.bind(this);
  }
  goBack(){
    this.props.navigator.pop();
  }
  render(){
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          title={{ title: 'Create Event', tintColor: 'white' }}
          tintColor={Colors.brandPrimary}
          leftButton={<BackButton handlePress={this.goBack}/>}
        />
        <View style={globals.flexCenter}>
          <Text style={globals.h2}>CreateEvent</Text>
        </View>
      </View>
    );
  }
};

export default CreateEvent;
```
![create event](/images/chapter-9/create-event-1.png)

## Selecting Date and Numerical Information

Now if the user selects **Create Event**, they should be directed to this page. Now we need to fill in the form to create an event. Remember that our events have the following schema: 

```
groupId: String
createdAt: Number
start: Number
end: Number
location: Object
going: Array 
name: String
capacity: Number
```

Since many of these fields require a numeric value, we’re going to explore using a **Picker** component. As far as the starting time and ending time, we will need some type of date selector.  Let’s design our form to take the name, location, and capacity in the first part, and the start and end times for the 2nd part.

```javascript
/* application/components/groups/CreateEvent.js */
import Icon from 'react-native-vector-icons/Ionicons';
import NavigationBar from 'react-native-navbar';
import React, { Component, PropTypes } from 'react';
import { ScrollView, View, Text, TextInput, Slider, TouchableOpacity } from 'react-native';
import { GooglePlacesAutocomplete } from 'react-native-google-places-autocomplete';
import { find } from 'underscore';

import Colors from '../../styles/colors';
import BackButton from '../shared/BackButton';
import { KeyboardAwareScrollView } from 'react-native-keyboard-aware-scroll-view';
import Config from 'react-native-config';
import { globals, formStyles, autocompleteStyles } from '../../styles';

const styles = formStyles;

class CreateEvent extends Component{
  constructor(){
    super();
    this.saveLocation = this.saveLocation.bind(this);
    this.submitForm = this.submitForm.bind(this);
    this.goBack = this.goBack.bind(this);
    this.state = {
      capacity    : 50,
      location    : null,
      name        : '',
      showPicker  : false,
    };
  }
  submitForm(){
    this.props.navigator.push({
      name: 'CreateEventConfirmation',
      group: this.props.group,
      location: this.state.location,
      capacity: this.state.capacity,
      eventName: this.state.name,
    })
  }
  saveLocation(data, details=null){
    if ( ! details ) { return; }
    let location = {
      ...details.geometry.location,
      city: find(details.address_components, (c) => c.types[0] === 'locality'),
      state: find(details.address_components, (c) => c.types[0] === 'administrative_area_level_1'),
      county: find(details.address_components, (c) => c.types[0] === 'administrative_area_level_2'),
      formattedAddress: details.formatted_address
    };
    this.setState({ location });
    this.name.focus();
  }
  goBack(){
    this.props.navigator.pop();
  }
  render(){
    let { capacity } = this.state;
    return (
      <View style={[globals.flexContainer, globals.inactive]}>
        <NavigationBar
          title={{ title: 'Create Event', tintColor: 'white' }}
          tintColor={Colors.brandPrimary}
          leftButton={<BackButton handlePress={this.goBack}/>}
        />
        <KeyboardAwareScrollView style={[styles.formContainer, globals.mt1]} contentInset={{bottom: 49}}>
          <Text style={styles.h4}>* Where is the event?</Text>
          <View style={globals.flex}>
            <GooglePlacesAutocomplete
              styles={autocompleteStyles}
              placeholder='Type a place or street address'
              minLength={2}
              autoFocus={true}
              fetchDetails={true}
              onPress={this.saveLocation}
              getDefaultValue={() => ''}
              query={{
                key: Config.GOOGLE_PLACES_API_KEY,
                language: 'en'
              }}
              currentLocation={false}
              currentLocationLabel='Current Location'
              nearbyPlacesAPI='GooglePlacesSearch'
              GoogleReverseGeocodingQuery={{}}
              GooglePlacesSearchQuery={{ rankby: 'distance' }}
              filterReverseGeocodingByTypes={['locality', 'adminstrative_area_level_3']}
              predefinedPlaces={[]}
            />
          </View>
          <Text style={styles.h4}>
            {"* What's the event name?"}
          </Text>
          <View style={styles.formField}>
            <TextInput
              returnKeyType="next"
              ref={(el) => this.name = el }
              onChangeText={(name) => this.setState({ name })}
              placeholderTextColor='#bbb'
              style={styles.input}
              placeholder="Type a name"
            />
          </View>
          <Text style={styles.h4}>
            Attendee capacity
          </Text>
          <View style={styles.formField}>
            <View style={styles.pickerButton}>
              <Text style={styles.input}>
                {capacity ? capacity : 'Choose a duration'}
              </Text>
            </View>
          </View>
          <View style={globals.mv1}>
            <Slider
              style={styles.slider}
              defaultValue={capacity}
              value={capacity}
              step={10}
              minimumValue={10}
              maximumValue={200}
              onValueChange={(val) => this.setState({capacity: val})}
            />
          </View>
        </KeyboardAwareScrollView>
        <TouchableOpacity
          onPress={this.submitForm}
          style={[styles.submitButton, styles.buttonMargin]}>
          <Text style={globals.largeButtonText}>
            Next
          </Text>
        </TouchableOpacity>
      </View>
    )
  }
}

export default CreateEvent;

```

Let's review the code:
- Our use of Google Places autocomplete should be pretty familiar by now. Notice, however, that we only pass in 2 options to the `query` property, which allows us to search actual addresses, rather than just cities.
- This is the first time we are using the `<Slider/>` component. This is a nice way to get values that are along some sort of numerical range. The API for `Slider` is very simple and straightforward.

After the user fills out this part of the form and presses "Next", they reach a blank screen. That's because we haven't yet defined the second part of event creation -- 'CreateEventConfirmation.'

![create event](/images/chapter-9/create-event-2.png)

### Using Mobile Picker Components

In order to render the second part of our `events` form, let's modify **GroupsView.js** to include our new **CreateEventConfirmation** route.

```javascript
/* application/components/groups/GroupsView.js */
/* ... */
import CreateEventConfirmation from './CreateEventConfirmation';
/* ... */
case 'CreateEventConfirmation':
  return (
    <CreateEventConfirmation
      {...this.props}
      {...this.state}
      {...route}
      navigator={navigator}
    />
  );
/* ... */
```

And let's add a simple component as our new **CreateEventConfirmation** component:

```javascript
/* application/components/groups/CreateEventConfirmation.js */
import React, { Component } from 'react';
import { View, Text } from 'react-native';
import NavigationBar from 'react-native-navbar';
import Icon from 'react-native-vector-icons/Ionicons';

import Colors from '../../styles/colors';
import BackButton from '../shared/BackButton';
import { globals } from '../../styles';

class CreateEventConfirmation extends Component{
  render(){
    let titleConfig = { title: 'Create Event', tintColor: 'white' };
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          title={titleConfig}
          tintColor={Colors.brandPrimary}
          leftButton={<BackButton handlePress={this.goBack}/>}
        />
        <View style={globals.flexCenter}>
          <Text style={globals.h2}>CreateEventConfirmation</Text>
        </View>
      </View>
    )
  }
};

export default CreateEventConfirmation;

```
We should direct to a simple page now after the first part of the form. Now it's time to fill in the rest! But first, let's make a commit here.

[Commit](https://github.com/buildreactnative/assemblies-tutorial/tree/fd1808d98d749c4a9b70c80ed6168e125540646a) - "Render first part of event creation form"

![create event](/images/chapter-9/create-event-3.png)

Now let's move on to fleshing out the second part of our form. First we'll have to **npm install** the package `react-native-picker`. Once that is done we can fill in our content.

```javascript
import React, { Component, PropTypes } from 'react';
import {
  ScrollView,
  View,
  Text,
  TextInput,
  DatePickerIOS,
  Modal,
  TouchableOpacity
} from 'react-native';

import moment from 'moment';
import Icon from 'react-native-vector-icons/Ionicons';
import NavigationBar from 'react-native-navbar';
import Picker from 'react-native-picker';
import Colors from '../../styles/colors';
import { Headers } from '../../fixtures';
import { KeyboardAwareScrollView } from 'react-native-keyboard-aware-scroll-view';
import BackButton from '../shared/BackButton';
import { API, DEV } from '../../config';
import { globals, formStyles } from '../../styles';
const styles = formStyles;

function setErrorMsg({ location, name }){
  if (! location ){
    return 'You must provide a location.';
  } else if (! name ){
    return 'You must provide a name.';
  } else {
    return '';
  }
};

class CreateEventConfirmation extends Component{
  constructor(){
    super();
    this.goBack = this.goBack.bind(this);
    this.saveStart = this.saveStart.bind(this);
    this.saveEnd = this.saveEnd.bind(this);
    this.submitForm = this.submitForm.bind(this);
    this.toggleStartModal = this.toggleStartModal.bind(this);
    this.toggleEndModal = this.toggleEndModal.bind(this);
    this.state = {
      description       : '',
      end               : new Date(),
      errorMsg          : '',
      finalEnd          : new Date(),
      finalStart        : new Date(),
      showEndModal      : false,
      showStartModal    : false,
      start             : new Date(),
    };
  }
  submitForm(){
    let errorMsg = setErrorMsg({...this.props, ...this.state});
    console.log('SUBMIT', errorMsg);
    if (errorMsg !== ''){
      this.setState({ errorMsg }); return;
    }
    let event = {
      capacity    : this.props.capacity,
      description : this.state.description,
      createdAt   : new Date().valueOf(),
      end         : this.state.finalEnd.valueOf(),
      going       : [ this.props.currentUser.id ],
      groupId     : this.props.group.id,
      location    : this.props.location || {},
      name        : this.props.eventName,
      start       : this.state.finalStart.valueOf(),
    };
    fetch(`${API}/events`, {
      method: 'POST',
      headers: Headers,
      body: JSON.stringify(event)
    })
    .then(response => response.json())
    .then(data => this.props.navigator.push({
      name: 'Group',
      group: this.props.group
    }))
    .catch(err => {
      console.log('ERR', err);
      this.setState({ errorMsg: err.reason })
    })
    .done();
  }
  goBack(){
    this.props.navigator.pop();
  }
  toggleStartModal(){
    this.setState({ showStartModal: ! this.state.showStartModal })
  }
  toggleEndModal(){
    this.setState({ showEndModal: ! this.state.showEndModal })
  }
  saveStart(){
    this.setState({
      showStartModal: false,
      finalStart: this.state.start,
      end: this.state.start,
      finalEnd: this.state.start
    })
  }
  saveEnd(){
    this.setState({
      showEndModal: false,
      finalEnd: this.state.end
    })
  }
  render(){
    let { finalStart, finalEnd } = this.state;
    let titleConfig = { title: 'Confirm Event', tintColor: 'white' };
    return (
      <View style={[globals.flexContainer, globals.inactive]}>
        <NavigationBar
          title={titleConfig}
          tintColor={Colors.brandPrimary}
          leftButton={<BackButton handlePress={this.goBack}/>}
        />
        <KeyboardAwareScrollView 
          style={[styles.formContainer, globals.pv1]} 
          contentInset={{bottom: 49}}
        >
          <Text style={styles.h4}>
            {"* When does the event start?"}
          </Text>
          <View style={styles.formField}>
            <TouchableOpacity 
              style={styles.flexRow} 
              onPress={this.toggleStartModal}
            >
              <Text style={styles.input}>
                {finalStart ? moment(finalStart).format('dddd MMM Do, h:mm a') : 'Choose a starting time'}
              </Text>
              <Icon 
                name="ios-arrow-forward" 
                color='#777' 
                size={30} 
                style={globals.mr1}
              />
            </TouchableOpacity>
          </View>
          <Text style={styles.h4}>
            * When does the event end?
          </Text>
          <View style={styles.formField}>
            <TouchableOpacity 
              style={styles.flexRow} 
              onPress={this.toggleEndModal}
            >
              <Text style={styles.input}>
                {finalEnd ? moment(finalEnd).format('dddd MMM Do, h:mm a') : 'Choose an ending time'}
              </Text>
              <Icon 
                name="ios-arrow-forward" 
                color='#777' 
                size={30} 
                style={globals.mr1}
              />
            </TouchableOpacity>
          </View>
          <Text style={styles.h4}>
            Leave a note for your attendees
          </Text>
          <TextInput
            ref={(el) => this.description = el }
            returnKeyType="next"
            blurOnSubmit={true}
            clearButtonMode='always'
            onChangeText={(description) => this.setState({ description })}
            placeholderTextColor='#bbb'
            style={styles.largeInput}
            multiline={true}
            placeholder="Type a summary of the event..."
          />
          <View style={[styles.error, globals.ma1]}>
            <Text style={styles.errorText}>
              {this.state.errorMsg}
            </Text>
          </View>
        </KeyboardAwareScrollView>
        <TouchableOpacity 
          onPress={this.submitForm} 
          style={[styles.submitButton, styles.buttonMargin]}
        >
          <Text style={globals.largeButtonText}>
            Save
          </Text>
        </TouchableOpacity>
        <Modal
          animationType='slide'
          transparent={true}
          visible={this.state.showStartModal}
          onRequestClose={this.saveStart}
          >
         <View style={styles.modal}>
           <View style={styles.datepicker}>
             <DatePickerIOS
               date={this.state.start}
               minimumDate={new Date()}
               minuteInterval={15}
               mode='datetime'
               onDateChange={(start) => this.setState({ start })}
             />
             <View style={styles.btnGroup}>
               <TouchableOpacity 
                 style={styles.pickerButton} 
                 onPress={() => this.setState({ showStartModal: false })}
               >
                 <Text style={styles.btnText}>
                   Cancel
                 </Text>
               </TouchableOpacity>
               <TouchableOpacity 
                 style={[styles.pickerButton, globals.brandPrimary]} 
                 onPress={this.saveStart}
               >
                 <Text style={[styles.btnText, { color: 'white' }]}>
                   Save
                 </Text>
               </TouchableOpacity>
             </View>
           </View>
         </View>
        </Modal>
        <Modal
          animationType={"slide"}
          transparent={true}
          visible={this.state.showEndModal}
          onRequestClose={this.saveEnd}
          >
         <View style={styles.modal}>
           <View style={styles.datepicker}>
             <DatePickerIOS
               date={this.state.end}
               minimumDate={new Date()}
               minuteInterval={15}
               mode='datetime'
               onDateChange={(end) => this.setState({ end })}
             />
             <View style={styles.btnGroup}>
               <TouchableOpacity 
                 style={styles.pickerButton} 
                 onPress={() => this.setState({ showEndModal: false })}
               >
                 <Text style={styles.btnText}>
                   Cancel
                 </Text>
               </TouchableOpacity>
               <TouchableOpacity 
                 style={[styles.pickerButton, globals.brandPrimary]} 
                 onPress={this.saveEnd}
               >
                 <Text style={[styles.btnText, globals.buttonText]}>
                   Save
                 </Text>
               </TouchableOpacity>
             </View>
           </View>
         </View>
        </Modal>
      </View>
    )
  }
}

export default CreateEventConfirmation;


```

This has a lot to go over. Let's go step by step.

- We are using the **Picker** component from **react-native-picker** to select dates. This component provides more customization in regards to styling than the out-of-the-box **Picker** component from React native. 
- We are also using the **Modal** component. We are able to toggle the visibility of both the starting time modal and the ending time modal through our state object. We are also modifying the ending time once the starting time is selected.
- Our `handleSubmit` method gathers all of the event information and posts it to the database. From there, we direct the navigator to the **Group** screen. 

Right now, the form will save the event and redirect correctly. However, we don't see our events on the group screen. This is because we aren't fetching them from the database. That is what we will fix next.

![create event](/images/chapter-9/create-event-4.png)
![create event](/images/chapter-9/create-event-5.png)





