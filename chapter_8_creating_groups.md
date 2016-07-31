# Chapter 8: Creating Groups

In the last chapter, we built out a messaging feature. We were able to fetch messages related to a group of users and create new messages. Now we’ll focus on adding some new features to our tab-bar navigation – **Groups** and **Calendar**.

Before we get started, let's make the development process easier and add persistent user login. This way, every time we refresh the app, we won't have to login. 

### Persistent User Login

To do this, when a user logs in, we need to store their session id in local storage. We use React Native's **AsyncStorage** module for this. **AsyncStorage** works as a simple dictionary of keys and values, with a **getItem** and **setItem** method. Let's edit **Login.js** and **RegisterConfirmation.js** to save this session id.

```javascript
/* application/components/accounts/Login.js */
/* ... */
import { 
  Text, 
  View, 
  ScrollView, 
  TextInput, 
  TouchableOpacity, 
  AsyncStorage 
} from 'react-native';
/* ... */
fetchUserInfo(sid){
  AsyncStorage.setItem('sid', sid);
  fetch(`${API}/users/me`, { headers: extend(Headers, { 'Set-Cookie': `sid=${sid}`}) })
  .then(response => response.json())
  .then(user => this.updateUserInfo(user))
  .catch(err => this.connectionError())
  .done();
}
/* ... */
```

```javascript
/* application/components/RegisterConfirmation.js */
/* ... */
import { 
  Image, 
  ScrollView, 
  Text, 
  TouchableOpacity, 
  View, 
  Dimensions, 
  AsyncStorage 
} from 'react-native';
/* ... */
getUserInfo(sid){ 
  AsyncStorage.setItem('sid', sid);
  fetch(`${API}/users/me`, { 
    headers: extend(Headers, { 'Set-Cookie': `sid=${sid}`}) 
  })
  .then(response => response.json())
  .then(user => {
    this.props.updateUser(user);
    this.props.navigator.push({
      name: 'Dashboard'
    });
  })
  .catch((err) => {})
  .done();
}
/* ... */
```

Now in our main **index.ios.js** we can modify our component to first check if a session id is saved. If it is, we can fetch the user information and direct the user to the dashboard. If not, we load the landing screen as per usual.

```javascript
/* index.ios.js */
import React, { Component } from 'react';
import {
  AppRegistry,
  ActivityIndicator,
  View,
  Navigator,
  AsyncStorage
} from 'react-native';

import Landing from './application/components/Landing';
import Register from './application/components/accounts/Register';
import RegisterConfirmation from './application/components/accounts/RegisterConfirmation';
import Login from './application/components/accounts/Login';
import Dashboard from './application/components/Dashboard';
import { Headers } from './application/fixtures';
import { extend } from 'underscore';
import { API, DEV } from './application/config';
import { globals } from './application/styles';

const Loading = () => (
  <View style={globals.flexCenter}>
    <ActivityIndicator size='large'/>
  </View>
);

class assemblies extends Component {
  constructor(){
    super();
    this.logout = this.logout.bind(this);
    this.updateUser = this.updateUser.bind(this);
    this.state = {
      user          : null,
      ready         : false,
      initialRoute  : 'Landing',
    }
  }
  componentDidMount(){
    this._loadLoginCredentials()
  }
  async _loadLoginCredentials(){
    try {
      let sid = await AsyncStorage.getItem('sid');
      console.log('SID', sid);
      if (sid){
        this.fetchUser(sid);
      } else {
        this.ready();
      }
    } catch (err) {
      this.ready(err);
    }
  }
  ready(err){
    this.setState({ ready: true });
  }
  fetchUser(sid){
    fetch(`${API}/users/me`, { 
      headers: extend(Headers, { 'Set-Cookie': `sid=${sid}`})
    })
    .then(response => response.json())
    .then(user => this.setState({ 
      ready: true, 
      initialRoute: 'Dashboard',
      user
    }))
    .catch(err => this.ready(err))
    .done();
  }
  logout(){
    this.nav.push({ name: 'Landing' })
  }
  updateUser(user){
    this.setState({ user });
  }
  render() {
    if ( ! this.state.ready ) { return <Loading /> }
    return (
      <Navigator
        style={globals.flex}
        ref={(el) => this.nav = el }
        initialRoute={{ name: this.state.initialRoute }}
        renderScene={(route, navigator) => {
          switch(route.name){
            case 'Landing':
              return (
                <Landing navigator={navigator}/>
            );
            case 'Dashboard':
              return (
                <Dashboard
                  navigator={navigator}
                  logout={this.logout}
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
          }
        }}
      />
    );
  }
}

AppRegistry.registerComponent('assemblies', () => assemblies);

```

Now if you log in and refresh the screen, you should still be logged in!

![login](/images/chapter-8/login-1.png)


A few things:
- We are using the **async** and **await** functionality to retrieve the local storage information. **await** is used to ensure that the value is properly retrieved before running subsequent code.
- We use React Native's **ActivityIndicator** component to show a spinner while the session id is being fetched. We can move this to a separate file for reuse, like **application/components/shared/Loading.js**.
- We also make our **initialRoute** dynamic, meaning that it is dependant on **this.state.initialRoute**. If our async function returns false, the landing page will render as usual. If it is successful, the value of **this.state.initialRoute** will be set to **Dashboard**, and the app will automatically log our user in.

