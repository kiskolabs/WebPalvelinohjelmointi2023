# Stimulus framework

Stimulus is a JavaScript framework used for adding interactivity to HTML without having to write a lot of custom JavaScript.<br />
It provides a set of JavaScript modules that can be attached to HTML elements with specific attributes, allowing you to write simpler and more maintainable code.<br />
It works well with Rails and uses the latest browser technologies to provide a fast and smooth user experience.

> For the more information, e.g. installation instructions can be found at https://stimulus.hotwired.dev/

Pros:

- Helps reduce the amount of JavaScript that needs to be written in order to achieve common functionality.
- Provides a clear and easy-to-use API for dispatching and listening to events on the webpage.
- Can improve performance by avoiding some of the overhead associated with a full JavaScript framework.

Cons:

- May not be suitable for more complex applications that require a lot of client-side functionality.
- Stimulus controllers can become bloated and difficult to manage if not carefully organized.
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

Example below has a div -element with the data-controller attribute. Attribute has a value `hello`, so it's associated to the Stimulus controller named hello_controller.js (example above).<br />
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

Actions are methods defined within a controller that respond to user events or changes in the application state. Actions are identified using the data-action attribute and can be triggered by a wide variety of events, such as clicks, form submissions, or even custom events.<br />
Here's an example of adding action to the button -element `<button data-action="click->hello#greet">Greet</button>`. This code attaches a click event listener to the button element with the controller identifier "hello" and calls its "greet" method when the button is clicked.

```html
<h1>Greetings</h1>
<div data-controller="hello">
  <input type="text" />
  <button data-action="click->hello#greet">Greet</button>
</div>
```

Stimulus has default events for elements. This means that example click can be removed from the data-action for buttons.<br />
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

### Targets

Targets are special attributes that allow a controller to reference and manipulate specific elements within its scope. Stimulus loads your controller class and checks for target names in the static "targets" array. It adds three properties for each target name found: "sourceTarget" evaluates to the first source target, "sourceTargets" evaluates to an array of all source targets, and "hasSourceTarget" returns true or false depending on if there is a source target present.

```html
<h1>Greetings</h1>
<div data-controller="hello">
  <input data-hello-target="name" type="text" />
  <button data-action="hello#greet">Greet</button>
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

Values are data that can be stored and accessed within a controller by using the value method. Values can be declared in several ways such as static, attribute on an HTML element or dynamic. The example shows an example of how to add value to an attribute of an HTML element and how to define and access a value in a Stimulus controller.<br />

The structure of the HTML attribute to a value is data-[controller name]-[attribute name]-value. At the example below attribute is `data-hello-greeting-value` and the setted value of it is `Welcome`.

```html
<h1>Greetings</h1>
<div data-controller="hello" data-hello-greeting-value="Welcome">
  <input data-hello-target="name" type="text" />
  <button data-action="hello#greet">Greet</button>
</div>
```

A static values array is created in the controller, and it includes the attribute name `greeting` and it's type `String`. By adding the attribute `data-hello-greeting-value="Welcome"` to the div element, the value `"Welcome"`is assigned to the `greetingValue` variable in the controller. This value can then be accessed in the controller code.

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
    initialize()	{
      // Once, when the controller is first instantiated
    }

    [name]TargetConnected(target: Element) {
      // Anytime a target is connected to the DOM
    }

    [name]TargetDisconnected(target: Element) {
      // Anytime a target is disconnected from the DOM
    }

    connect() {
      // This code is executed when an controller is connected to the DOM
    }

    disconnect() {
      // This code is executed when an controller is disconnected from the DOM
    }
  }
```

These methods can be used, for example, to set up event listeners or clean up after the controller is removed from the page.

## Ratebeer example

Lets update users ratings delete functionality so, that we will use Stimulus to create ability the delete multiple ratings simultaneously instead of deleting only one at a time.<br />
Lets start by creating a new partial file called `_ratings.html.erb` in the `/app/views/users` folder.<br />
After that move ratings code part (below) from the `/app/views/users/show.html.erb` file to the ratings partial file `/app/views/users/_ratings.html.erb`.

