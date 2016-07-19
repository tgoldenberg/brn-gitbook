# Chapter 11: Adding a Calendar View

Now that we've successfully added notifications and sculpted our `ActivityView`, there's pretty much one view left to really build out, which is our `CalendarView`. We want to show a list of upcoming events, both those that our user is attending and those that they are not, in chronological order. We also want to have "sticky" headers for each day that we show  events. Let's see what we can do.

First let's set up the routing for our Calendar view.

```javascript
import React, { Component } from 'react';
import {
  Navigator,
  StyleSheet
} from 'react-native';

import Calendar from './Calendar';
import Event from '../groups/Event';

class CalendarView extends Component{
  render(){
    return (
      <Navigator
        initialRoute={{ name: 'Calendar' }}
        style={styles.container}
        renderScene={(route, navigator) => {
          switch(route.name){
            case 'Calendar':
              return (
                <Calendar
                  {...this.props}
                  {...route}
                  {...this.state}
                  navigator={navigator}
                />
            );
            case 'Event':
              return (
                <Event
                  {...this.props}
                  {...route}
                  {...this.state}
                  navigator={navigator}
                />
            );
          }
        }}
      />
    )
  }
}

let styles = StyleSheet.create({
  container: {
    flex: 1
  }
});

export default CalendarView;

```

Then let's flesh out `Calendar.js`.

```javascript
application/components/calendar/Calendar.js


```