[Commit](https://github.com/buildreactnative/assemblies-tutorial/tree/161c53bf7f30d9098a231805a47411827c436a4d) - "Add persistent user login"

### Adding a Groups and Calendar Tab

Now we want to add a tab for showing groups and for showing a calendar view. Let’s edit **Dashboard.js** and add the two files **application/calendar/CalendarView.js** and **application/groups/GroupsView.js**. 

```javascript
/* application/components/Dashboard.js */
/* … */
import CalendarView from './calendar/CalendarView';
import GroupsView from './groups/GroupsView';
/* … */
<TabBarItemIOS
  title='Groups'
  selected={ this.state.selectedTab == 'Groups' }
  iconName='ios-people'
  onPress={() => this.setState({ selectedTab: 'Groups' })}
>
  <GroupsView currentUser={user}/>
</TabBarItemIOS>
<TabBarItemIOS
  title='Calendar'
  selected={ this.state.selectedTab == 'Calendar' }
  iconName='ios-calendar'
  onPress={() => this.setState({ selectedTab: 'Calendar'})}
>
  <CalendarView currentUser={user}/>
</TabBarItemIOS>
/* … */
```
```javascript
/* application/components/calendar/CalendarView.js */

import React, { Component } from 'react';
import { View, Text } from 'react-native';
import { globals } from '../../styles';

class CalendarView extends Component{
  render(){
    return (
      <View style={globals.flexCenter}>
        <Text style={globals.h2}>CALENDAR VIEW</Text>
      </View>
    )
  }
};

export default CalendarView;
```
```javascript
/* application/components/groups/GroupsView.js */

import React, { Component } from 'react';
import { View, Text } from 'react-native';
import { globals } from '../../styles';

class GroupsView extends Component{
  render(){
    return (
      <View style={globals.flexCenter}>
        <Text style={globals.h2}>GROUPS VIEW</Text>
      </View>
    )
  }
};

export default GroupsView;
```

![groups](/images/chapter-8/groups-view-1.png)


### Rendering Groups

We want to start off by rendering some groups. But we haven’t created any. How can we get around this? We can create some data, of course! Let’s create some groups in our Deployd dashboard **localhost:2403/dashboard**, and then render them in our groups view.

In the Deployd dashboard, add the following groups, replacing the **USER_ID** with your personal user **id**.
```
name: “React Native NYC”,
description: “A meetup for people interested in learning React Native, the mobile development library created by Facebook.”,
createdAt: 1469976754482,
members: [
    {
        "userId": "USER_ID",
        "confirmed": true,
        "role": "owner",
        "joinedOn": 1468113633150
    }
],
color: "blue",
image: “https://s3-us-west-2.amazonaws.com/assembliesapp/welcome%402x.png”,
technologies: [“React Native”, "JavaScript" ],
location: {
	"lat": 41.308274,
	"lng": -72.9278835,
	"city": {
		"long_name": "New Haven",
		"short_name": "New Haven",
		"types": [
			"locality",
			"political"
		]
	},
	"state": {
		"long_name": "Connecticut",
		"short_name": "CT",
		"types": [
			"administrative_area_level_1",
			"political"
		]
	},
	"county": {
		"long_name": "New Haven County",
		"short_name": "New Haven County",
		"types": [
			"administrative_area_level_2",
			"political"
		]
	},
	"formattedAddress": "New Haven, CT, USA"
}	

```

Now copy the data with new titles and descriptions, such as : 

```
title: “Machine Learning NYC”
description: “Meetup for machine learning enthusiasts”

title: “Python NYC”
description: “Meetup for Python enthusiasts”

title: “Ruby NYC”
description: “Meetup for Ruby enthusiasts”
```

![groups](/images/chapter-8/deployd-groups-1.png)

Now we’re ready to fetch these groups and render them in our `GroupsView` page! Make to make one of them has an empty array `[]` as the value for **members**. This way, they will show up in our suggested groups.

First, we have to turn **GroupsView** into another **Navigator** component. We will set **Groups** as our initial route, and render the a blank screen in **Groups.js**.

```javascript
import React, { Component } from 'react';
import { Navigator } from 'react-native';
import { find, isEqual } from 'underscore';

import Groups from './Groups';
import Headers from '../../fixtures';
import { API, DEV } from '../../config';
import { globals } from '../../styles';

class GroupsView extends Component{
  constructor(){
    super();
    this.state = {
      groups            : [],
      ready             : false,
      suggestedGroups   : [],
    }
  }
  componentWillMount(){
    this._loadGroups(this.props.currentUser);
  }
  _loadGroups(currentUser){ /* fetch all groups that the current user belongs to */
    let query = {
      members: { $elemMatch: { userId: currentUser.id } },
      $limit: 10
    };
    fetch(`${API}/groups/?${JSON.stringify(query)}`)
    .then(response => response.json())
    .then(groups => this._loadSuggestedGroups(groups))
    .catch(err => this.ready(err))
    .done();
  }
  _loadSuggestedGroups(groups){
    this.setState({ groups, ready: true });
    let query = { /* query groups that the user does not belong to but are nearby */
      id: { $nin: groups.map(group => group.id) },
      'location.city.long_name': this.props.currentUser.location.city.long_name,
      $limit: 4
    };
    fetch(`${API}/groups/?${JSON.stringify(query)}`)
    .then(response => response.json())
    .then(suggestedGroups => this.setState({ suggestedGroups }))
    .catch(err => this.ready(err))
    .done();
  }
  ready(err){
    this.setState({ ready: true })
  }
  render(){
    return (
      <Navigator
        style={globals.flex}
        initialRoute={{ name: 'Groups' }}
        renderScene={(route, navigator) => {
          switch(route.name){
            case 'Groups':
              return (
                <Groups
                  {...this.props}
                  {...this.state}
                  navigator={navigator}
                />
            );
          }
        }}
      />
    );
  }
};

export default GroupsView;
```
```javascript
/* application/components/groups/Groups.js */

import React, { Component } from 'react';
import { View, Text } from 'react-native';
import { globals } from '../../styles';

class Groups extends Component{
  render(){
    return (
      <View style={globals.flexCenter}>
        <Text style={globals.h2}>GROUPS VIEW</Text>
        {this.props.groups.map((group, idx) => (
          <Text key={idx}>{group.name}</Text>
        ))}
      </View>
    )
  }
};

export default Groups;
```
![groups](/images/chapter-8/groups-view-2.png)


Let's go over the above code:
- As before, we're setting **GroupsView** to be a **Navigator** component, this time with only one route so far, **Groups**. The **Groups** component expect **props** of an array of groups and suggested groups. We fetch these in the `componentDidMount` method of **GroupsView**, using MongoDB queries. 
- The `$elemMatch` query in Mongo looks for nested values in an object. Here, our **members** fields of the **groups** collection is an object with a nested field of **userId**. We search for groups that have a **members** object with the **userId** of our current user.
- We use the `$nin` Mongo query to find groups that are in the same city as our user, but that do **not** have the same id as any of the groups that the user belongs to. This way we can show groups that the user might be interested in.
- In **Groups.js**, we use the groups that we fetched and render a simple **Text** component with each group's name. We will flesh this out more next.

### Rendering Groups

Now that we've successfully fetched our data, we need to render it properly. We should also make sure that while the value `ready` is set to `false`, we should load our loading spinner (which we moved to **application/components/shared/Loading.js**). Here is the updated **Groups.js**:

```javascript
import React, { Component } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  Image,
  ScrollView
} from 'react-native';

import Icon from 'react-native-vector-icons/MaterialIcons';
import NavigationBar from 'react-native-navbar';
import Colors from '../../styles/colors';
import Loading from '../shared/Loading';
import { globals, groupsStyles } from '../../styles';
const styles = groupsStyles;

export function formatGroups(groups){
  if (groups.length % 2 === 1 ){
    return groups.concat(null);
  } else {
    return groups;
  }
};

const AddGroupBox = ({ handlePress }) => (
  <TouchableOpacity
    onPress={handlePress}
    style={styles.groupImage}>
    <View style={[styles.groupBackground, globals.inactive]} >
      <Icon name="add-circle" size={60} color={Colors.brandPrimary} />
    </View>
  </TouchableOpacity>
);

export const EmptyGroupBox = () => (
  <View style={styles.groupsContainer}>
    <View style={styles.groupImage}>
      <View style={[styles.groupBackground, globals.inactive]} />
    </View>
  </View>
);

const EmptyGroupBoxes = ({ handlePress }) => (
  <View style={styles.boxContainer}>
    <View style={styles.groupsContainer}>
      <AddGroupBox handlePress={handlePress}/>
      <EmptyGroupBox />
    </View>
  </View>
);

const EmptySuggestedGroupBoxes = () => (
  <View style={styles.boxContainer}>
    <View style={globals.flexRow}>
      <EmptyGroupBox />
      <EmptyGroupBox />
    </View>
  </View>
)

export const GroupBoxes = ({ groups, visitGroup, visitCreateGroup }) => {
  console.log('GROUPS', groups);
  if (! groups.length ) {
    return <EmptyGroupBoxes handlePress={visitCreateGroup}/>
  }
  return (
    <View style={styles.boxContainer}>
      {groups.map((group, idx) => {
        if (!group) { return <EmptyGroupBox key={idx}/>}
        return (
          <TouchableOpacity 
            key={idx} 
            style={globals.flexRow} 
            onPress={() => visitGroup(group)}
          >
            <Image 
              source={{uri: group.image}} 
              style={styles.groupImage}
            >
              <View style={[styles.groupBackground, {backgroundColor: group.color,}]} >
                <Text style={styles.groupText}>
                  {group.name}
                </Text>
              </View>
            </Image>
          </TouchableOpacity>
        )
      })}
    </View>
  );
}

const SuggestedGroupBoxes = ({ groups, visitGroup }) => {
  if (! groups.length ) { return <EmptySuggestedGroupBoxes /> }
  return (
    <View style={styles.boxContainer}>
      {groups.map((group, idx) => {
        if (!group) { return <EmptyGroupBox key={idx}/>}
        return (
          <TouchableOpacity 
            key={idx} 
            style={globals.flexRow} 
            onPress={() => visitGroup(group)}
          >
            <Image 
              source={{uri: group.image}} 
              style={styles.groupImage}
            >
              <View style={[styles.groupBackground, {backgroundColor: group.color,}]} >
                <Text style={styles.groupText}>
                  {group.name}
                </Text>
              </View>
            </Image>
          </TouchableOpacity>
        );
      })}
    </View>
  );
};

const AddButton = ({ handlePress }) => (
  <TouchableOpacity style={globals.pa1} onPress={handlePress}>
    <Icon name="add-circle" size={25} color="#ccc" />
  </TouchableOpacity>
)

class Groups extends Component{
  constructor(){
    super();
    this.visitCreateGroup = this.visitCreateGroup.bind(this);
    this.visitGroup = this.visitGroup.bind(this);
  }
  visitGroup(group){
    this.props.navigator.push({
      name: 'Group',
      group
    })
  }
  visitCreateGroup(){
    this.props.navigator.push({ name: 'CreateGroup' })
  }
  render(){
    let { groups, suggestedGroups, ready, navigator } = this.props;
    if (! ready ) { return <Loading /> }
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          title={{title: 'My Groups', tintColor: 'white'}}
          tintColor={Colors.brandPrimary}
          rightButton={<AddButton handlePress={this.visitCreateGroup}/>}
        />
        <ScrollView style={[globals.flex, globals.mt1]}>
          <Text style={[globals.h4, globals.mh2]}>Your Assemblies</Text>
          <GroupBoxes
            groups={formatGroups(groups)}
            navigator={navigator}
            visitGroup={this.visitGroup}
            visitCreateGroup={this.visitCreateGroup}
          />
          <Text style={[globals.h4, globals.mh2]}>You Might Like</Text>
          <SuggestedGroupBoxes
            groups={formatGroups(suggestedGroups)}
            navigator={navigator}
            visitGroup={this.visitGroup}
          />
        </ScrollView>
      </View>
    )
  }
};

export default Groups;

```

![groups](/images/chapter-8/groups-view-3.png)

Let's review what we just did:
- We add a **formatGroups** function to ensure that there is an even number of both groups and suggested groups. This is to maintain an even layout with Flexbox's **flexWrap** quality.
- If a group is null, we render an empty box, that's all. We refactor this empty box into the component **EmptyGroupBox**.
- If there are no groups at all, we just need to render empty boxes, except we want one of the boxes to have a call to action, or CTA. That is where our **AddGroupBox**` comes in. It has a `handlePress` method, that when press, should redirect to a form to create a new group.
- We render our suggested groups in the same way, with the exception that if there are no suggested groups, we can just show two empty boxes. That is what our **EmptySuggestedGroupBoxes** component does.
- Each of the **GroupBoxes** elements has a callback when pressed that routes the user to the screen of that particular group. 
- You'll also notice that we add a **AddButton** component as our **rightButton** property of the navigation bar. This is another way that a user can initiate the group creation process.
- We still have to wire up the new routes of **Group** and **CreateGroup**, but we're well on our way!

Let's make a commit here.

[Commit]() - "Render groups in main Groups screen"


### Loading State

Just as we did for our groups screen, it's a good standard to show a spinner or other animation when making any data fetch. Let's implement this in our `Conversations` component as well.

```javascript
/* application/components/messages/Conversations.js */
/* ... */
import Loading from '../shared/Loading';
/* ... */
class Conversations extends Component{
  /* .... */
  dataSource(){
    return (
      new ListView.DataSource({ rowHasChanged: rowHasChanged })
      .cloneWithRows(this.props.conversations)
    );
  }
  render() {
    if (! this.props.ready) { return <Loading/> }
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          title={{ title: 'Messages', tintColor: 'white' }}
          tintColor={Colors.brandPrimary}
        />
        <ListView
          enableEmptySections={true}
          dataSource={this.dataSource()}
          contentInset={{ bottom: 49 }}
          renderRow={this._renderRow}
        />
      </View>
    );
  }
};

