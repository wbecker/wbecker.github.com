---
layout: default
title: Good Node, Bad Node
---

Node.js is apparently famous for being hard to write nice code. Its async tendencies can create "pyramids", which are seen as ugly. 

Having starting using node just recently I have found that I have been pumping out code at great rates, but on the downside, have been doing it all in the same file, with a long chain of code dangling off one or two entry points. 

So, this initial set of posts will investigate what could be seen to be wrong with my code, and how to make things less wrong. 

Before we start, you might like to get a feeling for the application [here](http://chattrr.net). It's a bookmarklet that allows you to chat with other people looking at the web site you are on. It's a good example, since the archetypal node application is a chat program, but this one takes it a couple steps further than normal.

To start off with, let's analyse my code and see what it is doing. It's 995 lines that does a fair amount of things: 

 * Regularly sends messages out to clients, informing them who is using the system.
 * Creates a server which:
   * Serves static content (javascript, css, html)
   * Creates a unique bookmarklet id
   * Handles a REST request to get a paged log of messages, rendered using jade
 * Handles shutting down the server and database cleanly on a SIGINT.
 * Works out which clients are on
 * Acts on clients from users which may:
   * Create a new user
   * Ensure the user has sent a matching password if one is set, and tells them if they haven't.
   * Determines what URL they should be chatting upon (this may be forced by the user, or will be decided based on the usage of other URLs).
   * Sends necessary data for when the user first joins a channel
   * Change a user's nick name
   * Change a user's password
   * Tell a user who else is on their channel
   * Set parameters relating to which URL they should visit
   * Sets whether the screen should flash when a new message is received
   * Sets the amount of history they receive when they log on
   * Set the language their text should be translated to.
 * Broadcasts or sends annonucements to users, translated into the language of their choosing.
 * Handles user's logging off
 * Centralises the creation of redis keys
 * Some now unused code that was needed to migrate some redis keys from one datatype to another.
 * Loads relevant data from a config file.

So there's about 25 features there, an average 40 lines each - better than Java!

One question that may be asked is how can you handle dealing with such a big file of disparate features? I have been working on this project non-stop out of business hours for two weeks, so I have kept the entirety of how it works in my short term memory. I had better put some order in before I start forgetting things!

What's good:

 * Code is arranged is a sensible manner, being located in a way which mimics the program's flow.
 * Functions are well named and do one thing.
 * Code is not repeated.
 * It passes jsLint.

This is what a normal programmer can do, hopefully without too much thought! The problem comes in imposing some structure on these basics. What are symptoms of the problem:

 * Some code get's quite indented - 7 levels.
 * There are 15 global variables, plus the require statements.
 * All the functions are assigned to a global object, just so they didn't have to be defined with the other globals.
 * handleMessageContents(), between line 480 - 540 is a bunch of else if statements, depending on the contents of the message. Although each condition does the bulk of it's work in another function, this can be improved.

So what are we going to do:

 * Try to break up code into smaller files, aiming for no more than 200 lines each. We do this, with the understanding that each file will be a complete module that can be understood on its own, with a minimal understanding of the broader context.
 * Eliminate global variables: pass variables to the functions that need them. Reducing global state will mean that there is less dependency between sections.
 * Break up overly indented code (ifs+loops+async functions), so no function has more than one level of function nesting. 
 * Refactor handleMessageContents to have functions registered against each possible message parameter. Then when a message is received, instead of going through a set of if/elses, the right function can be directly found.
