# Chapter 17: App Design for Non-Designers


React Native is an incredible technology. The framework makes it possible for a single web developer to create an entire app herself. To be sure, this is progress.

But with great power comes great responsibility. Now that you *can* build a whole app yourself, how do you make sure it's actually usable? It's completely possible to be a great developer and not have the first idea about how to make your app intuitive and attractive.

We've got you covered. You don't need to hire a designer or spend years at art school to build an app your users will love. In this chapter we'll cover the basics of good design, as well as some resources to get you started. 

## Principle #1: Code is Expensive; Design First

When you're getting started with a new mobile project, it can be tempting to just dive right in and start building. There are some situations in which this makes sense - maybe you're trying to learn a new technology, or building a quick proof-of-concept of something tricky. But for he most part, you shouldn't write a line of code until you have a pretty good idea of what you're building.

Why should you design first? No matter how good of a developer you are, you can always work through design ideas with illustrations faster than you can code them. Moreover, a lot of newer design tools are more intuitive than their bloated precursors. 

Find your way to a great app design is about rapidly generating rough sketches, running them by potential users, and keeping the ones that work. 

Here's an example of why designing first makes more sense. Let's say you're trying to plan the navigation structure of your app, and you are trying to decide between tabbed browsing or using hamburger menu. Both have their merits, but you want to make sure you're picking the right one for your use case. The last thing you want is to go with one approach, code it, release it, only to realize your users find it confusing. A far better approach is to whip up a couple of quick sketches, and see which one people prefer. Design ideas are cheap, they can be discarded quickly if they don't work, but code takes time to rewrite and test.

###Sketching out Ideas

You don't need any design talent to scratch out some rough ideas and run them by friends or potential users. Here are some no-nonsense tools to get you started

#### Sketch

[Sketch](https://www.sketchapp.com/) is one of the most intuitive design tools I've ever seen. While programs like Adobe Illustrator have extremely steep learning curves, a complete newbie can be drawing app ideas in Sketch in minutes. It even comes with templates for app design pre-configured for the right dimensions. Basically, you can draw shapes and add text quickly and a rough mockup of an app idea in no time. It also comes with great tools for exporting assets and an extensive plugin library. 

Disclaimer: Sketch costs $99 for the full version, but I can say it is absolutely worth it if you're going to be doing regular sketching, and it's far cheaper than it's competitors. It does have a 30-day free trial if you expect to only need it for a few weeks.

#### InVision

[InVision](https://www.invisionapp.com/) is a free service that lets you create clickable/tappable prototypes from images you upload. If you've got some rough PNGs you've drawn in Sketch, you can quickly create a prototype you can share via a link with friends and potential users. You can even let them comment directly on your designs and iterate through to the best ideas. The company has built an incredible following among the design community for how easy it is to use. It also offers paid plans which provide a deeper feature set if you're serious

#### Pen and Paper

Seriously, one of the oldest design tools is still one of the best. Design agencies all over the world have conference rooms covered with whiteboards because drawing is one of the best ways to work through ideas. If you find digital tools to overwhelming, just close your laptop and get a pen and paper. No matter how rough your sketches, you'll be able to get an idea of the core screens that make your app in under an hour. At the very least, it'll give you a clear picture of what you're coding before you start.

## Principle #2: Don't Do Something New

This may sound like insane advice. After all, shouldn't your app solve people's problems in some new, inventive way? Actually no, successful products usually rely heavily on common design patterns with a bit of innovation thrown in in key areas.

Imagine if you were trying to design a new bike. But not just any bike, you're going to completely re-imagine the way people think about bikes. You decide that in order to innovate it's important you throw out all of these stupid conventions? Why should you use rubber tires? Just because we've been using rubber tires for a century doesn't mean we should continue doing so. Maybe polyurethane would be more durable. And instead of carbon fiber frames, you decide someone really needs to look into trying bamboo. And finally, you think the angles are all wrong - people clearly would prefer to lean back further as they ride, right? 

You can see where this is going. It's one thing to challenge some of the core assumptions underpinning a product's design, it's another thing to challenge *all* of them. The reality is that most well-established products are the culmination of extensive research and usability testing, and you can benefit by leveraging what works. 

### Leveraging Design Patterns

The easiest way to design an app that users will understand and love is to look at what other successful apps do. I'm not saying you directly copy someone else's design, I'm saying you draw inspiration from it and follow the patterns that work. 

When you're about to embark on a new design project, start by taking screenshots of all of the apps in your industry or just those that you like. How do they handle navigation? How do the handle registration? If your app has a chat interface, look at ten apps that do chat well side-by-side. Chances are you'll see a lot of similarities and you can include those similarities n your app. Using common paradigms helps users understand how your app works, since they've lkely seen similar patterns before. 

##Principle #3: Standardize, then Standardize Some More

React Native requires you to build your app as a series of components arranged in a hierarchy of parents and children. This approach has a lot of benefits for a developer - it enforces unidrectional data flow to improve debugging, it encourages code reusability, and it makes it easier to leverage open source components. But the component approach isn't just powerful for the developer, it makes it far easier to design a consistent experience.

When planning your app, keep an idea on how many similar components you're creating. How many different kinds of buttons do you have? If you answered six, ask yourself if you you can collapse some of your styles. How many different classes do you have for text? Most apps won't need more than three or four heading styles, a paragraph style, and one or two for buttons.  Better yet, if you reuse the same styles again and again, you can tweak them quickly and see how they look across the app.

### Identify Your Components

If you've made sketches of your app, you can probably start to draw boxes around your different kinds of views, cards, buttons, etc. You can then decide on intuitive names for these components before you translate them to code. If you spend a little time planning before you code, you can figure out which components will need to handle logic, and which will just be for presentation. For example, if you have a couple of button components, you can re-use them anytime you need a button, and just have them inherit a value `prop`. This way if you decide you want your buttons to be blue instead of green, all you need to do is update one value and try it out. 