export default Conversations;
```

Now if we refresh and visit the messages tab, we should see a spinner before the data is loaded. A better experience, don't you think?

### Creating Groups

Out of the four operations of **create**, **read**, **update**, and **delete**, we have accomplished the **read** part of our **groups** collection. Now what about creating groups? 

Well, we can certainly reuse elements from our user login and registration forms to make it easier. What information are we looking for? 

```
name: String
description: String
location: Object
technologies: Array
color: String
image: String
```

The last two fields are really optional, but we can include them in our form too. Let's design another two-part form to create a feed, somewhat similar to our user registration process. We will need routes in our parent-level **GroupsView** component for both **CreateFeed** and **CreateFeedConfirmation**. Let's add those in.

```javascript
/* application/components/groups/GroupsView.js */
/* ... */
import CreateGroup from './CreateGroup';
import CreateGroupConfirmation from './CreateGroupConfirmation';
/* ... */
switch(route.name){
  case 'Groups':
    return (
      <Groups
        {...this.props}
        {...this.state}
        navigator={navigator}
      />
  );
  case 'CreateGroup':
    return (
      <CreateGroup
        {...this.props}
        {...this.state}
        {...route}
        navigator={navigator}
      />
  );
  case 'CreateGroupConfirmation':
    return (
      <CreateGroupConfirmation
        {...this.props}
        {...this.state}
        {...route}
        navigator={navigator}
      />
  );
}
...
```

Now of course we have to define **CreateGroup.js** and **CreateGroupConfirmation.js**.

```javascript
/* application/components/groups/CreateGroup.js */