```erb
<ul>
  <% @user.ratings.each do |rating| %>
    <li>
      <%= "#{rating.score} #{rating.beer.name}" %>
      <% if @user == current_user %>
        <%= button_to 'delete', rating, method: :delete, form: { style:'display:inline-block;', data: { 'turbo-confirm': 'Are you sure?' } } %>
      <% end %>
    </li>
  <% end %>
</ul>
```

Delete the ratings code from `show.html.erb` and add ratings -partial `<%= render partial: 'ratings' %>` to under the ratings header.

```erb
<h4>Ratings</h4>
<%= render partial: 'ratings' %>
```

After that lets modify partial by removing list elements and delete button.

```erb
<% @user.ratings.each do |rating| %>
  <%= "#{rating.score} #{rating.beer.name}"%>
<% end %>
```

First we add div element with two attributes first we add data-controller attribute `data-controller="rating"` and then second class attribute `class="ratings mb-4"`.<br />
Then we add surrounding div element `<div class="rating">` for each row with class rating.<br />
Inside that div we add input field `<input type="checkbox" name="ratings[]" value="<%= rating.id %>" />` and span element ` <span><%= "#{rating.score} #{rating.beer.name}" %></span>` for the rating data.<br />
Last we delete button `<button data-action="rating#destroy">Delete selected</button>` with attribute `data-action="rating#destroy"` to call stimulus controller `rating_controller.js`.<br />
Checkbox input field and delete button are surrounded with if clause to check that current user is the one who made the rating.

```erb
<div data-controller="rating" class="ratings mb-4">
  <% @user.ratings.each do |rating| %>
    <div class="rating">
      <% if @user == current_user %>
        <input type="checkbox" name="ratings[]" value="<%= rating.id %>" />
      <% end %>
      <span><%= "#{rating.score} #{rating.beer.name}" %></span>
    </div>
  <% end %>
  <% if @user == current_user %>
    <button data-action="rating#destroy">Delete selected</button>
  <% end %>
</div>
```

Now we have templates ready, so lets modify routes.rb `/app/config/routes.rb` for the ratings destroy.<br />
Let's remove destroy from ratings resources (below) and add new separate delete method, because we want to remove rating id from delete method route.<br />
We did this because we want ability send several ratings IDs once and not only one.

```ruby
resources :ratings, only: [:index, :new, :create]
delete 'ratings', to: 'ratings#destroy'
```

Lets modify ratings_controller destroy -method to handle several rating IDs. <br />
First we read ratings IDs from the request body `request.body.string.split(',')` and then we convert string value to array of the IDs.<br />
Loop through rating IDs `destroy_ids.each do |id|` and find them by id `Rating.find(id)` and if the current user is the one who made the rating, then we destroy the rating.<br />
Add rescue `rescue StandardError => e` for the inside rating IDs loop, so if some reason destroy fails or Rating couldn't be found, it catches error and will continue loop after the error.<br />
After destroy we want to return ratings partial to javascript so we can update the view without reload, so we add respond_to part, before that we reload user data to be fresh.

```ruby
def destroy
  destroy_ids = request.body.string.split(',')
  destroy_ids.each do |id|
    rating = Rating.find(id)
    rating.destroy if current_user == rating.user
  rescue StandardError => e
    puts "Rating record has an error: #{e.message}"
  end
  @user = current_user
  respond_to do |format|
    format.html{ render partial: '/users/ratings', status: :ok, user: @user }
  end
end
```

Now let's create Stimulus ratings controller file `ratings_controller.js` to the folder `/app/javascript/controllers/`.<br />
Open file and import `Controller` class from the `@hotwire/stimulus` module. <br />

```javascript
import { Controller } from "@hotwired/stimulus";
```

