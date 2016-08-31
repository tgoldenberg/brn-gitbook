# Chapter 12: Creating a Calendar View

Now that we've successfully added notifications and sculpted our **ActivityView**, there's pretty much one view left to really build out, which is our **CalendarView**. We want to show a list of upcoming events, both those that our user is attending and those that they are not, in chronological order. We also want to have "sticky" headers for each day that we show  events. Let's see what we can do.

First let's set up the routing for our Calendar view.

```javascript
/* application/components/calendar/CalendarView.js */
import React, { Component } from 'react';
import { Navigator } from 'react-native';

import Calendar from './Calendar';
import Event from '../groups/Event';
import { API, DEV } from '../../config';
import { extend } from 'underscore';
import { globals } from '../../styles';

class CalendarView extends Component{
  constructor(){
    super();
    this.state = {
      events  : [],
      ready   : false,
    }
  }
  _loadGroups(){
    let query = {
      members: {
        $elemMatch: {
          userId: this.props.currentUser.id
        }
      }
    };
    fetch(`${API}/groups?${JSON.stringify(query)}`)
    .then(response => response.json())
    .then(groups => this._loadEvents(groups))
    .catch(err => this.ready(err))
    .done()
  }
  ready(err){
    this.setState({ ready: true });
  }
  _loadEvents(groups){
    let dateQuery = { end: { $gt: new Date().valueOf() }};
    let query = {
      $or: [
        extend({}, dateQuery, { groupId: { $in: groups.map((g) => g.id) }}),
        extend({}, dateQuery, { going:  { $elemMatch: { $eq: this.props.currentUser.id }}}),
        extend({}, dateQuery, { 'location.city.long_name': this.props.currentUser.location.city.long_name })
      ],
      $limit: 20,
    };
    fetch(`${API}/events?${JSON.stringify(query)}`)
    .then(response => response.json())
    .then(events => this.setState({ events, ready: true }))
    .catch(err => this.ready(err))
    .done();
  }
  componentDidMount(){
    this._loadGroups();
  }
  render(){
    return (
      <Navigator
        initialRoute={{ name: 'Calendar' }}
        style={globals.flex}
        renderScene={(route, navigator) => {
          switch(route.name){
            case 'Calendar':
              return (
                <Calendar
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
          }
        }}
      />
    )
  }
};

export default CalendarView;

```

Then let's flesh out **Calendar.js**.