import React, { Component } from 'react';
import { View, Text } from 'react-native';
import NavigationBar from 'react-native-navbar';
import Icon from 'react-native-vector-icons/Ionicons';

import BackButton from '../shared/BackButton';
import { globals } from '../../styles';

class CreateGroup extends Component{
  constructor(){
    super();
    this.goBack = this.goBack.bind(this);
  }
  goBack(){
    this.props.navigator.pop();
  }
  render(){
    let titleConfig = { title: 'Create Group', tintColor: 'white' };
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          title={titleConfig}
          tintColor={Colors.brandPrimary}
          leftButton={<BackButton handlePress={this.goBack}/>}
        />
        <View style={globals.flexCenter}>
          <Text style={globals.h2}>
            CreateGroup
          </Text>
        </View>
      </View>
    );
  }
};

export default CreateGroup;

```

```javascript
/* application/components/groups/CreateGroupConfirmation.js */
import React, { Component } from 'react';
import { View, Text } from 'react-native';
import NavigationBar from 'react-native-navbar';
import Icon from 'react-native-vector-icons/Ionicons';

import BackButton from '../shared/BackButton';
import { globals } from '../../styles';

class CreateGroupConfirmation extends Component{
  constructor(){
    super();
    this.goBack = this.goBack.bind(this);
  }
  goBack(){
    this.props.navigator.pop();
  }
  render(){
    let titleConfig = { title: 'Create Group', tintColor: 'white' };
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          title={titleConfig}
          tintColor={Colors.brandPrimary}
          leftButton={<BackButton handlePress={this.goBack}/>}
        />
        <View style={globals.flexCenter}>
          <Text style={globals.h2}>
            CreateGroupConfirmation
          </Text>
        </View>
      </View>
    );
  }
};

