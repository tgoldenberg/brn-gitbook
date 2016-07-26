# Establishing a Database Schema

Let's go over what we want to do in our app and what kind of data we'll need. Already we have a `users` collection, with the following relevant fields:
```
id: String
username: String
password: String
firstName: String
lastName: String
technologies: Array of Strings
avatar: String
location: Object {
  city: Object {
    long_name: String,
    short_name: String
  }
  state: Object {
    long_name: String,
    short_name: String
  }
  formattedAddress: String,
  lat: Float,
  lng: Float
}
```
This way of notation simply expresses that location is an `Object` with a field `city` which is also an object with the fields `long_name` and `short_name`. It's important to keep track of the structure of our data as we create it, since the wrong data type can cause errors in our program.

## Defining Collections

Now what other collections will we need? We certainly need a `messages` collection, since we offer user-to-user messaging as a feature of our app. We also need a `groups` collection, since an assembly is a type of group that has members and hosts events. We need an `events` collection to highlight new events for our users. Here's a schema we can use for each one. We'll want a `conversations` collection to easily render conversation data, and a `notifications` collection for informing users of new messages and events.

```
**messages**

id: String
createdAt: Integer
text: String
senderId: String
recipientId: String

**conversations**

id: String
user1Id: String
user2Id: String
lastMessageText: String
lastMessageDate: Integer

**groups**

id: String,
createdAt: Integer
members: Array of Objects {
  userId: String,
  confirmed: Boolean,
  role: String,
  joinedAt: Integer
}
name: String
technologies: Array of Strings
description: String
color: String
image: String

**events**

id: String
groupId: String
createdAt: Integer
start: Integer
end: Integer
location: Object {
  lat: Float
  lng: Float
  city: Object
  state: Object
  formattedAddress: String
}
going: Array of Strings
capacity: Integer

**notifications**
id: String
type: String
message: String
participants: Array of Strings
createdAt: Integer

```

Now let's create these collections from our Deployd interface at `localhost:2403`. This should be a straight-forward process of creating a collection and then editing its fields in the `properties` tab.

![deployd](Screen Shot 2016-07-25 at 11.21.20 PM.png)
![deployd](Screen Shot 2016-07-25 at 11.21.27 PM.png)

## Deployd in Production

While all this is well and good for local development, what about when we want our app to go live? At that point we will need to deploy Deployd to an actual server. For full instructions on this, please read the API deployment chapter in the appendix.

## Creating Messages

Now that we have our data models in place, we can start to replace our fixture data with real data. We also have to consider what we want our `MessagesView` to contain.

Currently `MessagesView.js` displays a list of conversations with the last message sent displayed. Intuitively, if we press on one of these conversations, we should be directed to a conversation view, where we can scroll through all the messages and create new messages. We also may want to be able to direct to a Profile View for when any of the user’s avatars is pressed.

Therefore, we have to replace the static component of `MessagesView` and replace it with a new `Navigator` with the two routes `Conversations` and `Conversation`. 

Let’s move the contents of `MessagesView.js` to another file, `Conversations.js`, and fill in `MessagesView.js` with a new `Navigator` component. We'll also create a dumb `Conversation.js` file for now.

```javascript
application/components/messages/Conversation.js

import Icon from 'react-native-vector-icons/Ionicons';
import NavigationBar from 'react-native-navbar';
import React, { Component } from 'react';
import { View, Text } from 'react-native';

import { globals } from '../../styles';
import BackButton from '../shared/BackButton';

class Conversation extends Component{
  constructor(){
    super();
    this.goBack = this.goBack.bind(this);
  }
  goBack(){
    this.props.navigator.pop();
  }
  render(){
    let { user, currentUser } = this.props;
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          tintColor={Colors.brandPrimary}
          title={{ title: `${user.firstName} ${user.lastName}`, tintColor: 'white' }}
          leftButton={<BackButton handlePress={this.goBack}/>}
        />
        <View style={globals.flexCenter}>
          <Text style={globals.h2}>Conversation</Text>
        </View>
      </View>
    )
  }
};

export default Conversation;
```