```javascript
/* application/components/calendar/Calendar.js */
import moment from 'moment';
import Icon from 'react-native-vector-icons/Ionicons';
import NavigationBar from 'react-native-navbar';
import React, { Component } from 'react';
import {
  View,
  Text,
  ListView,
  TouchableOpacity
} from 'react-native';
import { uniq, flatten, find, contains } from 'underscore';
import {
  getSectionData,
  getRowData,
  sectionHeaderHasChanged,
  rowHasChanged
} from '../../utilities';
import Loading from '../shared/Loading';
import { globals, calendarStyles } from '../../styles';

const styles = calendarStyles;

const EmptyList = ({ ready }) => {
  if (! ready ) { return <Loading /> }
  return (
    <View style={[globals.textContainer, calendarStyles.emptyText]}>
      <Text style={styles.h2}>
        No events scheduled. Explore groups in the groups tab or create your own to start an event.
      </Text>
    </View>
  );
};

class EventList extends Component{
  constructor(props){
    super(props);
    this.renderRow = this.renderRow.bind(this);
    this.renderSectionHeader = this.renderSectionHeader.bind(this);
    this.visitEvent = this.visitEvent.bind(this);
    this.state = {
      dataSource: this._loadData(props.events)
    };
  }
  _loadData(events){
    let dataBlob = {};
    let dates = uniq(events.map(evt => new Date(evt.start).toLocaleDateString())); /* Get all unique dates */
    let sections = dates.map((date, id) => ({
      date    : new Date(date),
      events  : events.filter(event => new Date(event.start).toLocaleDateString() === date),
      id      : id,
    }));
    let sectionIDs = sections.map((section, id) => id);
    let rowIDs = sectionIDs.map(sectionID => sections[sectionID].events.map((e, id) => id));

    sections.forEach(section => {
      dataBlob[section.id] = section.date;
      section.events.forEach((event, rowID) => {
        dataBlob[`${section.id}:${rowID}`] = event;
      });
    });

    return new ListView.DataSource({
      getSectionData: getSectionData,
      getRowData: getRowData,
      rowHasChanged: rowHasChanged,
      sectionHeaderHasChanged: sectionHeaderHasChanged
    })
    .cloneWithRowsAndSections(dataBlob, sectionIDs, rowIDs);
  }
  visitEvent(event){
    this.props.navigator.push({
      name: 'Event',
      event
    })
  }
  renderRow(event, sectionID, rowID){
    let isGoing = contains(event.going, this.props.currentUser.id);
    return (
      <TouchableOpacity
        style={styles.row}
        onPress={() => this.visitEvent(event)}
      >
        <View style={globals.flex}>
          <View style={styles.textContainer}>
            <Text style={styles.h4}>
              {event.name}
            </Text>
            <Text style={styles.h5}>
               {event.going.length} going
             </Text>
            { isGoing && <Text style={[globals.primaryText, styles.h5]}><Icon name="ios-checkmark" color={Colors.brandSuccess}/> Yes</Text> }
          </View>
        </View>
        <View style={styles.textContainer}>
          <Text style={[styles.dateText, globals.mh1]}>
            {moment(event.start).format('h:mm a')}
          </Text>
          <Icon
            style={styles.arrow}
            name="ios-arrow-forward"
            size={25}
            color={Colors.bodyTextLight}
          />
        </View>
      </TouchableOpacity>
    )
  }
  renderSectionHeader(sectionData, sectionID){
    return (
      <View style={styles.sectionHeader}>
        <Text style={styles.sectionHeaderText}>
          {moment(sectionData).format('dddd MMM Do')}
        </Text>
      </View>
    )
  }
  render(){
    return (
      <ListView
        enableEmptySectionHeaders={true}
        style={globals.flex}
        contentInset={{ bottom: 49 }}
        automaticallyAdjustContentInsets={false}
        dataSource={this.state.dataSource}
        renderRow={this.renderRow}
        renderSectionHeader={this.renderSectionHeader}
      />
    )
  }
}

class Calendar extends Component{
  render(){
    let { events, ready } = this.props;
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          tintColor={Colors.brandPrimary}
          title={{ title: 'Calendar ', tintColor: 'white' }}
        />
        { events && events.length ? <EventList {...this.props}/> : <EmptyList ready={ready} /> }
      </View>
    )
  }
};

export default Calendar;


```

Let’s go over a few things. 

- Notice how in order to get proper section headers, we had to modify our **events** data. First we create an array of unique dates, which become our sections, and then for each of these sections, we create a rowID for each event that takes place on that date. We then use the **ListView** component to render them. As we are using sections, we define our event rows and section headers through the `renderRow` and `renderSectionHeaders` properties of the **ListView**.
- When querying for our calendar events, we first fetch the groups that our user belongs to. Then we query the upcoming events for those groups. 
- We also refactor some relevant functions that we use in our **ListView**, for example, `getSectionData`, etc. Let's place those in **application/utilities/index.js**.

```javascript
/* application/utilities/index.js */
export function rowHasChanged(r1, r2) {
  return r1 != r2;
};
export function sectionHeaderHasChanged(s1, s2){
  return s1 != s2;
};
export function getSectionData(dataBlob, sectionID) {
  return dataBlob[sectionID]
};
export function getRowData(dataBlob, sectionID, rowID){
  return dataBlob[`${sectionID}:${rowID}`];
}
```

![calendar](/images/chapter-12/calendar-1.png)
![calendar](/images/chapter-12/calendar-2.png)

Let's make a commit there.

