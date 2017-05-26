---
layout: post
title: Massive Tanks
date: '2014-08-03T16:00:00.001+02:00'
author: Tomasz Krug
tags:
- Java
- Phaser
- Atmosphere
- Dyn4j
modified_time: '2014-09-06T22:23:40.224+02:00'
blogger_id: tag:blogger.com,1999:blog-5263052209480207518.post-4139681135351879300
blogger_orig_url: http://www.castlecodestein.com/2014/08/massive-tanks.html
---

<div style="text-align: center;"><a href="https://massivetanks.com/client/assets/sprites/logo.png"><img style="display: inline-block;" src="https://massivetanks.com/client/assets/sprites/logo.png" /></a></div> 

Last year together with my friends we took part in a 10 hour long coding marathon organized by SII in all major cities of Poland. We decided to create a multiplayer 2D game inspired by Super Tank from NES. And it was a good choice as we managed to win the first prize in Poznań and the fourth place in the second stage of the contest - the internet poll for the best application.

The source code is available on [GitHub](https://github.com/LetsCoders/MassiveTanks). It is not the version we completed in 10 hours but a heavily refactored one after a lot of bugfixes.

[Try it out yourself!](https://massivetanks.com)

The game server internally uses:
* [Atmosphere](https://github.com/Atmosphere/atmosphere) framework (websocket communication)
* [Dyn4j](http://www.dyn4j.org/) physics engine
* Google [Guice](https://github.com/google/guice)
* Google [Guava](https://github.com/google/guava)
* [Jackson](https://github.com/FasterXML/jackson) for converting objects to JSON.

In the web client we used [Phaser](http://phaser.io/) as a canvas renderer and javascript physics engine. 