```javascript
application/components/messages/MessagesView.js

import React, { Component } from 'react';
import { Navigator } from 'react-native';
import { flatten, uniq } from 'underscore';

import Conversation from './Conversation';
import Conversations from './Conversations';
import { DEV, API } from '../../config';
import { globals } from '../../styles';

class MessagesView extends Component{
  constructor(){
    super();
    this.state = {
      conversations : [],
      ready         : false,
      users         : [],
    };
  }
  componentDidMount(){
    this._loadConversations();
  }
  _loadConversations(){
    let { currentUser } = this.props;
    let query = {
      $or: [
        { user1Id: currentUser.id },
        { user2Id: currentUser.id }
      ],
      $limit: 10, $sort: { lastMessageDate: -1 }
    };
    fetch(`${API}/conversations?${JSON.stringify(query)}`)
    .then(response => response.json())
    .then(conversations => this._loadUsers(conversations))
    .catch(err => this.ready(err))
    .done();
  }
  _loadUsers(conversations){
    let userIds = uniq(flatten(conversations.map(c => [c.user1Id, c.user2Id])));
    let query = { id: { $in: userIds }};
    fetch(`${API}/users?${JSON.stringify(query)}`)
    .then(response => response.json())
    .then(users => this.setState({ conversations, users, ready: true }))
    .catch(err => this.ready(err))
    .done();
  }
  ready(err){
    this.setState({ ready: true });
  }

  render(){
    return (
      <Navigator
        style={globals.flex}
        initialRoute={{ name: 'Conversations' }}
        renderScene={(route, navigator) => {
          switch(route.name){
            case 'Conversations':
              return (
                <Conversations
                  {...this.props}
                  {...this.state}
                  navigator={navigator}
                />
            );
            case 'Conversation':
              return (
                <Conversation
                  {...this.props}
                  {...this.state}
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

export default MessagesView;
```

If done correctly, the Messages View should look exactly the same as before! Don’t worry, we have a long way to go. Let’s commit here.

[Commit]() – "Refactor MessagesView into a Navigation component and create simple Conversation and UserProfile components"

## Fetching Message Data

Now we will want to replace our `FAKE_USERS` and `FAKE_MESSAGES` data for real data. To get started, we can create a conversation in the `data` tab of the collection, at `localhost:2403/dashboard`. 

To do this, we should first create a few users, but using the `data` tab of the `users` collection. You can add different emails and names, and copy/paste the location and technologies from the already existing user.

