---
layout: post
title: Ruby AOP for Observability Experiment
date: 2023-04-28 08:07 -0400
categories: SoftwareDevelopment Observability Ruby
---

For a while now I have been thinking about how to add composable observability to my production ruby platform. In the past I have seen the wiring of such tools happen in a shared baseclass that ends up connecting all components of the system and slowly growing with shared behaviors. Of course that's more of a culture problem than a programming pattern problem. ¯\\_(ツ)_/¯

Focusing on modularity I put together an example [https://github.com/ninjapanzer/ruby_aop_logging_aspect_example](https://github.com/ninjapanzer/ruby_aop_logging_aspect_example)

as a kind of proof of what a more modular/composable kind of observability could look like. This project enables annotations like that found in #sorbet `sig { params(...).void }` except it adds metrics logging to a specific method.


Only a toy but with some batteries included. Self contained projects are much easier for the unfamilar to experiment with. Regardless of if you ever use this pattern in ruby, being able to identify it and have it not be a mystery the real win, so I hope this gets your gears turning :)


More on AOP

[https://en.wikipedia.org/wiki/Aspect-oriented_programming](https://en.wikipedia.org/wiki/Aspect-oriented_programming)
