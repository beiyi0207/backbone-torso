# ![backbone-torso](http://vecnatechnologies.github.io/backbone-torso/images/black_website_logo.svg)

[Backbone](http://www.backbonejs.org) provides many convenient tools for building Single Page Applications (SPAs) out of the box, and we absolutely love using it!  However, Backbone only gives us the bare bones (excuse the overused pun) of what you need to build a complex front end application. Torso gives you the blueprints and some extra tools to build something quickly that scales well. Torso was first created by [Vecna](http://www.vecna.com) employees that needed to build robotics and health care applications that have intense feature sets and large user bases.

### What is Torso?

Torso is a two-part project. The first part is the [guidelines](/docs/RULES.md) on the principles of a good Backbone (including Torso) application. These come from hard-fought lessons building large-scale Backbone webapps. The second part is a set of Backbone [modules](http://vecnatechnologies.github.io/backbone-torso/#modules) that you can bring in all at once, or use pick-and-choose style, that fill functionality holes in Backbone. These additions to Backbone's base building blocks include form handling, two-way binding, smart rerendering, data validation tools, polling caches, sub-view management, and more.

We've created Torso's documentation through a tool called [TonicDev](http://tonicdev.com). Typically, tonicdev lets you run code inline to your documentation, but because of the DOM manipulation and setup of Torso, the code blocks don't run. But you can still see "live" examples inline to the documentation. 

#### Torso Intro
An explanation about what Torso is and how to get it
- **Intro**: http://tonicdev.com/torso/introduction

#### Torso Views

What makes them special?
- **Basics**: http://tonicdev.com/torso/view-basics
- **Lifecycle**: http://tonicdev.com/torso/view-lifecycle
- **Child Views**: http://tonicdev.com/torso/view-child-views
- **Feedback**: http://tonicdev.com/torso/view-feedback
- **Forms**: http://tonicdev.com/torso/view-forms
- **Transitions**: http://tonicdev.com/torso/view-transitions

More documentation to come on the other Torso modules.

Besides the guidelines and modules, there is a [documentation](/docs/DOCUMENTATION.md) page that touches on every module of Torso but at a higher level than the tonicdev pages. Also, a list of the added modules and their API is found [here](/docs/API.md).

### Play with Torso Live:
You can look at a torso application setup [here](https://cdn.rawgit.com/vecnatechnologies/backbone-torso/master/demo/dist/index.html). It's extremely basic, but if you inspect the source code, you'll see a standard torso application setup with detailed comments.


### Getting started  
If you want to jump right in,
Torso has a [yeoman](http://www.yeoman.io) [torso generator](https://github.com/vecnatechnologies/generator-torso) to get you off to a fast start.
#### Install yeoman
> npm install -g yo

#### Install generator-torso
> npm install -g generator-torso

#### Create a new project
> cd path/to/project
> yo torso `<name of project>`

#### How to bring Torso in
`let Torso = require('backbone-torso');`

Remember that Backbone.$ needs to be set. Torso relies on Backbone.$ being set as well.

`require('backbone').$ = require('jquery');`

#### What's in it?  
The yeoman generator will have created a simple application. You can open the browser at `localhost:3000` if gulp watch is on or open `file:///<path to project>/dist/index.html` to see the "hello world". Try adding another perspective view by using `yo torso:pod demo` and adding an entry in the router. Now, you should be able to switch between "web pages" by adding and removing "#demo" from the url.

### What does Torso look like?

**2-way bindings for forms**:

form-template.hbs
```
// The HTML form for a name field. Just add a data-model attribute to an input or select tag
<form>
  <label for="name">Name</label>
  <input name="name" type="text" data-model="name"></input>
  <button id="submit">Submit</button>
</form>
```

MyFormView.js
```
Torso.FormView.extend({
  template: require('./form-template.hbs'),

  events: {
    'click #submit': 'submit'
  },

  initialize: function(personModel) {
    this.personModel = personModel;
    // The form view's model can act as the mediator between the html form and another model.
    this.model.setMapping('person', 'name', this.personModel);
  },

  submit: function() {
    console.log(this.personModel.toJSON()); // {}
    this.model.push();
    console.log(this.personModel.toJSON()); // {name: 'text from your input'}
  }
})
```

**Feedback**

feedback-template.hbs
```
<input type="text" class="my-input"></input>
<div data-feedback="input-feedback"></div>
// Just add a data-feedback attribute to re-render that element at times you choose
```

MyView.js
```
Torso.View.extend({
  template: require('./feedback-template.hbs'),
  feedback: [{
    when: {
      '.my-input': ['keyup']
    },
    then: function(event) {
      return {
        text: 'You wrote: ' + this.$el.find('.my-input').val()
      };
      // If you used a form model, you could do: this.model.get('field-name') instead of grabbing from the dom
    },
    to: 'input-feedback'
  }]
})
```

**Rendering**

my-template.hbs
```
<div class="{{view.color}}">
  {{index}}: {{model.name}}
</div>
```

MyView.js
```
Torso.View.extend({
  template: require('./my-template.hbs'),

  initialize: function() {
    this.model = new Torso.Model({name: 'Bob'});
    this.set('color', 'blue');
    // because view has a cell built in, you can also do:
    // this.on('change:color', this.render);
  },

  prepare: function() {
    var context = Torso.View.prototype.prepare.call(this);
    console.log(context); // {model: {name: 'Bob'}, view: {color: 'blue'}}
    context.index = 1;
    return context;
  }

  // if you only need model and viewState, no need to override prepare
})
```


**Child View**

parent-view-template.hbs
```
<div>I'm a parent</div>
<div inject="my-child-view"></div>
```

ParentView.js
```
Torso.View.extend({
  template: require('./parent-view-template.hbs'),

  initialize: function() {
    this.childView = new ChildView();
  },

  attachTrackedViews: function() {
    this.attachView('my-child-view', this.childView);
  }
})
```

### Philosophy
You might have heard "convention over configuration" as a way to hide framework implementation and reduce code. Typically, if you have desires to do something that lies just over the edge of convention approaching the unconventional, those frameworks are a headache to get working. Torso is a how-to for "configuration using convention". The goal of Torso is to define some abstractions, [rules of thumb](/docs/RULES.md), and conventions that sit on top of an unopinionated framework like Backbone, which allows us to move quickly through the easy bits and still have the flexibility to handle the edge cases.

To see this, let's examine a simple Backbone View:
``` js
var basicView = Backbone.View.extend({
  events: {
    'click .list-item': 'handler'
  },
  initialize: function() {...},
  render: function() {...},
  handler: function() {...}
});
```
Backbone offers an easy starting point, but there are many unanswered questions.
* What happens when you add event listeners or timers that need to be removed after the view is done?
* Should the render method take arguments?
* Does calling render change the state of the application?
* When can I call render on this view?
* Is there any way to keep the functionality of the view going but not have it be part of the DOM?

Each point raised here can break functionality if it is inconsistent within your app. With Torso, we lay down the laws that answer questions just like these and keep your application consistent.
A Torso view's render methods never take arguments, can be called at any time, and never change the state of the application. We'll explain how to make this possible, as well as covering other rules, [here](/docs/RULES.md).

### Behaviors
[Behaviors](/modules/Behavior.js) are a way for torso to share view mechanics between different view hierarchies.

Here are some behaviors that come with torso:
* [Data Behavior](/docs/modules/behaviors/DATA_BEHAVIOR.md)

### Want to help out?
Check out the [dev page](/docs/DEVELOPMENT.md).

## Credits
Originally developed by [Vecna Technologies, Inc.](http://www.vecna.com/) and open sourced as part of its community service program. See the LICENSE file for more details.
Vecna Technologies encourages employees to give 10% of their paid working time to community service projects.
To learn more about Vecna Technologies, its products and community service programs, please visit http://www.vecna.com.