export default CreateGroupConfirmation;
```

![group](/images/chapter-8/create-group-view-1.png)

Now we link to our new routes when a user presses the **Add Group** button in **Groups.js**.

Now we need to flesh out the form a bit. First let's install another **npm** package - **react-native-keyboard-aware-scroll-view**. This package will ensure that our input fields don't get hidden by the keyboard. We simply use it in the same way as we have used the **ScrollView** component.

```javascript
/* application/components/groups/CreateGroup.js */

import Icon from 'react-native-vector-icons/Ionicons';
import NavigationBar from 'react-native-navbar';
import React, { Component } from 'react';
import Config from 'react-native-config';
import { View, Text, TextInput, TouchableOpacity } from 'react-native';
import { GooglePlacesAutocomplete } from 'react-native-google-places-autocomplete';
import { KeyboardAwareScrollView } from 'react-native-keyboard-aware-scroll-view';
import { find, extend } from 'underscore';

import Colors from '../../styles/colors';
import BackButton from '../shared/BackButton';
import { globals, autocompleteStyles, formStyles } from '../../styles';

const styles = formStyles;

class CreateGroup extends Component{
  constructor(){
    super();
    this.handleSubmit = this.handleSubmit.bind(this);
    this.handlePress = this.handlePress.bind(this);
    this.goBack = this.goBack.bind(this);
    this.state = {
      description   : '',
      errorMsg      : '',
      location      : null,
      name          : '',
    }
  }
  handleSubmit(){
    let { name, location, summary, description } = this.state;
    this.props.navigator.push({
      name        : 'CreateGroupConfirmation',
      groupName   : name,
      description,
      location,
      summary,
    })
  }
  handlePress(data, details){
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
    let { navigator } = this.props;
    return (
      <View style={[globals.flexContainer, globals.inactive]}>
        <NavigationBar
          title={{ title: 'Create Assembly', tintColor: 'white' }}
          tintColor={Colors.brandPrimary}
          leftButton={<BackButton handlePress={this.goBack}/>}
        />
        <KeyboardAwareScrollView style={styles.formContainer} contentInset={{ bottom: 49}}>
          <Text style={styles.h4}>* Where is your group located?</Text>
          <GooglePlacesAutocomplete
            styles={autocompleteStyles}
            placeholder='Your city'
            minLength={2} 
            autoFocus={false}
            fetchDetails={true}
            onPress={this.handlePress}
            getDefaultValue={() => { return ''; }}
            query={{
              key: Config.GOOGLE_PLACES_API_KEY,
              language: 'en',
              types: '(cities)',
            }}
            currentLocation={false}
            currentLocationLabel="Current location"
            nearbyPlacesAPI='GooglePlacesSearch'
            GoogleReverseGeocodingQuery={{}}
            GooglePlacesSearchQuery={{ rankby: 'distance',}}
            filterReverseGeocodingByTypes={['locality', 'administrative_area_level_3']}
            predefinedPlaces={[]}
          />
          <Text style={styles.h4}>* Name of your assembly</Text>
          <View style={styles.formField}>
            <TextInput
              returnKeyType="next"
              autofocus={true}
              ref={(el) => this.name = el }
              onSubmitEditing={() => this.description.focus()}
              onChangeText={(name) => this.setState({ name })}
              placeholderTextColor='#bbb'
              style={styles.input}
              placeholder="Name of your assembly"
            />
          </View>
          <Text style={styles.h4}>Who should join and why?</Text>
          <TextInput
            ref={(el) => this.description = el }
            returnKeyType="next"
            blurOnSubmit={true}
            clearButtonMode='always'
            onChangeText={(text)=> this.setState({ description: text })}
            placeholderTextColor='#bbb'
            style={styles.largeInput}
            multiline={true}
            placeholder="Type a message to get people interested in your group..."
          />
        </KeyboardAwareScrollView>
        <TouchableOpacity
          onPress={this.handleSubmit}
          style={[styles.submitButton, styles.buttonMargin]}>
          <Text style={globals.largeButtonText}>Next</Text>
        </TouchableOpacity>
      </View>
    )
  }
}

