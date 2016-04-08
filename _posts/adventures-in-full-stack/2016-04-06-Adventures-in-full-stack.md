---
layout: post
title: Adventures in full stack
---

In order to update and consolidate my skills in web development, particularly the parts of the stack that have seen a lot of development in recent years (frontend frameworks), or which I have less experience of (managing cloud servers with configuration management tools), I've decided to develop a small project and write about it here.


## The project: Everybot ##

I'm going to write a flexible web app for creating Twitter bots in the style of [everyword](https://twitter.com/everyword). Everyword was a Twitter account that tweeted every dictionary word, in order; it inspired various successors such as [fuck every word](https://twitter.com/fuckeveryword) (each dictionary word with the word "fuck" in front of it") and [every word is gay](https://twitter.com/everywordisgay) (each dictionary word with "gay" in front of it). I've occasionally thought of mildly amusing ideas for accounts along similar lines, but as they were only ever mildly amusing, they've never seemed worth the effort of actually writing a bot to implement. But perhaps if you add together all the mild amusement involved in these ideas, and the ideas other people might come up with, it will end up being worth the effort. Anyway, it seems like a good sized project to work on, small but not completely toy.

The project will have three parts: a web app, i.e. and HTTP server that will allow for the creating and editing of bots; a backend, i.e., the actual bot which will post to Twitter; and a frontend, a single page JavaScript app that users will use to create and control their bots. In addition, the project will also involve a certain amount of infrastructure.


## The web app ##

I want to use a full stack framework for this part, because [Zawinski's law](http://www.catb.org/jargon/html/Z/Zawinskis-Law.html) means I'll probably need a bunch of things that work well together (at least, a REST routing framework, an ORM, a database migration system, authentication and interaction with Twitter OAuth), and choice is bad, particularly when you can have someone else make the choice for you. The two clear options for a full stack framework are [Ruby on Rails](http://rubyonrails.org/) or [Django](https://www.djangoproject.com/); I'm going to go with Ruby on Rails, as it will give me a chance to firm up my Ruby skills (I'm already pretty confident with Python). Although this is mostly supposed to be an API server for the JavaScript frontends, I'll also write a very basic HTML interface here that I can use for testing before I've written the actual frontend (so that's two more parts of the stack Rails saves me from making a choice about -- a template engine and an asset pipeline).


## The backend ##

The point of this whole thing is to periodically post tweets according to according to certain patterns, and this component will sit on a server, periodically wake up and see if anything needs to be posted, and if so post it. This is a long-running and primarily IO bound application so I want a language that makes it easy to do other stuff while you're waiting on IO, and makes it hard to make mistakes that will crash the application. The first requirement would include things like NodeJS, with its async IO, and the new async IO library in Python 3.4, but the second requirement makes me think a statically typed language would be better, and the two options I'm considering are Go and Haskell, which both have good concurrency support. I'll probably pick Haskell, as I have some familiarity with it but haven't had a chance to use it on a real-world application, and I'd like to find out more about that. I might re-implement the backend in Go later, as an opportunity to learn that language.


## The frontend ##

As the initial point of this project was to learn about newer frontend frameworks, I'm already planning to implement this at least twice. First, I'll try [Ember](http://emberjs.com/), which seems fairly widely used, and also seems appealingly full-featured from my "choice is bad" perspective. I'll also try [React](https://facebook.github.io/react/) (possible with [Redux](http://redux.js.org/)), which is an interesting, newer approach. The other widely-used frontend framwork seems to be [Angular](https://angularjs.org/), but it's currently undergoing a fairly extensive redesign, I'm not sure yet whether to try the current version or the future version, Angular 2 (or, frankly, just to skip it, as it looks overengineered and convoluted).


## The infrastructure ##

The first element of infrastructure I'll need is a [Vagrantfile](https://www.vagrantup.com/) to set up a contained and consistent development environment. I'll use [Ansible](https://www.ansible.com/) to script the configuration of this environment. I've picked Ansible because it seems like the simplest configuration management system; using a configuration management system at all will become more useful when I need to manage my test environment (a VM on my computer) and the production environment (a VM in the cloud somewhere). I could put the various components (the web server, the backend, the database) in different containers, and manage them that way, but I don't see any particular advantage to that for a simple system like this; I may experiment with [Docker](https://www.docker.com/) at a later date just to find out more about it.


