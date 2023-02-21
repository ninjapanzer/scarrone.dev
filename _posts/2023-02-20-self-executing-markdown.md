---
layout: post
title: Self Executing Markdown
date: 2023-02-20 23:33 -0500
categories: Ideation
---

## Inspiration

In most open and closed source software at some point we have to onboard new developers. There is the challenge that if we build automation scripts and supporting documentation they may drift from each other. This often leads to make files and bash scripts that are platform locked and sometimes poorly documented. Leaving us challenged when dependencies drift or an upstream adds new features that are auto-configured for existing users but un-configured for new users.

Any who what if these documents were the same item biased towards explanation vs scripting. The solution is a toolchain that can support multiple target architectures while being readable and informative.

## The Idea

Its pretty simple, extend markdown fenced codeblock syntax to respect some additional metadata allowing for executing shell commands. It would look a little something like this.

    ```bash {platform="ubuntu", exec="sh", sudo="false", args=""}
      gem install rails
    ```
or
    ```ruby {platform="macos", exec="ruby", sudo="false"}
      puts "hello"
    ```

The meta information here is reminiscent of `#!`(shebangs) thus when our parser observes this file it can make a sane system call to the right command. When we render the markdown to HTML this information disappears into attributes of the render markup but when parsed with a more fluent tool it can act as as a procedural script.

## Features

- [ ] Can differentiate between standard fenced codeblocks and those with executable metadata
- [ ] Can create a set of steps by reading the entire file
- [ ] Can create an execution plan that identifies a platform that matches the users environment (Ubuntu, arch, macos, windows)
- [ ] Can capture errors during execution and track success steps. Allowing for commands state to be resumed or rewound or fast forwarded
- [ ] Can open a support issue based on current state and last error output to various targets (Slack, JIRA, GH Issues)
- [ ] Can identify new steps in an existing execution plan. Showing what procedural steps have been added or removed since last run.
- [ ] Provides caveats and hints based on locality of documentation to the codeblock
- [ ] Can match error output with known errata based on regex output matching

## Execution

So far I have built an extension for Kramdown that attempts to identify the meta information about a command but the way that Kramdown is written its very much a monkey patch to force new features in to it [Kramdown Parser Automation](https://github.com/ninjapanzer/kramdown-parser-automation). There is a lot of clobbering public constants to make this work :(.

I also tried to integrate this into markdown lint. As it will be essential to inform users if their code style is bad, thus breaking automation. But this is so heavily reliant on Kramdown that it almost makes it more effort to load a parser than it does to just write it from scratch.

I will continue to look for a new parser in another language. Maybe something in Go or Rust where the goal isn't directly conversion of markdown to markup which is the focus of Kramdown.