export default CreateGroup;

```
![create group form](/images/chapter-8/create-assembly-1.png)

Let's go over this:
- Many parts of this form will look very similar. That's because they are the same components we used in our user registration process. This is the easy part. The next part we will have to be a bit more creative.
- Our `handleSubmit` method takes the saved component state and passes it to the next route - **CreateGroupConfirmation**. We pass in the field `groupName` because the field `name` is already taken by the `Navigator` component.

In the second part, we want our users to select a background image, a color, and a list of related technologies. Let's make a commit at this point.

[Commit](https://github.com/buildreactnative/assemblies-tutorial/tree/90d955ba98c87ecc330727a70ad143f2c91eb7ab) - "Render first part of group creation form"


### Confirmation Form for Creating Groups

Here is the interface of our `CreateGroupConfirmation` component. Notice many of the similarities between it and our `RegisterConfirmation` form from before. In both we are using `react-native-image-picker` to set user images, and we are using the `Dropdown` component to select technologies. The only novelty is selecting the colors, which implements the `flex-wrap` functionality of flexbox.

```javascript
/* application/components/groups/CreateGroupConfirmation.js */
import Icon from 'react-native-vector-icons/Ionicons';
import ImagePicker from 'react-native-image-picker';
import NavigationBar from 'react-native-navbar';
import Dropdown, {
  Select,
  Option,
  OptionList
} from 'react-native-selectme';
import React, { Component } from 'react';
import { View, Text, ScrollView, TouchableOpacity, Image, Dimensions } from 'react-native';
import { uniq } from 'underscore';

import Colors from '../../styles/colors';
import BackButton from '../shared/BackButton';
import { API, DEV } from '../../config';
import { Technologies, ImageOptions, DefaultAvatar, Headers } from '../../fixtures';
import { SolidColors, BackgroundImage } from '../../fixtures';
import { globals, formStyles, selectStyles, optionTextStyles, overlayStyles } from '../../styles';
const { width: deviceWidth, height: deviceHeight } = Dimensions.get('window');

const styles = formStyles;

const TechnologyList = ({ technologies, handlePress }) => (
  <View style={styles.textContainer}>
    {technologies.map((technology, idx) => (
      <TouchableOpacity key={idx} onPress={() => handlePress(idx)} style={styles.technology}>
        <Text style={[styles.h6, globals.primaryText]}>{technology}</Text>
      </TouchableOpacity>
    ))}
  </View>
)

