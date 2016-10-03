---
layout: post
title: Game of life
date: '2016-09-29T08:00:00.001+02:00'
author: Tomasz Krug
tags:
- angular.js
- gulp
- canvas
modified_time: '2016-09-29T09:23:40.224+02:00'
---

So what do you do when you are a bit tired of Java but still want to create something small and fun? Personally I like to play with the HTML5 canvas element.

This time I implemented Conway's Game of life in Angular.js.

You can find the source on [GitHub](https://github.com/Edhendil/game-of-life).

If you don't want to build it yourself demo is available [here](https://edhendil.github.io/game-of-life/).

<!--more-->

#### What is it?

Let's say you have a grid of cells. Some of these cells are alive and some are dead. That's the initial state at t=0. 

To calculate the state at t=1 we have to check how many alive neighbours each of these cells has and follow these rules:

* if an alive cell has 2 or 3 alive neighbouring cells then it will be alive in the next state, otherwise it will be marked as dead
* if a dead cell has exactly 3 alive neighbouring cells then it will be marked as alive in the next state, otherwise it will be marked as dead

Repeat these steps for t=2, 3, 4 and so on as long as you want.

Now we just have to display these states. This is done by painting the canvas element according to the current state. Alive cells are shown as black pixels while dead cells are white.

#### But what for?

The set of rules governing this game is very simple but the patterns that emerge are complicated. There are many different structures and they can evolve for hundreds of steps. Some of them are immortal, some are die out quickly. There are breeders, spaceships, glider guns, oscillators. It's just fun to watch.

What's more you don't have to stick to the original rules. There are many others that generate interesting results.

For more information you can check [Wikipedia](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life).

#### Technical aspect

This project also acts as a template for an Angular application. I use bower to download dependencies and then gulp to concatenate and minify all sources. 

In the gulpfile.js file there are defined tasks for checking code style, formatting the code, running a simple local web server and watching for changes in the sources.

README file on GitHub repository contains instructions for building and running this application on localhost.