[Commit 25](https://github.com/buildreactnative/assemblies-tutorial/tree/9803794854743f00cb203f6e32d8dec6470dc103) - "Render the Calendar View"



### Fixing the Profile View

Now that we have a functional **CalendarView**, we can focus our attention on some of the other areas of the app. What about our **ProfileView**? We are currently rendering the user information, but don’t have a way for the user to edit that information. This is a good opportunity to re-use some of the interface from the **Register** and **RegisterConfirmation** components. We should also start to think about how the user experience will be on a mobile device. We can use the **react-native-keyboard-aware-scroll-view** package to make sure that the focused input is above the device’s keyboard. You can always check this with the `cmd + sft + k` command, which will toggle the native keyboard view on the iOS simulator.

Let’s go to parts of our app that didn’t consider this before and add the component **KeyboardAwareScrollView** in place of our regular **ScrollView**. 

Then our **Register.js** file will look like this:

```javascript
/* ... */
import { KeyboardAwareScrollView } from 'react-native-keyboard-aware-scroll-view';
/* ... */
return (
  <View style={styles.container}>
    <NavigationBar
      title={titleConfig}
      tintColor={Colors.brandPrimary}
      leftButton={<LeftButton handlePress={() => navigator.pop()}/>}
    />
    <KeyboardAwareScrollView style={styles.formContainer}>
      <TouchableOpacity onPress={()=> navigator.push({ name: 'Login' })}>
```

We can also add this to **CreateEvent.js**, **CreateGroup.js**, and **CreateEventConfirm.js**.

### Adding to the Profile View

Now that we’ve addressed some user experience issues in our app, what about the Profile View? So far, the only action a user can take is `logout`. Let’s add the ability to change your avatar, technologies, and basic information. First let’s change our **ProfileView.js** component to be a navigation component with three routes:

```javascript
/* application/components/profile/ProfileView.js */
import React, { Component } from 'react';
import { Navigator } from 'react-native';

import UserProfile from './UserProfile';
import UserSettings from './UserSettings';
import UserTechnologies from './UserTechnologies';
import { globals } from '../../styles';

class ProfileView extends Component{
  render(){
    return (
      <Navigator
        initialRoute={{ name: 'UserProfile' }}
        style={globals.flex}
        renderScene={(route, navigator) => {
          switch(route.name){
            case 'UserProfile':
              return (
                <UserProfile
                  {...this.props}
                  {...route}
                  navigator={navigator}
                />
            );
            case 'UserSettings':
              return (
                <UserSettings
                  {...this.props}
                  {...route}
                  navigator={navigator}
                />
            );
            case 'UserTechnologies':
              return (
                <UserTechnologies
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
};

export default ProfileView;

```

And then we can render the components `UserProfile`, `UserSettings`, and `UserTechnologies`, reusing parts of our registration form.

```javascript
/* application/components/profile/UserProfile.js */
import Icon from 'react-native-vector-icons/Ionicons';
import ImagePicker from 'react-native-image-picker';
import NavigationBar from 'react-native-navbar';
import React, { Component } from 'react';
import { 
  Image, 
  ScrollView, 
  Text, 
  TouchableOpacity, 
  View 
} from 'react-native';

import Colors from '../../styles/colors';
import { ImageOptions, Headers } from '../../fixtures';
import { API, DEV } from '../../config';
import { globals, profileStyles } from '../../styles';

const styles = profileStyles;

class UserProfile extends Component{
  constructor(){
    super();
    this.visitTechnologies = this.visitTechnologies.bind(this);
    this.showImagePicker = this.showImagePicker.bind(this);
    this.visitSettings = this.visitSettings.bind(this);
  }
  showImagePicker(){ /* select image from camera roll for avatar */
    ImagePicker.showImagePicker(ImageOptions, (response) => {
      if (response.didCancel || response.error) { return; }
      const avatar = 'data:image/png;base64,' + response.data;
      fetch(`${API}/users/${this.props.currentUser.id}`, {
        method: 'PUT',
        headers: Headers,
        body: JSON.stringify({ avatar: avatar })
      })
      .then(response => response.json())
      .then(user => this.props.updateUser(user))
      .catch(err => console.log('ERR:', err))
      .done();
    });
  }
  visitTechnologies(){
    this.props.navigator.push({ 
      name: 'UserTechnologies', 
      currentUser: this.props.currentUser 
    });
  }
  visitSettings(){
    this.props.navigator.push({ 
      name: 'UserSettings', 
      currentUser: this.props.currentUser 
    });
  }
  render() {
    let { currentUser } = this.props;
    let titleConfig = { title: 'Profile', tintColor: 'white' };
    return (
      <View style={[globals.flexContainer, globals.inactive]}>
        <NavigationBar
          title={titleConfig}
          tintColor={Colors.brandPrimary}
        />
        <ScrollView style={globals.flex}>
          <View style={styles.flexRow}>
            <TouchableOpacity 
              style={[globals.flexCenter, globals.pv1]} 
              onPress={this.showImagePicker}
            >
              <Image 
                source={{uri: currentUser.avatar}} 
                style={styles.avatar}
              />
            </TouchableOpacity>
            <View style={styles.infoContainer}>
              <Text style={globals.h4}>
                {currentUser.firstName} {currentUser.lastName}
              </Text>
              <Text style={globals.h5}>
                {currentUser.location.city.long_name}, {currentUser.location.state.short_name}
              </Text>
            </View>
          </View>
          <TouchableOpacity 
            style={styles.formButton} 
            onPress={this.visitTechnologies}
          >
            <Text style={globals.h4}>
              My Technologies
            </Text>
            <Icon name='ios-arrow-forward' size={30} color='#ccc' />
          </TouchableOpacity>
          <TouchableOpacity 
            style={styles.formButton} 
            onPress={this.visitSettings}
          >
            <Text style={globals.h4}>
              Settings
            </Text>
            <Icon name='ios-arrow-forward' size={30} color='#ccc' />
          </TouchableOpacity>
          <TouchableOpacity 
            style={styles.logoutButton} 
            onPress={this.props.logout}
          >
            <Text style={styles.logoutText}>
              Logout
            </Text>
          </TouchableOpacity>
        </ScrollView>
      </View>
    );
  }
};

export default UserProfile;
```

```javascript
/* application/components/profile/UserSettings.js */
import Icon from 'react-native-vector-icons/Ionicons';
import NavigationBar from 'react-native-navbar';
import React, { Component } from 'react';
import { 
  Text, 
  View, 
  TextInput, 
  TouchableOpacity, 
  Dimensions 
} from 'react-native';
import { GooglePlacesAutocomplete } from 'react-native-google-places-autocomplete';
import { KeyboardAwareScrollView } from 'react-native-keyboard-aware-scroll-view';
import { find } from 'underscore';

import Colors from '../../styles/colors';
import { Headers } from '../../fixtures';
import BackButton from '../shared/BackButton';
import { GooglePlacesCityConfig } from '../../config';
import { DEV, API } from '../../config';
import { globals, formStyles, autocompleteStyles } from '../../styles';

const styles = formStyles;
const { height: deviceHeight, width: deviceWidth } = Dimensions.get('window');

function setErrorMsg({ location, firstName, lastName, email }){
  if (typeof location !== 'object' || ! location.city ) {
    return 'Must provide valid location.';
  } else if (firstName === ''){
    return 'Must provide a valid first name.';
  } else if (lastName === '') {
    return 'Must provide a valid last name.';
  } else if (email === ''){
    return 'Must provide a valid email address.';
  } else {
    return '';
  }
}
class UserSettings extends Component{
  constructor(props){
    super(props);
    this.goBack = this.goBack.bind(this);
    this.saveSettings = this.saveSettings.bind(this);
    this.selectLocation = this.selectLocation.bind(this);
    this.state = {
      email     : props.currentUser.username,
      errorMsg  : '',
      firstName : props.currentUser.firstName,
      lastName  : props.currentUser.lastName,
      location  : props.currentUser.location,
    }
  }
  saveSettings(){
    let errorMsg = setErrorMsg(this.state);
    if (errorMsg !== '') {
      this.setState({ errorMsg }); return;
    }
    let user = {
      location: this.state.location,
      firstName: this.state.firstName,
      lastName: this.state.lastName,
      email: this.state.email
    };
    fetch(`${API}/users/${this.props.currentUser.id}`, {
      method: 'PUT',
      headers: Headers,
      body: JSON.stringify(user)
    })
    .then(response => response.json())
    .then(user => this.updateUser(user))
    .catch(err => console.log('ERR:', err))
    .done();
  }
  updateUser(user){
    this.props.updateUser(user);
    this.goBack();
  }
  selectLocation(data, details){
    if ( ! details ) { return; }
    let location = {
      ...details.geometry.location,
      city: find(details.address_components, (c) => c.types[0] === 'locality'),
      state: find(details.address_components, (c) => c.types[0] === 'administrative_area_level_1'),
      county: find(details.address_components, (c) => c.types[0] === 'administrative_area_level_2'),
      formattedAddress: details.formatted_address
    };
    this.setState({ location });
  }
  goBack(){
    this.props.navigator.pop();
  }
  render(){
    return (
      <View style={[globals.flexContainer, globals.inactive]}>
        <NavigationBar
          title={{ title: 'User Settings', tintColor: 'white' }}
          tintColor={Colors.brandPrimary}
          leftButton={<BackButton handlePress={this.goBack}/>}
        />
        <KeyboardAwareScrollView style={[styles.formContainer, globals.mt1]}>
          <Text style={styles.h4}>{"* Where are you looking for assemblies?"}</Text>
          <View ref="location" style={{flex: 1,}}>
            <GooglePlacesAutocomplete
              styles={autocompleteStyles}
              placeholder='Your city'
              minLength={2}
              autoFocus={false}
              fetchDetails={true}
              onPress={this.selectLocation}
              getDefaultValue={() => {return this.state.location.city.long_name;}}
              query={GooglePlacesCityConfig}
              currentLocation={false}
              currentLocationLabel="Current location"
              nearbyPlacesAPI='GooglePlacesSearch'
              GoogleReverseGeocodingQuery={{}}
              GooglePlacesSearchQuery={{rankby: 'distance',}}
              filterReverseGeocodingByTypes={['street_address']}
              predefinedPlaces={[]}>
            </GooglePlacesAutocomplete>
          </View>
          <Text style={styles.h4}>* Email</Text>
          <View style={styles.formField}>
            <TextInput
              ref={(el) => this.email = el }
              returnKeyType="next"
              onChangeText={(email) => this.setState({ email })}
              keyboardType="email-address"
              autoCapitalize="none"
              maxLength={144}
              value={this.state.email}
              placeholderTextColor='#bbb'
              style={styles.input}
              placeholder="Your email address"
            />
          </View>
          <Text style={styles.h4}>* First Name</Text>
          <View style={styles.formField}>
            <TextInput
              ref={(el) => this.firstName = el }
              returnKeyType="next"
              maxLength={20}
              value={this.state.firstName}
              onChangeText={(firstName) => this.setState({ firstName })}
              placeholderTextColor='#bbb'
              style={styles.input}
              placeholder="Your first name"
            />
          </View>
          <Text style={styles.h4}>* Last name</Text>
          <View style={styles.formField} ref="lastName">
            <TextInput
              returnKeyType="next"
              maxLength={20}
              ref={(el) => this.lastName = el }
              onChangeText={(lastName) => this.setState({ lastName })}
              placeholderTextColor='#bbb'
              value={this.state.lastName}
              style={styles.input}
              placeholder="Your last name"
            />
         </View>
        </KeyboardAwareScrollView>
        <TouchableOpacity 
          style={[styles.submitButton, styles.buttonMargin]} 
          onPress={this.saveSettings}
        >
          <Text style={globals.largeButtonText}>
            SAVE
          </Text>
        </TouchableOpacity>
      </View>
    )
  }
}

export default UserSettings;


```

```javascript
/* application/components/profile/UserSettings.js */

import Icon from 'react-native-vector-icons/Ionicons';
import NavigationBar from 'react-native-navbar';
import Dropdown, { Select, Option, OptionList } from 'react-native-selectme';
import React, { Component } from 'react';
import { 
  Text, 
  View, 
  ScrollView, 
  TextInput, 
  TouchableOpacity, 
  Dimensions 
} from 'react-native';
import { KeyboardAwareScrollView } from 'react-native-keyboard-aware-scroll-view';
import { uniq } from 'underscore';
import TechnologyList from '../shared/TechnologyList';

import Colors from '../../styles/colors';
import BackButton from '../shared/BackButton';
import {DEV, API} from '../../config';
import { Technologies, Headers } from '../../fixtures';
import { 
  globals, 
  formStyles, 
  selectStyles, 
  optionTextStyles, 
  overlayStyles 
} from '../../styles';

const styles = formStyles;
const { height: deviceHeight, width: deviceWidth } = Dimensions.get('window');

class UserTechnologies extends Component{
  constructor(props){
    super(props);
    this.goBack = this.goBack.bind(this);
    this.selectTechnology = this.selectTechnology.bind(this);
    this.removeTechnology = this.removeTechnology.bind(this);
    this.saveSettings = this.saveSettings.bind(this);
    this.state = {
      technologies: props.currentUser.technologies,
      errorMsg: '',
    }
  }
  selectTechnology(technology){
    this.setState({
      technologies: uniq(this.state.technologies.concat(technology))
    });
  }
  removeTechnology(index){
    let { technologies } = this.state;
    this.setState({
      technologies: [
      ...technologies.slice(0, index),
      ...technologies.slice(index + 1)
      ]
    })
  }
  goBack(){
    this.props.navigator.pop();
  }
  saveSettings(){
    fetch(`${API}/users/${this.props.currentUser.id}`, {
      method: 'PUT',
      headers: Headers,
      body: JSON.stringify({ technologies: this.state.technologies })
    })
    .then(response => response.json())
    .then(user => {
      this.props.updateUser(user);
      this.goBack();
    })
    .catch(err => {})
    .done();
  }
  render(){
    let { technologies } = this.state;
    return (
      <View style={[globals.flexContainer, globals.inactive]}>
        <NavigationBar
          title={{ title: 'User Technologies', tintColor: 'white' }}
          tintColor={Colors.brandPrimary}
          leftButton={<BackButton handlePress={this.goBack}/>}
        />
        <KeyboardAwareScrollView 
          style={[styles.formContainer, globals.mt1]} 
          contentInset={{bottom: 49}}
        >
          <View style={globals.flex}>
            <Text style={styles.h4}>
              {"Select technologies"}
            </Text>
            <Select
              width={deviceWidth}
              height={55}
              styleText={optionTextStyles}
              style={selectStyles}
              optionListRef={() => this.options}
              defaultValue="Add a technology"
              onSelect={this.selectTechnology}>
              {Technologies.map((tech, idx) => (
                <Option styleText={optionTextStyles} key={idx}>
                  {tech}
                </Option>
              ))}
            </Select>
            <OptionList 
              ref={(el) => this.options = el } 
              overlayStyles={overlayStyles}
            />
          </View>
          <TechnologyList 
            technologies={technologies} 
            handlePress={this.removeTechnology}
          />
        </KeyboardAwareScrollView>
        <TouchableOpacity 
          style={[styles.submitButton, styles.buttonMargin]} 
          onPress={this.saveSettings}
        >
          <Text style={globals.largeButtonText}>
            SAVE
          </Text>
        </TouchableOpacity>
      </View>
    )
  }
}

export default UserTechnologies;

```

Let's review:
- In **ProfileView**, we establish the different routes we will be using and pass on `this.props` to each child component. 
- In **UserProfile**, we offer the option to update the user avatar through our **ImagePicker** component. We also give the option to route to **UserTechnologies** or **UserSettings**
- **UserSettings** is basically reusing the same elements of our **Register** component from user accounts. The save button updates our user through a database call and updates the component state of our top-level **Navigator** component. Make sure on **index.ios.js** that the `updateUser` is passed to **Dashboard**:
 
```javascript
/* index.ios.js */
/* ... */
case 'Dashboard':
  return (
    <Dashboard
      updateUser={this.updateUser}
      navigator={navigator}
      logout={this.logout}
      user={this.state.user}
    />
);
/* ... */
```
```javascript
/* application/components/Dashboard.js */
/* ... */
  <ProfileView currentUser={user} logout={this.logout} updateUser={this.props.updateUser}/>
/* ... */
```
Notice how we were able to re-use a lot of styles / parts of our previous components. Optimizing this makes development faster and more enjoyable. Always remember the concept of DRY, don't repeat yourself. 

Let's commit here.

[Commit](https://github.com/buildreactnative/assemblies-tutorial/tree/c7ba88dca60c3df08e03e178e7fba64ed8eb23d4) - "Render Profile components"

![profile](/images/chapter-12/profile-1.png)
![profile](/images/chapter-12/profile-2.png)
![profile](/images/chapter-12/profile-3.png)

