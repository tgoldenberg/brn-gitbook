# Chapter 18: Adding Redux to your App

Redux, according to its Github README, is a "predictable state container for JavaScript apps." In other words, it's a way to create a single source of truth for your client-side state. This can very useful for complex applications that share data across different views. 

#### Setting Things Straight

However, do not think that everything application **needs** Redux. This is not true. Redux is an excellent pattern for when state is shared across many components. It is also a reliable way to test user flows and locating bottlenecks and bugs. There are situations where you probably shouldn't use Redux, though, and definitely not to start when using React or React Native. Here are some tweets by Dan Abramov, creator of Redux.

* ["Don’t use Redux unless you *tried* local component state and were dissatisfied."](https://twitter.com/dan_abramov/status/725089243836588032)
* ["There is so much FUD about local state. “setState is bad”, “no-set-state”, “root of all evil”, “use functional components”. State is fine."](https://twitter.com/dan_abramov/status/725089775783391232)
* ["Beginners take Redux principles (“use single state tree”) and think they’re universally good advice for any React app"](https://twitter.com/dan_abramov/status/760891641657954305)

And from Ryan Florence, author of `react-router`:

* ["why do so many think React means React + Webpack + Redux + CSS Modules + Hot Loading + Server Rendering + ..."](https://twitter.com/ryanflorence/status/724934112822329344)

#### How does Redux work?

Let's examine how Redux works. If you think of your application as a tree of components, then state flows downwards from higher to lower components. But what happens when sibling components need to be aware of a change? Then a lower component has to pass a callback to the higher component which then shares the state with the sibling. This happens in **Assemblies** with our user state. We initiate the initial user data in our **index.ios.js** file and pass it to the child components. However, in **profile.js**, for example, we initiate a change in the user object -- the user updates their profile. This change then needs to be passed up to the parent component via a callback (**updateUser**), and then the change is shared with the rest of the application.

What's the way around this? Redux introduces a client data store that holds application state, and that individual components can "connect" to. That means that we don't have to pass `currentUser={user}` as **props** in all of our components. Any component that needs access to the user object can "connect" to the Redux store.

![redux](https://css-tricks.com/wp-content/uploads/2016/03/redux-article-3-02.svg)

This means that our components can theoretically become "lighter", carrying less props. It also means that sharing data across components is seamless. Finally, holding state in a single store makes logging changes and identifying bugs much easier (more on this later). 

Instead of explaining Redux in abstract terms, let's dig in right away in the codebase and start implementing, learning more as we go along. Since the user data is shared across all of our components, it makes sense to incorporate user data in our Redux store to start. 


