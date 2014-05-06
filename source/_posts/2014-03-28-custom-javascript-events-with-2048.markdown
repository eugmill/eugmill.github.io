---
layout: post
title: "Custom Javascript Events with 2048"
date: 2014-03-28 10:54:09 -0500
comments: true
categories: javascript 
---

After wasting a few hours playing with 2048, I realized that hey, I’m learning javascript, and 2048 is Javascript! I dove into the 2048 source code to see what I could find.

2048 is entirely client side, which means there is no server to manage. A high level view of the code is below:

* *application.js* – Launches the game
* *game_manager.js* – Acts as the main controller
* *grid.js* – Represents the state of the game board
* *tile.js* – Represents a single tile object
* *html_actuator.js* – Deals with the front end and updating the html itself.
* *keyboard_input_manager.js* – Deals with the input, whether it be keyboard or phone swipes
* *local_storage_manager.js* – Deals with saving game state.

Great! Let’s take a look. When the game is started, the GameManager constructor takes in the html_actuator, keyboard_input_manager and storage_manager objects as arguments. We do some basic initializing, and then this:

```javascript
function GameManager(size, InputManager, Actuator, StorageManager) {
  ...
  this.inputManager.on("move", this.move.bind(this));
  this.inputManager.on("restart", this.restart.bind(this));
  this.inputManager.on("keepPlaying", this.restart.bind(this));
  ...
}
```

Up until then, I had only really dealt with pretty basic DOM events. Click, hover, submit, were dependable and familiar. Wtf is restart?

###Custom Events

Some digging led me to reading about custom events in Javascript. Custom events are pretty much exactly like DOM events, but both the dispatches and the listeners are defined by you.

From the mozilla documentation: 

```javascript
// Listen for the event.
elem.addEventListener('build',function (e) {...},false);

// Dispatch the event.
elem.dispathEvent(event);
```

You can also include data in the events you throw.
```javascript
var event = new CustomEvent('build', {'detail': elem.dataset.time});

function eventHandler(e) {
  log('The time is: '+e.detail);
}
```

So where does that leave us? Let's take another look at the 2048 code.
```javascript
KeyboardInputManager.prototype.on = function (event, callback) {
  if (!this.events[event]) {
    this.events[event] = [];
  }
  this.events[event].push(callback);
};

KeyboardInputManager.prototype.emit = function (event, data) {
  var callbacks = this.events[event];
  if (callbacks) {
    callbacks.forEach(function (callback) {
      callback(data);
    });
  }
};

KeyboardInputManager.prototype.listen = function () {
  var self = this;
  ...
  document.addEventListener("keydown", function (event) {
    ...
        event.preventDefault();
        self.emit("move", mapped);
    ...
  });
}
```

The “on” method lets us attach callbacks to custom events in the InputManager object. The emit function lets us throw custom events with directional data attached to them.

Let’s go over the steps in order

1. I hit the up arrow on the keyboard
2. The listen function captures it and “emits” a “move” event with data “up”
3. The emit function calls the callback for “move”, which points to the move function in the GameManager. 

What about this weird bind thing? Well, the “on” method is defined in the InputManager object, but the callback function, “move”, is defined in the Gamemanager.

We are telling the callback that when it refers to “this”, it should refer to the GameManager, not the InputManager, even though it will be called from the InputManager.

In conclusion, reading through other people’s source code can be both educational and useful. Since I’ve started reading through it, not only have I become a better developer, but it makes playing 2048 itself a lot easier:

{% img center /images/2014-03-28-2048.png %}
