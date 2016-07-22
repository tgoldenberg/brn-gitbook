# Chapter 12: Final Touches

Let's review what we've accomplished before and what our app can do. 

- User authentication (login, register, logout, edit profile)
- TabBar navigation 
- Ability to create groups
- Ability to schedule and RSVP to events
- Ability to message users and view messages by conversation
- Notification system for new events and messages

What else can we do to improve the quality of our app? For one, we can make logging in a little easier. Our users shouldn't have to login everytime they use the app (that might be more appropriate for a financial app, for example). 

To achieve this, we will store the users password in the `AsyncStorage` module and login when the user opens the app. If no account has been created, the app will run as usual.

```javascript

```