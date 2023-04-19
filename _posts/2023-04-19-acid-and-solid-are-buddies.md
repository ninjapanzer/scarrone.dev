---
layout: post
title: ACID and SOLID are buddies
date: 2023-04-18 09:50 -0500
categories: SoftwareDevelopment
---


## Atomic and heterogeneous
Does Conway's Law imply that your development and deployment processes must be homogenous? How does IPC serve as a stepping stone towards healthy isolation?

No, and just like in poetry, good ideas tend to recur. The crux of the matter is that atomic processes are inherently more durable. When you think of atomic in the context of software, you might assume I'm referring to the S.O.L.I.D principles. However, I'm actually talking about how your build system should adhere to A.C.I.D properties: atomicity, consistency, isolation, and durability.

Much of what we gain from the Single Responsibility and Open/Closed principles align with the objectives of A.C.I.D. Let's take dependency management as an example. When building our platform, we might choose a language like Ruby, which uses Bundler. Bundler is excellent for loading and creating dependencies for our atomic project. But what happens when we have multiple atomic projects? Combining their dependency trees into one master tree and having them share it would raise concerns, right?

I can tell you from experience that this scenario is entirely possible, even if it doesn't seem like it initially. So, what's the next step? We already have a unit of encapsulation with Bundler that can be applied to each project member. As a result, we have micro-dependencies (our packages) and macro-dependencies (our projects). This is likely sufficient, and we've created an artifact that encapsulates its responsibilities. However, assuming that we are process-bound, we haven't genuinely addressed any issues – in fact, it might seem like we've made things worse. Since each artifact must be loaded in the same runtime, the encapsulation will be broken as all the dependencies will have to be collapsed again, setting off alarm bells.

We've achieved atomicity and consistency but fallen short on isolation, and as a result, we can't accurately measure durability. Our breakpoint is at the runtime level. To ensure isolation, we need to separate the runtime into individual processes. I won't jump straight to microservices, as they're not always necessary. Microservices are primarily concerned with encapsulating resources and scaling, rather than process isolation. Container platforms often follow a pattern of one process per container, but with sidecars, it's not always the case when you need telemetry, logs, and a running application.

In a process-bound world like Ruby, what can we do? Before resorting to microservices, we can simply run all our processes in the same container and leverage IPC techniques like IO.pipe and Socket to facilitate communication between processes. If we want to be language-agnostic and potentially polyglot, gRPC could be our next option.

By dividing our runtime among processes, each process can isolate its dependencies, and our communications remain network-local to the resources running them. We've achieved isolation; now, for durability, we need a supervisor to keep the process running. Depending on your target environment and language, maintaining a healthy running process is nearly trivial. If you're working with Docker within Docker, those hands could wash themselves.

We've accomplished A.C.I.D for our application, but how does this impact the relationship between development and deployment homogeneity? It becomes easier, except we can no longer rely on our dependency manager for the final result. We must build a different type of artifact, one that provides process-level composability. This is where our development and deployments begin to diverge. As deployments focus more on resources than runtimes, our build system should create composable artifacts that represent complete, startable processes. In development, we'll compose these processes to meet the needs of the features we're developing, while in production deployment, they may be distributed among multiple resource stacks.

Runtime composability and dependency awareness now constitute a new layer. At first, we may find that an entire service is an entire application. However, there may be times when we want to decompose a part of an application into a composable artifact. To run a specific application, you may now need to run two processes. This is a positive development, and your build system can create both artifacts as well as the run orchestration needed to boot and maintain them.

This approach allows for a situation where we need to share a component. Since we cannot guarantee that any two components will ever be hosted on the same resource or deployed in a way that can be shared, our developers can bundle shared processes as formal dependencies when necessary. Each application will own its own version of the dependent process. In a production environment, we may allocate resources for that component and allow services to share it.

We now have two levels with the same assumptions of isolation: one with duplicate runtimes based on a development configuration, and one with distributed runtimes based on resource allocation. Both can function effectively, as the resources and processes encapsulate the same requirements, and as long as they don't exist in an environment violating the Open/Closed principle, they can coexist across multiple versions without conflict.

Some languages have well-designed build systems that can be configured in this manner, while others do not. Your mileage may vary, but adopting this kind of system means that transitioning to a polyglot environment becomes more manageable. You'll need to define hard boundaries between configuration, service discovery, circuit breakers, and integration testing/mocking earlier in your development cycle. It's easier to build these elements into the core assumption than retrofit them later.

## Origin of this approach
I am currently working on deconstructing a monolith and exploring how we can incorporate a more repeatable, self-documenting process without making too many DevOps decisions, while still allowing DevOps the flexibility to make decisions later.

Some of the challenges here are related to the language and its reliance on flattened runtime dependencies. The inability to create a well-isolated artifact means dependencies always feel burdensome, and any version drift is extremely painful.

Moreover, our system is highly coupled. While it has modules, those modules don't always respect the boundaries of the frameworks they rely on or their sibling modules.

It's going to be a long journey, and we need a way to isolate not only the dependencies but also the domains we encapsulate, without making all problems DevOps problems. By that, I mean the infrastructure that keeps everything running, not the development processes needed to deliver the SDLC, which are the responsibility of the entire engineering organization, not just DevOps.
