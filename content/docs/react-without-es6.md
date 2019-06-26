---
id: react-without-es6
title: ריאקט בלי Es6
permalink: docs/react-without-es6.html
---

לרוב, נגדיר קומפוננטה של ריאקט באמצעות מחלקה של JavaScript:

```javascript
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

אם את\ה עוד לא משתמש\ת עדיין ב-ES6, תוכל\י להשתמש במקום זאת ב-module `create-react-class`:

```javascript
var createReactClass = require('create-react-class');
var Greeting = createReactClass({
  render: function() {
    return <h1>Hello, {this.props.name}</h1>;
  }
});
```

ה-API של מחלקות ES6 דומה ל-`createReactClass()`, למעט מספר פרטים.

## הכרזה על Props ברירת מחדל {#declaring-default-props}

עם פונקציות ומחלקות ES6, `defaultProps` מוגדרת כתכונה של הקומפוננטה עצמה:

```javascript
class Greeting extends React.Component {
  // ...
}

Greeting.defaultProps = {
  name: 'Mary'
};
```

ב-`createReactClass()`, צריך להגדיר את `getDefaultProps()` כפונקציה של האובייקט שמועבר:

```javascript
var Greeting = createReactClass({
  getDefaultProps: function() {
    return {
      name: 'Mary'
    };
  },

  // ...

});
```

## הגדרת ה-State ההתחלתי {#setting-the-initial-state}

במחלקות ES6, ניתן להגדיר את ה-state ההתחלתי באמצעות השמה של `this.state` ב-constructor:

```javascript
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: props.initialCount};
  }
  // ...
}
```

ב-`createReactClass()`, צריך להביא מתודת `getInitialState` נפרדת שמחזירה את ה-state ההתחלתי:

```javascript
var Counter = createReactClass({
  getInitialState: function() {
    return {count: this.props.initialCount};
  },
  // ...
});
```

## Binding אוטומטי {#autobinding}

בקומפוננטות ריאקט שממומשות באמצעות מחלקות ES6, המתודות מתנהגות בצורה דומה (סמנטית) למחלקות ES6 רגילות. לפיכך, המתודות לא עושות bind אוטומטית למופע של המחלקה. תצטרך\תצטרכי להשתמש ב-`.bind(this)` בצורה מפורשת ב-constructor:

```javascript
class SayHello extends React.Component {
  constructor(props) {
    super(props);
    this.state = {message: 'Hello!'};
    // השורה הזו חשובה!
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    alert(this.state.message);
  }

  render() {
	// מכיוון ש-`this.handleClick` הוא bound, אפשר להשתמש בו כ-event handler
    return (
      <button onClick={this.handleClick}>
        Say hello
      </button>
    );
  }
}
```

עם שימוש ב-`createReactClass()`, זה לא נחוץ משום שנעשה bind לכל המתודות:

```javascript
var SayHello = createReactClass({
  getInitialState: function() {
    return {message: 'Hello!'};
  },

  handleClick: function() {
    alert(this.state.message);
  },

  render: function() {
    return (
      <button onClick={this.handleClick}>
        Say hello
      </button>
    );
  }
});
```

זה אומר שכתיבת מחלקות ES6 דורשת עבודה נוספת עבור event handler-ים, עם זאת יש יתרון בשיפור הביצועים בפיתוח אפליקציות גדולות.

אם כתיבת הקוד הנוסף (עם ה-binding) לא אטרקטיבית בעינייך, את\ה רשאי\ת לאפשר את הכתיב **הניסיוני** [Class Properties](https://babeljs.io/docs/plugins/transform-class-properties/) מ-proposal של Babel:

```javascript
class SayHello extends React.Component {
  constructor(props) {
    super(props);
    this.state = {message: 'Hello!'};
  }
  // WARNING: this syntax is experimental!
  // Using an arrow here binds the method:
  handleClick = () => {
    alert(this.state.message);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Say hello
      </button>
    );
  }
}
```

שים\שימי לב שהכתיב הזה הוא **ניסיוני** ועשוי להשתנות בעתיד, או שה-proposal לא יתקבל.
If you'd rather play it safe, you have a few options:
אם את\ה מעדיף\מעדיפה ללכת על בטוח, ישנן מספר אפשרויות:

* עשה\עשי bind למתודות בתוך ה-constructor.
* השתמש\י ב-arrow functions, לדוגמא `onClick={(e) => this.handleClick(e)}`.
* המשך\המשיכי להשתמש ב-`createReactClass`.

## Mixins {#mixins}

>**Note:**
>
>ES6 launched without any mixin support. Therefore, there is no support for mixins when you use React with ES6 classes.
>
>**We also found numerous issues in codebases using mixins, [and don't recommend using them in the new code](/blog/2016/07/13/mixins-considered-harmful.html).**
>
>This section exists only for the reference.

Sometimes very different components may share some common functionality. These are sometimes called [cross-cutting concerns](https://en.wikipedia.org/wiki/Cross-cutting_concern). `createReactClass` lets you use a legacy `mixins` system for that.

One common use case is a component wanting to update itself on a time interval. It's easy to use `setInterval()`, but it's important to cancel your interval when you don't need it anymore to save memory. React provides [lifecycle methods](/docs/react-component.html#the-component-lifecycle) that let you know when a component is about to be created or destroyed. Let's create a simple mixin that uses these methods to provide an easy `setInterval()` function that will automatically get cleaned up when your component is destroyed.

```javascript
var SetIntervalMixin = {
  componentWillMount: function() {
    this.intervals = [];
  },
  setInterval: function() {
    this.intervals.push(setInterval.apply(null, arguments));
  },
  componentWillUnmount: function() {
    this.intervals.forEach(clearInterval);
  }
};

var createReactClass = require('create-react-class');

var TickTock = createReactClass({
  mixins: [SetIntervalMixin], // Use the mixin
  getInitialState: function() {
    return {seconds: 0};
  },
  componentDidMount: function() {
    this.setInterval(this.tick, 1000); // Call a method on the mixin
  },
  tick: function() {
    this.setState({seconds: this.state.seconds + 1});
  },
  render: function() {
    return (
      <p>
        React has been running for {this.state.seconds} seconds.
      </p>
    );
  }
});

ReactDOM.render(
  <TickTock />,
  document.getElementById('example')
);
```

If a component is using multiple mixins and several mixins define the same lifecycle method (i.e. several mixins want to do some cleanup when the component is destroyed), all of the lifecycle methods are guaranteed to be called. Methods defined on mixins run in the order mixins were listed, followed by a method call on the component.
