# Chapter 10: Building a Calendar View

## 10.1 Building on the shoulders of giants

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
    console.log('SENDER', user.firstName);
    dpd.notifications.post({
        type: 'Message',
        participants: [{userId: recipientId, seen: false}],
        message: 'New message from ' + user.firstName,
        createdAt: new Date().valueOf()
    })
})
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
        })
    })
});

```
![deployd event](Screen Shot 2016-07-18 at 8.52.19 PM.png)
![new event](Screen Shot 2016-07-18 at 8.53.43 PM.png)

![notification](Screen Shot 2016-07-18 at 8.51.20 PM.png)

Now if we create another event, we will see a new notification has been created. Now it should be straightforward to render these in our `ActivityView`. We also want to load the next event that a user is signed up for, or if there is none, the next event nearby.

Let's edit `ActivityView.js`.

```javascript

```