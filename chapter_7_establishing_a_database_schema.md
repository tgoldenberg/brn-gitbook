# Chapter 7: Establishing a Database Schema

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

## 7.1 Defining Collections

Now what other collections will we need? We certainly need a `messages` collection, since we offer user-to-user messaging as a feature of our app. We also need a `groups` collection, since an assembly is a type of group that has members and hosts events. We need an `events` collection to highlight new events for our users. Here's a schema we can use for each one. Finally, we will want a `comments` collection for users to comment on individual events.

```
**messages**

id: String
createdAt: Integer
participants: Array of Strings
text: String
senderName: String
senderAvatar: String
senderId: String
recipientId: String

**groups**

id: String,
createdAt: Integer
memberIds: Array of Strings
members: Array of Objects {
  userId: String,
  confirmed: Boolean,
  role: String
}
name: String
technologies: Array of Strings
description: String
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
goingIds: Array of Strings
going: Array of Objects {
  userId: String
}
notGoingIds: Array of String
capacity: Integer

**comments**
id: String
eventId: String
createdAt: Integer
text: String
likes: Object of userId Strings

```

Now let's create these collections from our Deployd interface at `localhost:2403`. This should be a straight-forward process of creating a collection and then editing its fields in the `properties` tab.


