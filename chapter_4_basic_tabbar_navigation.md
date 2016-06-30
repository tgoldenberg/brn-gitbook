## Putting Together the Pieces

Last chapter we left off with the start of our project -- a working `Navigator` and a styled landing page. Next we're going to implement `TabBar` navigation inside of our `Dashboard` component.

While we'll be using the `TabBarIOS` component that comes with React Native, you can also components that are compatible with Android, such as [this package](https://github.com/alinz/react-native-tabbar). While it might not be possible to have 100% of your code be the same across iOS and Android, we can come pretty close, especially as contributors bridge the gaps. For now, we'll be focusing on our iOS app. See the Appendix for more information about sharing code across platforms.

Let's start by making three tabs - Dashboard, Messages, and Profile. We'll then fill out the screens with fake data. First replace the contents of `application/components/Dashboard.js` with the code below. Notice that we use the `react-native-vector-icons` package to customize our tab bar.

```javascript
import React, { Component } from 'react';

import {
  Dimensions,
  StyleSheet,
  TabBarIOS,
  Text,
  TouchableOpacity,
  View,
} from 'react-native';

import Icon, {TabBarItemIOS} from 'react-native-vector-icons/Ionicons';
import ProfileView from './profile/ProfileView';
import MessagesView from './messages/MessagesView';
import ActivityView from './activity/ActivityView';

export default class Dashboard extends Component{
  constructor(props){
    super(props);
    this.state = {
      selectedTab: 'Activity',
    };
  }
  render() {
    let { selectedTab } = this.state;
    return (
      <TabBarIOS style={styles.outerContainer}>
        <TabBarItemIOS
          title='Activity'
          selected={ selectedTab == 'Activity' }
          iconName='ios-pulse'
          onPress={() => this.setState({ selectedTab: 'Activity' })}
        >
          <ActivityView />
        </TabBarItemIOS>
        <TabBarItemIOS
          title='Messages'
          selected={ selectedTab == 'Messages' }
          iconName='ios-chatboxes'
          onPress={() => this.setState({ selectedTab: 'Messages' })}
        >
          <MessagesView />
        </TabBarItemIOS>
        <TabBarItemIOS
          title='Profile'
          selected={ selectedTab == 'Profile' }
          iconName='ios-person'
          onPress={() => this.setState({ selectedTab: 'Profile' })}
        >
          <ProfileView />
        </TabBarItemIOS>
      </TabBarIOS>
    );
  }
};

let styles = StyleSheet.create({
  outerContainer: {
    backgroundColor: 'white'
  },
  ...
...
```

Basically, we're defining each tab with basic information like it's title, the icon it uses, the component it should render upon selection, and passing in the name of the selected tab to `state`, so we can respond to it if necessary.

Now we have to create the tab components `ActivityView`, `MessagesView`, and `ProfileView`. Here's the code for `ActivityView`. Simply change the name of the component, the NavigationBar title, and text for the other two components.

```javascript
import React, { Component } from 'react';

import {
  StyleSheet,
  Text,
  TouchableOpacity,
  View,
} from 'react-native';

import NavigationBar from 'react-native-navbar';
import Colors from '../../styles/colors';

export default class ActivityView extends Component{
  render() {
    return (
      <View style={{ flex: 1 }}>
        <NavigationBar
          title={{ title: 'Activity', tintColor: 'white' }}
          tintColor={Colors.brandPrimary}
        />
        <View style={styles.container}>
          <Text style={styles.h1}>This is the ActivityView</Text>
        </View>
      </View>
    );
  }
};

let styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  h1: {
    fontSize: 22,
    fontWeight: 'bold',
    padding: 20,
  },
});
```

Here's what we have so far. Let's make a commit at this point.

![Empty TabBar Views](/images/chapter-4-basic-tabbar-navigation/empty-tabbar-views.png "Empty TabBar Views")

***

