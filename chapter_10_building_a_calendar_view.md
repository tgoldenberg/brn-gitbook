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
seen: Boolean
``` 

After each new message and event is created, we should send out a notification to the message recipient and the group members. Let's add this logic in Deployd with a callback after an event is created and after a message is created. Here's what that looks like.



