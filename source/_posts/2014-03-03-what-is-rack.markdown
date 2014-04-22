---
layout: post
title: "What is Rack, Really?"
date: 2014-03-03 09:09:39 -0400
comments: true
categories: [rack,ruby]
---

After spending some time on Ruby foundations, we’ve finally moved onto the web at Flatiron School's’s 5th Ruby class. The first topic we covered was [rack](http://rack.github.io/). According to the official site:

>"Rack provides a minimal interface between webservers supporting Ruby and Ruby frameworks.”

Well, that helps. Let’s break it down. What exactly is a web server? In its simplest form, you can implement a web server by doing the following:

1. Listen to a port (80 most of the time) until a request comes in
2. Deal with the request
3. Output the response
4. goto 1

That sounds great! But we don’t want to implement that from scratch. Luckily, there are plenty of web servers already out there! Not only does rack let you easily switch between web servers, it also let’s you use the same middleware across different applications.


###Taking a step back
In essence, there was a time where if you wanted to write a web app in Ruby, you had to write the interface to the server, which would change depending on the server you used. No more! 

Some big points to take away: 

1. Ruby is not a language for web development. 
Given the immense popularity of Rails, it’s easy to assume that Ruby was created as a language for web development. In reality, it was created as a general purpose language, and the tools for web development were added over time. Rack and rails are two such tools. The former is a framework, and Rack is the interface that connects the framework to the servers themselves.

2. The web itself is language agnostic
If we take a really big picture view of the web, it is just a bunch of http requests getting flung back and forth between computers. Everything is text and ultimately, whatever language is used on the server will have to create and receive http requests. Rack handles that aspect for Ruby applications. 

Sources:

http://stackoverflow.com/questions/15875941/creating-a-web-app-in-ruby-without-a-framework

http://www.youtube.com/watch?v=iJ-ZsWtHTIg

http://rack.github.io/

http://chneukirchen.org/blog/archive/2007/02/introducing-rack.html
