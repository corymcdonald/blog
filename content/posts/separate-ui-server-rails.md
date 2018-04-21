---
title: "Separate Ui Server Rails"
date: 2018-03-03T09:00:54-06:00
draft: true
---


<!-- # Separating the UI and Server from your Rails application -->

If you're on a large team or accepting a large amount of contributions ot your project I believe it makes sense for you to consider separating your frontend and backend code. This can provide the following benefits.

1. Modularity

  When you separate your frontend and backend code you gain the benefit of modularity. As applications grows it becomes easier to mix logic into your views that would have been better suited for a presenter or a decorator. When you divorce your frontend and backend it becomes much more difficult to introduce that logic that would violate the Law of Demeter.

2. Parallelization

  It's easy to say that a task can be worked in parallel but in practice it can be significantly more difficult. When you divorce your user interface and backend two programmers can agree and [design by contract](https://en.wikipedia.org/wiki/Design_by_contract) for what will be passed to the user interface. With this contract the two developers can work in parallel, since there the contract will most likely have mock data it can be easier to follow test-driven development.

3. Coherent controllers

  After working Ruby on Rails and similar MVC frameworks it seems that most views end up mixing instance variables from mulitple different classes. With the edition that aren't explicitly defined.

4. Demonstrate VS Code Git