[![GitHub logo](/images/github-logo.png "GitHub logo") Commit 5](https://github.com/buildreactnative/assemblies-tutorial/commit/f5bc72f5f44c9d0146602d4c75a7353d07dd9039) - Commit 5]() "Create empty TabBar views"

***

## Building the Profile View

Let's build out the tab screens. We'll be using fixtures for this. Add [this gist](https://gist.github.com/tgoldenberg/ef3dc76063ca68ecab09840f6b3eb5ab) as a file in `application/fixtures/fixtures.js`. Let's use this fixtures file to build out our `ProfileView` component.

```javascript
...
import Icon from 'react-native-vector-icons/Ionicons';
import { currentUser } from '../../fixtures/fixtures';

import React, { Component } from 'react';
import {
  View,
  Text,
  ScrollView,
  Image,
  StyleSheet,
  TouchableOpacity,
} from 'react-native';

export default class ProfileView extends Component{
  render() {
    console.log('CURRENT USER', currentUser);
    return (
      <View style={{ flex: 1 }}>
        <NavigationBar
          title={{ title: 'Profile', tintColor: 'white' }}
          tintColor={Colors.brandPrimary}
        />
        <ScrollView style={styles.container}>
          <View style={styles.profileHolder}>
            <TouchableOpacity style={styles.avatarHolder}>
              <Image source={{uri: currentUser.avatarUrl}} style={styles.avatar}/>
            </TouchableOpacity>
            <View style={styles.userInfoHolder}>
              <Text style={styles.name}>{currentUser.firstName} {currentUser.lastName}</Text>
              <Text style={styles.location}>{currentUser.location.city.long_name}, {currentUser.location.state.short_name}</Text>
            </View>
          </View>
          <TouchableOpacity style={styles.formField}>
            <Text style={styles.formName}>My Technologies</Text>
            <View>
              <Icon name='ios-arrow-forward' size={30} color='#ccc' />
            </View>
          </TouchableOpacity>
          <TouchableOpacity style={styles.formField}>
            <Text style={styles.formName}>Settings</Text>
            <View>
              <Icon name='ios-arrow-forward' size={30} color='#ccc' />
            </View>
          </TouchableOpacity>
          <TouchableOpacity style={styles.logoutButton}>
            <Text style={styles.logoutText}>Logout</Text>
          </TouchableOpacity>
        </ScrollView>
      </View>
    );
  }
};

let styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: Colors.inactive,
  },
  avatar: {
    width: 100,
    height: 100,
    borderRadius: 50,
    borderWidth: 1,
    borderColor: '#777',
  },
  profileHolder: {
    backgroundColor: 'white',
    flexDirection: 'row',
    justifyContent: 'center',
    alignItems: 'center',
    marginBottom: 10,
    paddingHorizontal: 30,
  },
  formField: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingHorizontal: 30,
    paddingVertical: 12,
    backgroundColor: 'white',
    marginVertical: 10,
  },
  formName: {
    fontWeight: '300',
    fontSize: 18,
  },
  name: {
    fontSize: 20,
    textAlign: 'center',
    fontWeight: '500',
  },
  location: {
    fontSize: 18,
    textAlign: 'center',
    fontWeight: '300',
  },
  userInfoHolder: {
    flex: 1.2,
    justifyContent: 'center',
    alignItems: 'stretch',
    paddingHorizontal: 12,
  },
  avatarHolder: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    paddingVertical: 10,
  },
  logoutButton: {
    position: 'absolute',
    left: 30,
  },
  logoutText: {
    paddingTop: 15,
    fontSize: 18,
    fontWeight: '300',
    color: 'red',
  },
});
```

Here we use the `ScrollView` component for the first time. ScrollView gives us greater layout flexibility then relying on a simple `View` component. Even though the components may fit on a certain view, using `ScrollView` ensures that all components will be visible on all phone sizes. As for the functionality of the `TouchableOpacity` components, we will add that in later.

Note that we also added an `inactive` variable to our `colors.js` file:

```javascript
export default Colors = {
  brandPrimary: '#3A7BD2',
  inactive: '#EBEEF5'
};
```

We are also referencing a `fixtures.js` file in a newly-created `application/fixtures` folder. You can download that file from GitHub by following the commit link below and add it to your project directory at the appropriate path.

Now let's make a commit.

![{Profile TabBar View}](/images/chapter-4-basic-tabbar-navigation/profile-tabbar-view.png "Profile TabBar View")

***
[![GitHub logo](/images/github-logo.png "GitHub logo") Commit 6](https://github.com/buildreactnative/assemblies-tutorial/commit/f5bc72f5f44c9d0146602d4c75a7353d07dd9039) - Commit 6]() "Add fixtures file and style profile view"
***

## Building the Messages View

Next, let's fill in our messages view. We'll be using our fixtures file with messages for now. One thing you'll notice is that these messages include all the relevant data for rendering them, including the author name and avatar url. Later, when implementing our back-end API, we will separate some of these data points, since a user should be able to change their name or profile photo. Therefore, it's better to store the `authorId` in the message long-term and refer to the `user` object.
For convenience, we'll be storing all the data in the messages object for now. We will be using the `ListView` in this component. The first thing we will need to do is convert our array of messages into an array of unique conversations. We then pass this conversation array (with the first message of each conversation) to our `ListView` as its `DataSource`. Let's look at the constructor for `MessagesView`

```javascript
...
import {
  StyleSheet,
  ListView,
  View,
  Image,
  Text,
  TouchableOpacity
} from 'react-native';
// import messages from fixture file
import { messages } from '../../fixtures/fixtures';

export default class MessagesView extends Component{
  constructor(props){
    super(props);
    let conversations = {};
    // store each message under a conversation key
    messages.forEach(msg => {
      let key = msg.participants.sort().join('-');
      if (conversations[key]) { conversations[key].push(msg); }
      else { conversations[key] = [msg]; }
    });
    let dataBlob = Object.keys(conversations).map(key => conversations[key]);
    // take the first message from each conversation
    this.state = {
      dataSource: new ListView.DataSource({
        rowHasChanged: (r1, r2) => r1 != r2
      })
      .cloneWithRows(dataBlob)
    };
  }
...
}

```

Now that we have our data, here's what the rest of the component looks like. By the way, we're using fake messages from [www.hipsteripsum.co](www.hipsteripsum.co). I find it to be more fun to work with than the traditional Latin lorem ipsum.

```javascript
  _renderRow(rowData){
    console.log('ROW DATA', rowData);
    return (
      <Text>{rowData[0].senderName}: {rowData[0].text}</Text>
    );
  }
  render() {
    return (
      <View style={{ flex: 1 }}>
        <NavigationBar
          title={{ title: 'Messages', tintColor: 'white' }}
          tintColor={Colors.brandPrimary}
        />
        <ListView
          dataSource={this.state.dataSource}
          contentInset={{ bottom: 49 }}
          automaticallyAdjustContentInsets={false}
          ref='messagesList'
          renderRow={this._renderRow.bind(this)}
        />
      </View>
    );
  }
};

let styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  h1: {
    fontSize: 22,
    fontWeight: 'bold',
    padding: 20,
  },
});
```

![{Messages Fixtures}](/images/chapter-4-basic-tabbar-navigation/messages-fixtures.png "Messages Fixtures")

Once we've confirmed that the data is being processed into conversations (both through the Chrome console React Native opens for debugging and our Simulator screen), we can refactor the rows into `Conversation` components. Replace the `Text` component in `_renderRow` with ```<Conversation conversation={rowData} />```. We'll also be using `moment` here for time/date formatting. Save the new `Conversation` component in your `application/components/messages` directory and be sure to import it at the top of your `MessagesView` component:

```javascript

import Colors from '../../styles/colors';
import moment from 'moment';
import Icon from 'react-native-vector-icons/Ionicons';
import { currentUser, FAKE_USERS } from '../../fixtures/fixtures';

import React, { Component } from 'react';
import {
  Component,
  StyleSheet,
  View,
  Text,
  Image,
  TouchableOpacity,
  Dimensions,
} from 'react-native';

let { width: deviceWidth, height: deviceHeight } = Dimensions.get('window');

export default class Conversation extends Component{
  render(){
    let { conversation } = this.props;
    let msg = conversation[0].text;
    let date = new Date(conversation[0].createdAt);
    let otherUser = conversation[0].participants.filter(p => p != currentUser.id)[0];
    let otherUserIdx = FAKE_USERS.map(u => u.id).indexOf(otherUser);
    let user = FAKE_USERS[otherUserIdx]
    return (
      <TouchableOpacity style={{ flex: 1 }}>
        <View style={styles.container}>
          <Image style={styles.profile} source={{uri: user.avatarUrl}}/>
          <View style={styles.fromContainer}>
            <View style={styles.container}>
              <Text style={styles.fromText}>{user.firstName} {user.lastName}</Text>
              <Text style={styles.sentText}>{moment(date).fromNow()}</Text>
            </View>
            <Text style={styles.messageText}>{msg.substring(0, 40)}...</Text>
          </View>
          <View style={styles.iconHolder}>
            <Icon size={30} name="ios-arrow-forward" color={Colors.bodyTextLight}/>
          </View>
        </View>
        <View style={styles.center}>
          <View style={styles.border}/>
        </View>
      </TouchableOpacity>
    );
  }
};

let styles = StyleSheet.create({
  container:{
    flex: 1,
    flexDirection: 'row',
    alignItems: 'center',
  },
  profile:{
    width: 50,
    height: 50,
    borderRadius: 25,
    marginHorizontal: 10,
    marginVertical: 10,
  },
  fromContainer:{
    justifyContent: 'center',
    marginLeft: 10,
    flex: 1,
  },
  fromText:{
    fontSize: 12,
    fontWeight: '700'
  },
  sentText:{
    fontSize: 12,
    fontWeight: '300',
    color: Colors.bodyTextGray,
    marginLeft: 10,
    fontWeight: '300',
    marginLeft: 10
  },
  iconHolder: {
    flex: 0.5,
    alignItems: 'flex-end',
    paddingRight: 25,
  },
  border: {
    height: 0,
    borderBottomWidth: 1,
    width: deviceWidth * 0.95,
    borderBottomColor: Colors.inactive,
  },
  messageText:{
    fontSize: 16,
    color: '#9B9B9B',
    fontStyle: 'italic',
    fontWeight: '300',
  },
  center: {
    alignItems: 'center',
  },
});
```

![{Styled Messages View}](/images/chapter-4-basic-tabbar-navigation/styled-messages-view.png "Styled Messages View")
***
[![GitHub logo](/images/github-logo.png "GitHub logo") Commit 7](https://github.com/buildreactnative/assemblies-tutorial/commit/3fe5fe1d185d1747b791b21c13b348ffbd9dc36a) - "Commit 7 - render Messages view with fixture data"

***

## 4.3 Styling the Activity View

So far we have a tab bar with two views filled in - `ProfileView` and `MessagesView`. Now let's fill in the other view, our `ActivityView`:

```javascript

import NavigationBar from 'react-native-navbar';
import Colors from '../../styles/colors';
import Icon from 'react-native-vector-icons/Ionicons';
import moment from 'moment';
import { notifications, upcomingEvent } from '../../fixtures/fixtures';

import React, { Component } from 'react';
import {
  ScrollView,
  View,
  Text,
  StyleSheet,
  InteractionManager,
  TouchableOpacity,
  Image,
  Dimensions,
  MapView,
} from 'react-native';

let { width: deviceWidth, height: deviceHeight } = Dimensions.get('window');

export default class ActivityView extends Component{
  constructor(){
    super();
    this.state = {
      ready: false
    }
  }
  componentDidMount(){
    InteractionManager.runAfterInteractions(() => {
      this.setState({ ready: true })
    })
  }

  _renderNotification(notification){
    return (
      <View style={styles.notificationsContainer}>
        <View style={{ flex: 1 }}>
          <View style={styles.row}>
            <TouchableOpacity style={styles.seenCircle}/>
            <TouchableOpacity style={styles.subjectTextContainer}>
              <Text style={styles.subjectText}>new {notification.type}</Text>
            </TouchableOpacity>
            <Text>{moment(new Date(new Date(notification.time))).fromNow()}</Text>
          </View>
          <View style={styles.messageContainer}>
            <Text style={styles.messageText}>{notification.message}</Text>
          </View>
        </View>
        <TouchableOpacity style={styles.timeContainer}>
          <Icon name='ios-arrow-forward' color='#777' size={25} />
        </TouchableOpacity>
      </View>
    );
  }
  _renderMapView(){
    const mapRegion = {
      latitude: upcomingEvent.location.lat,
      longitude: upcomingEvent.location.lng,
      latitudeDelta: 0.01,
      longitudeDelta: 0.01,
    };
    return (
      <MapView
        style={styles.map}
        region={mapRegion}
        annotations={[{latitude: mapRegion.latitude, longitude: mapRegion.longitude}]}
      />
    )
  }
  _renderMapFiller(){
    return (
      <View style={styles.map} />
    )
  }
  _renderScrollView(){
    let { ready } = this.state;
    return (
      <ScrollView
        contentContainerStyle={styles.scrollView}
        automaticallyAdjustContentInsets={false}
      >
        <View>
          <View style={styles.nextAssemblyContainer}>
            <Text style={styles.bodyText}>Next Assembly: </Text>
            <TouchableOpacity>
              <Text style={styles.eventName}>{upcomingEvent.name}</Text>
            </TouchableOpacity>
          </View>
          <Text style={styles.dateText}>{moment(new Date(upcomingEvent.start)).format('dddd MMM Do, h:mm a')}</Text>
        </View>
        {ready ? this._renderMapView() : this._renderMapFiller() }
        <View>
          <Text style={styles.bodyText}>Notifications</Text>
          <View style={styles.break}/>
          <View style={styles.notificationsHolder}>
            {notifications.map((n, idx) => {
              return (
                <View key={idx} style={{ flex: 1 }}>
                  {this._renderNotification(n)}
                </View>
              );
            })}
            <View style={styles.emptySpace} />
          </View>
        </View>
      </ScrollView>
    );
  }
  render() {
    return (
      <View style={{ flex: 1 }}>
        <View style={styles.topTab}>
          <TouchableOpacity style={[
            styles.leftSelectTab,
            styles.selectTab,
            styles.leftActiveTab
          ]}>
            <Text style={styles.activeTabText}>Nearby</Text>
          </TouchableOpacity>
          <TouchableOpacity style={[
            styles.rightSelectTab,
            styles.selectTab,
            styles.rightInactiveTab
          ]}>
            <Text style={styles.inactiveTabText}>Notifications</Text>
          </TouchableOpacity>
        </View>
        {this._renderScrollView()}
      </View>
    );
  }
};

let styles = StyleSheet.create({
  infoIcon: {
    marginHorizontal: 10,
  },
  scrollView: {
    flex: 1,
    paddingBottom: 80,
  },
  topTab: {
    flexDirection: 'row',
    justifyContent: 'center',
    alignItems: 'center',
    height: 80,
    paddingTop: 25,
    paddingBottom: 10,
    backgroundColor: Colors.brandPrimary,
  },
  nextAssemblyContainer: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  eventName: {
    color: Colors.brandPrimary,
    fontWeight: '500',
  },
  selectTab:{
    flex: 1,
    paddingVertical: 8,
    paddingHorizontal: 10,
    borderWidth: 1,
    borderColor: 'white',
  },
  leftSelectTab: {
    borderRadius: 4,
    borderTopRightRadius: 0,
    borderBottomRightRadius: 0,
    marginLeft: 5,
    borderRightWidth: 0,
  },
  rightSelectTab: {
    borderRadius: 4,
    borderTopLeftRadius: 0,
    borderBottomLeftRadius: 0,
    borderLeftWidth: 0,
    marginRight: 5,
  },
  leftActiveTab: {
    backgroundColor: 'white',
  },
  leftInactiveTab: {
    backgroundColor: Colors.brandPrimary,
  },
  rightActiveTab: {
    backgroundColor: 'white',
  },
  rightInactiveTab: {
    backgroundColor: Colors.brandPrimary,
  },
  activeTabText: {
    textAlign: 'center',
    color: Colors.brandPrimary,
  },
  inactiveTabText: {
    textAlign: 'center',
    color: 'white',
  },
  map: {
    backgroundColor: Colors.inactive,
		height: (deviceHeight / 3),
		width: deviceWidth
	},
  bodyText: {
    color: Colors.bodyText,
		fontSize: 16,
    fontWeight: '400',
		paddingHorizontal: 15,
    paddingVertical: 10,
  },
  nextEvent: {
    color: Colors.bodyTextLight,
    fontSize: 14,
    fontWeight: '300',
    fontStyle: 'italic',
  },
  dateText: {
    fontSize: 14,
    paddingBottom: 4,
    fontWeight: '300',
    fontStyle: 'italic',
    paddingHorizontal: 15,
    color: Colors.bodyText,
  },
  notificationsHolder:{
    flex: 1,
  },
  break: {
    borderWidth: 1,
    borderColor: '#ddd',
    marginHorizontal: 10,
  },
  notificationsContainer: {
    flex: 1,
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
  },
  row: {
    flexDirection: 'row',
    alignItems: 'center',
    marginVertical: 8,
  },
  seenCircle: {
    backgroundColor: Colors.brandPrimary,
    borderRadius: 7.5,
    width: 15,
    height: 15,
    marginHorizontal: 10,
  },
  emptySeen: {
    height: 15,
    width: 15,
    backgroundColor: 'white',
    marginHorizontal: 10,
  },
  subjectTextContainer: {
    marginRight: 5,
  },
  subjectText: {
    fontSize: 18,
    fontWeight: '500',
  },
  timeContainer: {
    alignItems: 'center',
    justifyContent: 'center',
    flexDirection: 'row',
    padding: 20,
  },
  timeText: {
    fontSize: 12,
    fontWeight: '300',
    paddingHorizontal: 4,
  },
  timeLink: {
    paddingHorizontal: 10,
  },
  messageContainer: {
    flex: 1,
  },
  messageText: {
    color: 'black',
    marginLeft: 35,
    fontSize: 14,
    fontStyle: 'italic',
    fontWeight: '300',
  },
});
```

![{Activity View}](/images/chapter-4-basic-tabbar-navigation/activity-view.png "Activity View")
***
![GitHub log](/images/github-logo.png "GitHub logo")(https://github.com/buildreactnative/assemblies-tutorial/tree/ch-4.3) - "Commit 8 - render Messages view with fixture data"
***

Notice we use the `InteractionManager` module. This prevents animation frames from dropping on navigator transitions, since the `MapView` can take up memory to render the map. Therefore, we ask the `InteractionManager` to wait until the navigator transtion is over before setting `ready` to `true`, which then renders our `MapView`.

## Summing Up

So far, we explored several of the UI components of React Native, navigation, and styling. We used fixture data to make this process as fast as possible. In the coming chapters we will start to flesh out the app with an external database and REST-ful read & write operations. We will also explore how to share components among different tabs and make our code more modular. The first step to this is user authentication, which we'll cover in the next chapter.