Once we have some users, we can then do the same for our `conversations` collection. Add your user id as `user1Id` and the other user as `user2Id`. Then fill in the `lastMessageText` with a text string and the `lastMessageDate` with a date value (simply typing `new Date().valueOf()` in the browser console is enough to get this.

Once we have a conversation in the database, we can work on fetching the data from the `MessagesView`. Replace the instances of `FakeUsers`, `FakeNotifications`, etc. in `Conversations.js`.

```javascript
import React, { Component } from 'react';
import { View, Text, Image, TouchableOpacity, ListView } from 'react-native';
import moment from 'moment';
import Icon from 'react-native-vector-icons/Ionicons';
import NavigationBar from 'react-native-navbar';
import { find } from 'underscore';

import Colors from '../../styles/colors';
import { globals, messagesStyles } from '../../styles';
import { rowHasChanged } from '../../utilities';

const styles = messagesStyles;

class Conversations extends Component{
  constructor(){
    super();
    this._renderRow = this._renderRow.bind(this);
    this.dataSource = this.dataSource.bind(this);
  }

  _renderRow(conversation){
    let { currentUser } = this.props;
    let otherUserID = find([conversation.user1Id, conversation.user2Id], (id) => id !== currentUser.id);
    let user = find(this.props.users, ({ id }) => id === otherUserID);
    return (
      <TouchableOpacity style={globals.flexContainer}>
        <View style={globals.flexRow}>
          <Image style={globals.avatar} source={{uri: user.avatar}}/>
          <View style={globals.flex}>
            <View style={globals.textContainer}>
              <Text style={styles.h5}>{user.firstName} {user.lastName}</Text>
              <Text style={styles.h6}>{moment(conversation.lastMessageDate).fromNow()}</Text>
            </View>
            <Text style={[styles.h4, globals.mh1]}>{conversation.lastMessageText.substring(0, 40)}...</Text>
          </View>
          <View style={styles.arrowContainer}>
            <Icon size={30} name="ios-arrow-forward" color={Colors.bodyTextLight}/>
          </View>
        </View>
        <View style={styles.divider}/>
      </TouchableOpacity>
    )
  }
  dataSource(){
    return (
      new ListView.DataSource({ rowHasChanged: rowHasChanged }).cloneWithRows(this.props.conversations)
    );
  }
  render() {
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          title={{ title: 'Messages', tintColor: 'white' }}
          tintColor={Colors.brandPrimary}
        />
        <ListView
          enableEmptySectionHeaders={true}
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

![screen](Simulator Screen Shot Jul 26, 2016, 12.01.25 AM.png)

Now you should see some real conversation data!

Here you can see that we are first fetching the conversations that are relevant to the user. Then we collect the userID’s that are relevant and fetch the user data for those IDs. This data then gets passed on to the `Conversations` component.

Also notice that we are using queries to fetch our data. In Deployd, we can add these Mongo queries at the end of our API call, preceded by a `?`. In the first query, we are asking for all conversations where the `user1Id` or the `user2Id` is equal to the current user's `id`. In the second we are fetching all users who have an `id` that is contained in the array of `userId`s.

Let’s create another user and another conversation and see how the UI changes.

![deployd](Screen Shot 2016-07-26 at 12.05.33 AM.png)
![screen](Simulator Screen Shot Jul 26, 2016, 12.06.17 AM.png)


And now is a good time to make a commit.

[Commit 13](https://github.com/buildreactnative/assemblies-tutorial/tree/7297fbd74e3decf4ce402ac69d445d4dbe6de0d5) – "Fetch conversation data and render in Conversations component"

## Adding Routing to our Messages View

Now that we at least have real `conversation` objects, we need to add routing to an individual `conversation` component.

Let's modify the `_renderRow` method in the file `application/components/messages/Conversations.js`

```javascript
import React, { Component } from 'react';
import { View, Text, Image, TouchableOpacity, ListView } from 'react-native';
import moment from 'moment';
import Icon from 'react-native-vector-icons/Ionicons';
import NavigationBar from 'react-native-navbar';
import { find } from 'underscore';

import Colors from '../../styles/colors';
import { globals, messagesStyles } from '../../styles';
import { rowHasChanged } from '../../utilities';

const styles = messagesStyles;

class Conversations extends Component{
  constructor(){
    super();
    this.visitConversation = this.visitConversation.bind(this);
    this._renderRow = this._renderRow.bind(this);
    this.dataSource = this.dataSource.bind(this);
  }

  visitConversation(user){
    this.props.navigator.push({
      name: 'Conversation',
      user
    })
  }
  _renderRow(conversation){
    let { currentUser } = this.props;
    let otherUserID = find([conversation.user1Id, conversation.user2Id], (id) => id !== currentUser.id);
    let user = find(this.props.users, ({ id }) => id === otherUserID);
    return (
      <TouchableOpacity style={globals.flexContainer} onPress={() => this.visitConversation(user)}>
        <View style={globals.flexRow}>
          <Image style={globals.avatar} source={{uri: user.avatar}}/>
          <View style={globals.flex}>
            <View style={globals.textContainer}>
              <Text style={styles.h5}>{user.firstName} {user.lastName}</Text>
              <Text style={styles.h6}>{moment(conversation.lastMessageDate).fromNow()}</Text>
            </View>
            <Text style={[styles.h4, globals.mh1]}>{conversation.lastMessageText.substring(0, 40)}...</Text>
          </View>
          <View style={styles.arrowContainer}>
            <Icon size={30} name="ios-arrow-forward" color={Colors.bodyTextLight}/>
          </View>
        </View>
        <View style={styles.divider}/>
      </TouchableOpacity>
    )
  }
  dataSource(){
    return (
      new ListView.DataSource({ rowHasChanged: rowHasChanged }).cloneWithRows(this.props.conversations)
    );
  }
  render() {
    return (
      <View style={globals.flexContainer}>
        <NavigationBar
          title={{ title: 'Messages', tintColor: 'white' }}
          tintColor={Colors.brandPrimary}
        />
        <ListView
          enableEmptySectionHeaders={true}
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

Now when you press on a conversation row, the navigator should go to the `Conversation` screen, which is currently just a placeholder.

![screen](Simulator Screen Shot Jul 26, 2016, 12.11.48 AM.png)

You should now get directed to our `Conversation` screen when you press on a conversation. Now we need to flesh out that view. Ideally we want to have all the messages in reverse chronological order, along with an input field on the bottom to send a new message.

Let's create a few messages in Deployd to get started. Here is some data to get started (replace the userIds with the appropriate ones from your Deployd database):

```
senderId: 15f9d0d11a023b8a
recipientId: b0d4dc1d8fd13a7d
createdAt: 1467764227276
text: "How are you?"

senderId: b0d4dc1d8fd13a7d
recipientId: 15f9d0d11a023b8a
createdAt: 1467764227276
text: "Fine, and you?"

senderId: 15f9d0d11a023b8a
recipientId: c8c0cfa404ee1831
createdAt: 1467764227276
text: "So what's going on tonight?"

senderId: c8c0cfa404ee1831
recipientId: 15f9d0d11a023b8a
createdAt: 1467764227276
text: "I don't know. You tell me."

```
![](Screen Shot 2016-07-05 at 8.20.44 PM.png)

Now we can query the related messages in the `componentWillMount` phase of our `Conversation` component and then render the messages in reverse chronological order.

First let's install two new packages:
- `react-native-invertible-scroll-view`
- `react-native-keyboard-spacer`

And then let's render our component:

```javascript
application/components/messages/Conversation.js

import moment from 'moment';
import InvertibleScrollView from 'react-native-invertible-scroll-view';
import KeyboardSpacer from 'react-native-keyboard-spacer';
import NavigationBar from 'react-native-navbar';
import React, { Component } from 'react';
import { View, Text, Image, TouchableOpacity, TextInput } from 'react-native';

import Colors from '../../styles/colors';
import { Headers } from '../../fixtures';
import BackButton from '../shared/BackButton';
import { DEV, API } from '../../config';
import { globals, messagesStyles } from '../../styles';

const styles = messagesStyles;

const Message = ({ user, message }) => {
  return (
    <Text>{message}</Text>
  )
};

class Conversation extends Component{
  constructor(){
    super();
    this.goBack = this.goBack.bind(this);
    this.createMessage = this.createMessage.bind(this);
    this.state = {
      messages  : [],
      message   : '',
    }
  }
  componentWillMount(){
    this._loadMessages();
  }
  _loadMessages(){
    let { user, currentUser } = this.props;
    console.log('USER IDS', user.id, currentUser.id);
    let query = {
      $or: [
        { senderId    : user.id, recipientId  : currentUser.id },
        { recipientId : user.id, senderId     : currentUser.id }
      ],
      $sort: { createdAt: -1 },
      $limit: 10
    };
    fetch(`${API}/messages?${JSON.stringify(query)}`)
    .then(response => response.json())
    .then(messages => this.setState({ messages }))
    .catch(err => console.log('ERR:', err))
    .done();
  }

  createMessage(){
    /* TODO: create a message */
  }
  goBack(){
    this.props.navigator.pop();
  }
  render(){
    let { user, currentUser } = this.props;
    return(
      <View style={globals.flexContainer}>
        <InvertibleScrollView inverted={true}>
          {this.state.messages.map((msg, idx) => (
            <Message
              key={idx}
              message={msg}
              user={msg.senderId === currentUser.id ? currentUser : user}
            />
          ))}
        </InvertibleScrollView>
        <View style={styles.navContainer}>
          <NavigationBar
            tintColor={Colors.brandPrimary}
            title={{ title: `${user.firstName} ${user.lastName}`, tintColor: 'white' }}
            leftButton={<BackButton handlePress={this.goBack}/>}
          />
        </View>
        <View style={styles.inputBox}>
          <TextInput
            multiline={true}
            value={this.state.message}
            placeholder='Say something...'
            placeholderTextColor={Colors.bodyTextLight}
            onChangeText={(msg) => this.setState({ message })}
            style={styles.input}
          />
          <TouchableOpacity
            style={ this.state.message ? styles.buttonActive : styles.buttonInactive }
            underlayColor='#D97573'
            onPress={this.createMessage}>
            <Text style={ this.state.message ? styles.submitButtonText : styles.inactiveButtonText }>Send</Text>
          </TouchableOpacity>
        </View>
        <KeyboardSpacer topSpacing={-50} />
      </View>
    )
  }
};

export default Conversation;

```

Here's an overview of what's going on:
- We use the `InvertibleScrollView` component to keep the screen at the bottom of our list of messages. This way when we add a new message, it is easy to scroll to the bottom of the screen.
- The `KeyboardSpacer` component allows us to keep the text input just about the keyboard when a user is typing. To simulate typing on the Simulator, just type cmd + sft + k to toggle keyboard mode.
- We fetch our messages in the `componentDidMount` method. Once the results are fetched, they should render in the `<Messages/>` component.

We still need to create a message, as well as flesh out our `Messages` component. Let's do that.

```javascript
import React, { Component } from 'react';
import { View, Text, Image, TouchableOpacity, TextInput } from 'react-native';
import moment from 'moment';
import InvertibleScrollView from 'react-native-invertible-scroll-view';
import KeyboardSpacer from 'react-native-keyboard-spacer';
import NavigationBar from 'react-native-navbar';

import Colors from '../../styles/colors';
import { Headers, DefaultAvatar } from '../../fixtures';
import BackButton from '../shared/BackButton';
import { DEV, API } from '../../config';
import { globals, messagesStyles } from '../../styles';

const styles = messagesStyles;

const Message = ({ user, message }) => {
  return (
    <View style={[globals.centeredRow, globals.pt1]}>
      <View>
        <Image style={globals.avatar} source={{uri: user.avatar? user.avatar : DefaultAvatar }} />
      </View>
      <View style={[styles.flexCentered, globals.pv1]}>
        <View style={globals.flexRow}>
          <Text style={styles.h5}>{`${user.firstName} ${user.lastName}`}</Text>
          <Text style={styles.h6}>{moment(new Date(message.createdAt)).fromNow()}</Text>
        </View>
        <View style={globals.flexContainer}>
          <Text style={styles.messageText}>{message.text}</Text>
        </View>
      </View>
    </View>
  )
};


class Conversation extends Component{
  constructor(){
    super();
    this.goBack = this.goBack.bind(this);
    this.createMessage = this.createMessage.bind(this);
    this.state = {
      messages  : [],
      message   : '',
    }
  }
  componentWillMount(){
    this._loadMessages();
  }
  _loadMessages(){
    let { user, currentUser } = this.props;
    console.log('USER IDS', user.id, currentUser.id);
    let query = {
      $or: [
        { senderId    : user.id, recipientId  : currentUser.id },
        { recipientId : user.id, senderId     : currentUser.id }
      ],
      $sort: { createdAt: -1 },
      $limit: 10
    };
    fetch(`${API}/messages?${JSON.stringify(query)}`)
    .then(response => response.json())
    .then(messages => this.setState({ messages }))
    .catch(err => console.log('ERR:', err))
    .done();
  }

  createMessage(){
    let { currentUser, user } = this.props;
    fetch(`${API}/messages`, {
      method: 'POST',
      headers: Headers,
      body: JSON.stringify({
        senderId      : currentUser.id,
        recipientId   : user.id,
        text          : this.state.message,
        createdAt     : new Date().valueOf(),
      })
    })
    .then(response => response.json())
    .then(data => this.setState({ message: '', messages: [ data, ...this.state.messages ]}) )
    .catch(err => {})
    .done();
  }
  goBack(){
    this.props.navigator.pop();
  }
  render(){
    let { user, currentUser } = this.props;
    return(
      <View style={globals.flexContainer}>
        <InvertibleScrollView inverted={true}>
          {this.state.messages.map((msg, idx) => (
            <Message
              key={idx}
              message={msg}
              user={msg.senderId === currentUser.id ? currentUser : user}
            />
          ))}
        </InvertibleScrollView>
        <View style={styles.navContainer}>
          <NavigationBar
            tintColor={Colors.brandPrimary}
            title={{ title: `${user.firstName} ${user.lastName}`, tintColor: 'white' }}
            leftButton={<BackButton handlePress={this.goBack}/>}
          />
        </View>
        <View style={styles.inputBox}>
          <TextInput
            multiline={true}
            value={this.state.message}
            placeholder='Say something...'
            placeholderTextColor={Colors.bodyTextLight}
            onChangeText={(msg) => this.setState({ message })}
            style={styles.input}
          />
          <TouchableOpacity
            style={ this.state.message ? styles.buttonActive : styles.buttonInactive }
            underlayColor='#D97573'
            onPress={this.createMessage}>
            <Text style={ this.state.message ? styles.submitButtonText : styles.inactiveButtonText }>Send</Text>
          </TouchableOpacity>
        </View>
        <KeyboardSpacer topSpacing={-50} />
      </View>
    )
  }
};

export default Conversation;


```

![screen](Simulator Screen Shot Jul 26, 2016, 12.29.06 AM.png)
![screen](Simulator Screen Shot Jul 26, 2016, 12.29.01 AM.png)



## Callback to Update Conversations (Optional)

One cool thing about Deployd is that we can set callback hooks from within our REST-ful actions. For example, once we send a new message, we should update the relevant `conversation` with the latest message and timestamp. Using the `Events` tab in our Deployd dashboard, this is done easily. Under `POST`, add the following lines.
```javascript
console.log('MESSAGE CREATED', this);
var text = this.text;
var senderId = this.senderId;
var recipientId = this.recipientId;
var createdAt = this.createdAt;

dpd.conversations.get({
    $or: [
        {user1Id: senderId, user2Id: recipientId},
        {user2Id: senderId, user1Id: recipientId}
    ]
})
.then(function(data){
    console.log('DATA', data);
    if (data.length) {
        dpd.conversations.put(data.id, {
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
```

Notice how we can use this hooks to log statements and to update other collections.

## Conclusion

In this chapter, we built out a message view for our users. We created a user-friendly way to share messages between two users. While this is a feature of numerous mobile applications, our work is not yet done. We still have a lot of features to build out - Calendar View, more Profile customizations, creating and editing Groups and Events, etc.

If you're interested in improving your messaging view even further, you can explore options like creating a real-time connection for live chat. This can be done in a number of ways - through a 3rd-party service such as Firebase or Pusher, through establishing a websocket connection with our Deployd API, or by using a real-time framework such as MeteorJS. In any case, congratulations and continue the good work!
