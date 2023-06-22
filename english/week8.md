<style>
td, th {
   border: 1px solid black;
}
</style>

# Stimulus framework

> For the more information, e.g. installation instructions can be found at https://stimulus.hotwired.dev/

Stimulus is a JavaScript framework used for adding interactivity to HTML without having to write a lot of custom JavaScript.<br />
It provides a set of JavaScript modules that can be attached to HTML elements with specific attributes, allowing you to write simpler and more maintainable code.<br />
It works well with Rails and uses the latest browser technologies to provide a fast and smooth user experience.

Pros:

- Encourages separation of concerns by keeping the HTML, JavaScript, and Ruby code separate.
- Helps reduce the amount of JavaScript that needs to be written in order to achieve common functionality.
- Provides a clear API for dispatching and listening to events on the page.
- Can improve performance by avoiding some of the overhead associated with a full JavaScript framework.

Cons:

- May not be suitable for more complex applications that require a lot of client-side functionality.
- Stimulus controllers can become bloated and difficult to manage if not carefully organized.
- Does not provide built-in support for state management or data fetching, which may be necessary for certain applications.
  - Stimulus takes a different approach. A Stimulus application's state lives as attributes in the DOM
- Could add an extra layer of complexity for developers who are not familiar with the framework.

## Stimulus framework components

Here is a basic breakdown of the primary building blocks of Stimulus applications.

### Controllers

When creating controller files for Stimulus, it is important to name them in a particular way. Specifically, the format for naming your controller files should be [identifier]\_controller.js, where the identifier refers to the data-controller identifier that is associated with each controller in your HTML.<br />
By following this naming convention, Stimulus will be able to properly link the controllers in your HTML with their corresponding JavaScript files.

Here is an example controller named "hello_controller.js".

```javascript
// /app/javascript/controllers/hello_controller.js
import { Controller } from "@hotwired/stimulus";

export default class extends Controller {}
```

Here's an example where HTML div -element has an identifier called `hello`, so it's associated to the Stimulus controller named hello_controller.js (example above).<br />
The's scope in Stimulus includes the element it's connected to and its children, but not the surrounding elements.
For example, the `<div>`, `<input>` and `<button>` below are part of the controller’s scope, but the surrounding `<h1>` element is not.

```html
<h1>Greetings</h1>
<div data-controller="hello">
  <input type="text" />
  <button>Greet</button>
</div>
```

### Actions

Actions are methods defined within a controller that respond to user events or changes in the application state. Actions are identified using the data-action attribute and can be triggered by a wide variety of events, such as clicks, form submissions, or even custom events.

Here's an example of adding action to the button -element `<button data-action="click->hello#greet">Greet</button>`. This code attaches a click event listener to the button element with the controller identifier "hello" and calls its "greet" method when the button is clicked.

```html
<h1>Greetings</h1>
<div data-controller="hello">
  <input type="text" />
  <button data-action="click->hello#greet">Greet</button>
</div>
```

### Targets

Targets are special attributes that allow a controller to reference and manipulate specific elements within its scope. Stimulus loads your controller class and checks for target names in the static "targets" array. It adds three properties for each target name found: "sourceTarget" evaluates to the first source target, "sourceTargets" evaluates to an array of all source targets, and "hasSourceTarget" returns true or false depending on if there is a source target present.

```html
<h1>Greetings</h1>
<div data-controller="hello">
  <input data-hello-target="name" type="text" />
  <button data-action="click->hello#greet">Greet</button>
</div>
```

Add the target's name to the controller's list of target definitions. A property with the name "nameTarget" will be automatically created which returns the first matching target element. This property can be used to read the element's value and build the greeting string. Update the hello_controller.js file accordingly.

```javascript
// /app/javascript/controllers/hello_controller.js
import { Controller } from "@hotwired/stimulus";

export default class extends Controller {
  static targets = ["name"];

  greet() {
    const name = this.nameTarget.value;
    console.log(`Hello, ${name}!`);
  }
}
```

### Values

Stimulus, values are essentially data that can be stored and accessed within a particular controller. They can be defined and accessed using the value method within a controller. A value can represent any kind of data, such as a string, number or boolean.

Values in Stimulus can be declared in several ways, including as a static value, an attribute on an HTML element, or a dynamic value returned by a function or Promise. Here's an example of how to define and access a value in a Stimulus controller:

```html
<h1>Greetings</h1>
<div data-controller="hello" data-hello-greeting-value="Welcome">
  <input data-hello-target="name" type="text" />
  <button data-action="click->hello#greet">Greet</button>
</div>
```

```javascript
// /app/javascript/controllers/hello_controller.js
import { Controller } from "@hotwired/stimulus";

export default class extends Controller {
  static targets = ["name"];
  static values = { greeting: String };

  greet() {
    const name = this.nameTarget.value;
    console.log(`${this.greetingValue}, ${name}!`);
  }
}
```

## Lifecycle methods

Stimulus provides a set of lifecycle methods that can be used to execute code when an instance of a controller is created, connected to the DOM, or disconnected from the DOM. These methods are defined in the controller class:

```javascript
  import { Controller } from 'stimulus';

  export default class extends Controller {
    constructor(...) {
      super(...);
      // This code is executed when an instance is created
    }

    connect() {
      // This code is executed when an instance is connected to the DOM
    }

    disconnect() {
      // This code is executed when an instance is disconnected from the DOM
    }
  }
```

These methods can be used, for example, to set up event listeners or clean up after the controller is removed from the page.

## Elements and default events

Stimulus has default events for elements. This means that click can be removed from the data-action for buttons.<br />
For example: `<button data-action="hello#greet">Greet</button>`.

Here is a complete list of the elements and their default events.

| Element           | Default Event |
| ----------------- | ------------- |
| a                 | click         |
| button            | click         |
| details           | toggle        |
| form              | submit        |
| input             | input         |
| input type=submit | click         |
| select            | change        |
| textarea          | input         |