class CreateGroupConfirmation extends Component{
  constructor(){
    super();
    this.handleSubmit = this.handleSubmit.bind(this);
    this.showImagePicker = this.showImagePicker.bind(this);
    this.selectTechnology = this.selectTechnology.bind(this);
    this.goBack = this.goBack.bind(this);
    this.removeTechnology = this.removeTechnology.bind(this);
    this.state = {
      color         : '#3F51B5',
      errorMsg      : '',
      image         : BackgroundImage,
      technologies  : [],
    }
  }
  handleSubmit(){
    /* TODO: submit group creation form */
  }
  showImagePicker(){ /* select image from camera roll for groupImage */
    ImagePicker.showImagePicker(ImageOptions, (response) => {
      if (response.didCancel || response.error) { return; }
      const image = 'data:image/png;base64,' + response.data;
      this.setState({ image });
    });
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
  render(){
    let { navigator } = this.props;
    let { technologies, image, color, errorMsg } = this.state;
    return (
      <View style={[globals.flexContainer, globals.inactive]}>
        <NavigationBar
          title={{ title: 'Create Assembly', tintColor: 'white' }}
          tintColor={Colors.brandPrimary}
          leftButton={<BackButton handlePress={this.goBack}/>}
        />
        <ScrollView style={styles.formContainer} contentInset={{ bottom: 49 }}>
          <Text style={styles.h4}>{"My technologies"}</Text>
          <Select
            width={deviceWidth}
            height={55}
            ref="select"
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
          <View>
            <TechnologyList technologies={technologies} handlePress={this.removeTechnology}/>
          </View>
          <TouchableOpacity style={styles.avatarContainer} onPress={this.showImagePicker.bind(this)}>
            <Icon name="ios-camera" size={30} color={Colors.brandPrimary}/>
            <Text style={[styles.h4, globals.primaryText]}>Add a Photo</Text>
          </TouchableOpacity>
          <View style={styles.groupImageContainer}>
            <Image source={{ uri: image ? image : BackgroundImage }} style={styles.groupImage}/>
          </View>
          <Text style={styles.h4}>What background color would you like?</Text>
          <View style={styles.colorsContainer}>
            {SolidColors.map((color, idx) => (
              <TouchableOpacity
                key={idx}
                style={[styles.colorBox, {backgroundColor: Colors[color], borderColor: this.state.color == Colors[color] ? Colors.highlight : 'transparent' }]}
                onPress={() => this.setState({color: Colors[color]})}
              >
              </TouchableOpacity>
            ))}
          </View>
          <View style={[styles.error, globals.ma1]}>
            <Text style={styles.errorText}>{errorMsg}</Text>
          </View>
        </ScrollView>
        <TouchableOpacity style={[styles.submitButton, styles.buttonMargin]} onPress={this.handleSubmit}>
          <Text style={globals.largeButtonText}>Create group</Text>
        </TouchableOpacity>
      </View>
    )
  }
};

export default CreateGroupConfirmation;

```
![create group confirmation](/images/chapter-8/create-group-confirmation.png)

Let's review:
- Just as in our registration component, we use the `react-native-selectme` package to select relevant technologies.
- Although we render the image differently, our use of the `react-native-image-picker` package is exactly the same as in user registration.

Now all we have to do is pass all of our data to the `handleSubmit` function and create a new group. We should also watch for errors and alert the user of any. Let’s fill in that function. We'll also create a function `setErrors` to check for errors.

```javascript
application/components/groups/CreateGroupConfirmation.js

function setErrorMsg({ location, name }){
  if (! location ){
    return 'You must provide a location.';
  } else if (! name ){
    return 'You must provide a name.';
  } else {
    return '';
  }
}
class CreateGroupConfirm extends Component{
  constructor(){
    super();
    this.handleSubmit = this.handleSubmit.bind(this);
    this.showImagePicker = this.showImagePicker.bind(this);
    this.selectTechnology = this.selectTechnology.bind(this);
    this.goBack = this.goBack.bind(this);
    this.removeTechnology = this.removeTechnology.bind(this);
    this.state = {
      color         : '#3F51B5',
      errorMsg      : '',
      image         : BackgroundImage,
      technologies  : [],
    }
  }
  handleSubmit(){
    let errorMsg = setErrorMsg(this.props);
    if (errorMsg !== '') { /* return error if missing information */
      this.setState({ errorMsg: errorMsg }); return;
    }
    let group = {
      color: this.state.color,
      image: this.state.image,
      technologies: this.state.technologies,
      description: this.props.description,
      location: this.props.location,
      name: this.props.groupName,
      members: [{
        userId: this.props.currentUser.id,
        role: 'owner',
        joinedAt: new Date().valueOf(),
        confirmed: true
      }],
      createdAt: new Date().valueOf()
    };
    fetch(`${API}/groups`, {
      method: 'POST',
      headers: Headers,
      body: JSON.stringify(group)
    })
    .then(response => response.json())
    .then(group => this.addGroup(group))
    .catch(err => { console.log('ERR', err)})
    .done();
  }
  addGroup(group){
    console.log('GROUP', group)
    this.props.addGroup(group);
    this.props.navigator.popToTop();
  }
...
```

Notice that we invoke a function `updateGroups` after the successful submission. This is because we want to update the `groups` array that we have in our parent `GroupsView` component. We then use `navigator` to navigate back to the groups screen. Let’s make sure we create the function `updateGroups` and that we add a new route `Group` with a corresponding component.

```javascript
application/components/groups/GroupsView.js
…
import Group from './Group';
…
constructor(){
  super();
  this.addGroup = this.addGroup.bind(this);
…
addGroup(group){
  this.setState({
    groups: [
      ...this.state.groups, group
    ]
  })
}
…
case 'CreateGroupConfirmation':
  return (
    <CreateGroupConfirmation 
      {...this.props} 
      {...route} 
      navigator={navigator}
      addGroup={this.addGroup}
    />
  );
case 'Group':
  return (
    <Group 
      {...this.props}
      {...route}
      navigator={navigator}
    />
  )
```

Now if we fill out the form with a new group, we should see it in our groups screen.

![create group example](/images/chapter-8/create-group-example-3.png)
![create group example](/images/chapter-8/create-group-example-2.png)
![create group example](/images/chapter-8/create-group-example-1.png)

Let's make a commit here.

[Commit]() - "Successfully create group"


### Viewing a Group

Now that we're able to create groups, we want to flesh out our `Group` view. Ideally, we want to show the group's background image, information on how many users it has, and a list of events. Let's edit `application/components/groups/Group.js`.

```javascript
/* application/components/Group.js */
import moment from 'moment';
import Icon from 'react-native-vector-icons/Ionicons';
import NavigationBar from 'react-native-navbar';
import React, { Component } from 'react';
import { View, ListView, ScrollView, TouchableOpacity, Text, Image, ActionSheetIOS } from 'react-native';
import { find, findIndex, isEqual } from 'underscore';

import BackButton from '../shared/BackButton';
import { API, DEV } from '../../config';
import { Headers } from '../../fixtures';
import { globals, groupsStyles } from '../../styles';

const styles = groupsStyles;

function isMember(group, currentUser){
  return findIndex(group.members, ({ userId }) => isEqual(userId, currentUser.id)) !== -1;
};

function showJoinButton(users, currentUser){
  return findIndex(users, ({ id }) => isEqual(id, currentUser.id)) === -1;
}

class EventList extends Component{
  render(){
    <View>
      {this.props.events.map((event, idx) => (
        <Text>{event.name}</Text>
      ))}
    </View>
  }
};

class JoinButton extends Component{
  render(){
    let { addUserToGroup, group, currentUser } = this.props
    let hasJoined = isMember(group, currentUser);
    return (
      <View style={[styles.joinButtonContainer, globals.mv1]}>
        <TouchableOpacity onPress={() => addUserToGroup(group, currentUser)} style={styles.joinButton}>
          <Text style={styles.joinButtonText}>{ hasJoined ? 'Joined' : 'Join'}</Text>
            <Icon
              name={hasJoined ? "ios-checkmark" : "ios-add"}
              size={30}
              color='white'
              style={styles.joinIcon}
            />
        </TouchableOpacity>
      </View>
    )
  }
}

export const GroupMembers = ({ users, members, handlePress }) => {
  return (
    <View>
      {members.map((member, idx) => {
        let user = find(users, ({ id }) => isEqual(id, member.userId));
        if ( ! user ) { return; }
        return (
          <TouchableOpacity key={idx} style={globals.flexRow} onPress={() => handlePress(user)}>
            <Image source={{uri: user.avatar}} style={globals.avatar}/>
            <View style={globals.textContainer}>
              <Text style={globals.h5}>{user.firstName} {user.lastName}</Text>
              <Text style={[styles.h4, globals.mh1]}>{member.role}</Text>
            </View>
          </TouchableOpacity>
        )
      })}
    </View>
  );
}

class Group extends Component{
  constructor(){
    super();
    this.goBack = this.goBack.bind(this);
    this.visitProfile = this.visitProfile.bind(this);
    this.state = {
      events    : [],
      ready     : false,
      users     : [],
    }
  }

  componentDidMount(){
    this._loadUsers();
  }

  _loadUsers(events){
    this.setState({ events })
    let query = {
      id: { $in: this.props.group.members.map(({ userId }) => userId ) },
      $limit: 100
    };
    fetch(`${API}/users?${JSON.stringify(query)}`)
    .then(response => response.json())
    .then(users => this.setState({ users, ready: true }))
    .catch(err => {})
    .done();
  }
  goBack(){
    this.props.navigator.replacePreviousAndPop({ name: 'Groups' });
  }
  visitProfile(user){
    this.props.navigator.push({
      name: 'Profile',
      user
    })
  }
  visitCreateEvent(group){
    this.props.navigator.push({
      name: 'CreateEvent',
      group
    })
  }
  render(){
    let { group, currentUser } = this.props;
    let showButton = showJoinButton(this.state.users, currentUser) && this.state.ready;
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          title={{title: group.name, tintColor: 'white'}}
          tintColor={Colors.brandPrimary}
          leftButton={<BackButton handlePress={this.goBack}/>}
        />
        <ScrollView style={globals.flex}>
          <Image source={{uri: group.image}} style={styles.groupTopImage}>
            <View style={styles.overlayBlur}>
              <Text style={styles.h1}>{group.name}</Text>
            </View>
            <View style={styles.bottomPanel}>
              <Text style={[globals.h4, globals.primaryText]}>
                {group.members.length} {group.members.length === 1 ? 'member' : 'members'}
              </Text>
            </View>
          </Image>
          <Text style={styles.h2}>Summary</Text>
          <Text style={[globals.h5, globals.ph2]}>{group.description}</Text>
          <Text style={styles.h2}>Technologies</Text>
          <Text style={[globals.h5, globals.ph2]}>{group.technologies.join(', ')}</Text>
          <View style={globals.lightDivider}/>
          { showButton ? <JoinButton addUserToGroup={this.props.addUserToGroup} group={group} currentUser={currentUser} /> : null }
          <Text style={styles.h2}>Events</Text>
          <View style={globals.lightDivider} />
          <Text style={styles.h2}>Members</Text>
          <View style={globals.lightDivider} />
          <GroupMembers
            members={group.members}
            users={this.state.users}
            handlePress={this.visitProfile}
          />
        </ScrollView>
      </View>
    )
  }
};

export default Group;


```
![new group example](/images/chapter-8/new-group-example-1.png)

Let's review the new code:
- In our `componentDidMount` lifecycle method, we fetch the users related to the group. We use the mongodb **$in** operator for this, fetching all the users that have an id in the groups members array. We also utilize the `$limit` option, to keep the users being fetched to 10 in number.
- We pass our fetched users to a **GroupMembers** component which renders each one at the bottom of the screen. Currently, these are pressable but lead to a blank screen. Later, we will have them direct to a user profile screen.
- We also have an events section that isn't being used currently. We will need implement functionality to create and render events before fleshing this out further.
- Also notice that if the user isn't a member, the **join** button will appear, but it throws an error when pressed. This is because we haven't defined an `addUserToGroup` method in our top-level `GroupsView` component.

Let's remember to make a commit at this point.

[Commit](https://github.com/buildreactnative/assemblies-tutorial/tree/95fedabf381a3f3cfffe180ab51c6b7d2cbd45b5) - "Render individual groups in Group component"

 
## Wrapping Up

So far in this chapter, we rendered groups that belong to a user and that are nearby. We created a form for making new groups and rendered individual groups. Next we have to add the ability to join different groups and create events for them. 
