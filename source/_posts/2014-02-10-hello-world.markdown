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