After that we define class that extends the Controller class. <br />

```javascript
export default class extends Controller {}
```

Inside the class we add `destroy` method and first we add confirmation dialog to inside that method.<br />
The `confirm()` function displays a dialog box with the message `"Are you sure you want to delete these selected ratings?"` and provides the user with two options, OK and Cancel.<br />
If the user clicks OK, the confirm() function returns true, otherwise it returns false and with false we return and rest of the code won't be executed.

```javascript
export default class extends Controller {
  destroy() {
    const confirmDelete = confirm(
      "Are you sure you want to delete these selected ratings?"
    );
    if (!confirmDelete) {
      return;
    }
  }
}
```

Then we select all the input elements with the attribute `name="ratings[]"` that are currently checked.<br />
The `querySelectorAll` method returns a NodeList and `Array.from()` is used to convert the NodeList into an array.<br />
The second argument of `Array.from()` is a mapping function `(checkbox) => checkbox.value` and it extracts the value property of each checkbox.<br />
The resulting array `selectedRatingsIDs` will contain the values of the selected checkboxes.<br />

```javascript
const selectedRatingsIDs = Array.from(
  document.querySelectorAll('input[name="ratings[]"]:checked'),
  (checkbox) => checkbox.value
);
```

Then we select the CSRF token from the meta tag `meta[name="csrf-token"]` and its content, then we add the content to the header object value with key `X-CSRF-Token`. <br />
By including the CSRF token in the fetch header, the server can verify that the request originated from the authenticated user and was not generated by a malicious third-party.

```javascript
const csrfToken = document.querySelector('meta[name="csrf-token"]').content;
const headers = { "X-CSRF-Token": csrfToken };
```

After that we fetch request to `/ratings` endpoint with method `DELETE` with headers object and the selectedRatingsIDs array as the request body.<br />

```javascript
fetch("/ratings", {
  method: "DELETE",
  headers: headers,
  body: selectedRatingsIDs,
});
```

If the response is okay, it retrieves the response body as text using response.text() and sets the HTML of the element with the class "ratings" to the retrieved text using document.querySelector().innerHTML.<br />
If the response is not okay, it throws an error with the message "Something went wrong".<br />
Any errors that occur during the fetch request or response handling are caught using .catch() and logged to the console.

```javascript
.then((response) => {
  if (response.ok) {
    response.text().then((html) => {
      document.querySelector("div.ratings").innerHTML = html;
    });
  } else {
    throw new Error("Something went wrong");
  }
})
.catch((error) => {
  console.log(error);
});
```

Here are all the blocks from above together as a `ratings_controller.js` file. <br />
Now we have working example where user can delete one or multiple ratings at once.

```javascript
import { Controller } from "@hotwired/stimulus";

export default class extends Controller {
  destroy() {
    const confirmDelete = confirm(
      "Are you sure you want to delete these selected ratings?"
    );
    if (!confirmDelete) {
      return;
    }

    const selectedRatingsIDs = Array.from(
      document.querySelectorAll('input[name="ratings[]"]:checked'),
      (checkbox) => checkbox.value
    );

    const csrfToken = document.querySelector('meta[name="csrf-token"]').content;
    const headers = { "X-CSRF-Token": csrfToken };

    fetch("/ratings", {
      method: "DELETE",
      headers: headers,
      body: selectedRatingsIDs,
    })
      .then((response) => {
        if (response.ok) {
          response.text().then((html) => {
            document.querySelector("div.ratings").innerHTML = html;
          });
        } else {
          throw new Error("Something went wrong");
        }
      })
      .catch((error) => {
        console.log(error);
      });
  }
}
```

<blockquote>

## Exercise 1

1. Add select all checkbox input to the users ratings partial
2. Create method to the Stimulus ratings -controller to handle click event and select/deselect all the ratings

</blockquote>

<blockquote>

## Exercise 2

</blockquote>

<blockquote>

## Exercise 3

</blockquote>
