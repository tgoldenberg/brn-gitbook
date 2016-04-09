# Chapter 4: MVP 
## 4.0 Putting Together the pieces

Last chapter we left with the start of our project -- a working `Navigator` and a designed landing page. Next we're going to implement a TabBar navigation inside of our `Dashboard` component.

Let's start with 3 tabs - Dashboard, Messages, and Profile. We'll them fill out the screens with fake data. First replace the contents of `application/components/Dashboard.js`. Notice that we use the `react-native-vector-icons` package to customize our tab bar.

```
import Icon, {TabBarItem} from 'react-native-vector-icons/Ionicons';
import ProfileView from './profile/ProfileView';
import MessagesView from './messages/MessagesView';
import ActivityView from './activity/ActivityView';

import React, {
  View,
  Text,
  StyleSheet,
  Component,
  TouchableOpacity,
  TabBarIOS,
  Dimensions,
} from 'react-native';

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
      <TabBarIOS>
        <TabBarItem 
          title='Activity'
          selected={ selectedTab == 'Activity' }
          iconName='clipboard'
          onPress={() => this.setState({ selectedTab: 'Activity' })}
        >
          <ActivityView />
        </TabBarItem>
        <TabBarItem
          title='Messages'
          selected={ selectedTab == 'Messages' }
          iconName='android-chat'
          onPress={() => this.setState({ selectedTab: 'Messages' })}
        >
          <MessagesView />
        </TabBarItem>
        <TabBarItem
          title='Profile'
          selected={ selectedTab == 'Profile' }
          iconName='gear-b'
          onPress={() => this.setState({ selectedTab: 'Profile' })}
        >
          <ProfileView />
        </TabBarItem>
      </TabBarIOS>
    );
  }
};

...

```

Now we have to create the tab components `ActivityView`, `MessagesView`, and `ProfileView`. Here's the code for `ActivityView`. Simply change the name of the component for the other 2 components.

```
import NavigationBar from 'react-native-navbar';
import Colors from '../../styles/colors';

import React, {
  View,
  Text,
  Component,
  StyleSheet,
  TouchableOpacity,
} from 'react-native';

export default class ActivityView extends Component{
  render() {
    return (
      <View style={{ flex: 1 }}>
        <NavigationBar
          title={{ title: 'Profile', tintColor: 'white' }}
          tintColor={Colors.brandPrimary}
        />
        <View style={styles.container}>
          <Text style={styles.h1}>This is ActivityView</Text>
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

<img src="phone-09.png" style="height: 300px;" />

*** 

<img src="github-logo.png" style="width: 40px;" /> [Commit 4]() "Implement basic tab bar navigation"

*** 

