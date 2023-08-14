+++
title = "Choice of Technology at Quiqup"
date = "2022-03-18T13:00:46+04:00"
author = "Danny Hawkins"
authorTwitter = "dannyhawkins"
categories = ["architecture"]
tags = ["architecture", "nextjs", "elixir", "eventsourcing"]
showFullContent = false
readingTime = true
hideComments = false
+++
## Introduction
Throughout the lifetime of Quiqup we been evolving our ability and understanding of good engineering. We've made a fair few mistakes along the way, but all of it has led to where we are now, and making the mistakes we have only make use more confident in our current decisions. I wanted to write a brief summary of our journey so far.

## Early Days
In the early days of Quiqup we built our API with Ruby on Rails, at the time (and still now) it’s a very popular option for rapid development, using convention over configuration allowing developers to produce consistent code and leverage the framework elements included to work fast.

We were building many features one after another, in many different areas of the business, and Ruby on Rails served us well as an API platform.

For frontend we started with Ionic (web on mobile technology), followed by native IoS for our for customer facing product (at that time our proposition was consumer facing). Internally for our dispatch tool we were using Angular1.3

## Pivot to B2B
When we realised how important our place was in the business to business world, providing logistics as a service, there was a big push to build out client facing tools for businesses. We started with a google form, to a react based form, to a fully fledged business focused offering called Quiqdash. This was all built out using React as we started to move away from Angular 1.3, primarily because React was considered to be much better for large apps composed of smaller components.

From the backend we were still with Ruby on Rails primarily, although we had our first Elixir service which was RealtimeEx, whose responsibility it was to receive updates from the CoreAPI via Redis pub/sub and push those updates out to connected web socket clients, primarily Quiqdash, and Dispatch. Elixir was selected for its erlang roots, erlang itself having a great reputation for web socket connections, distributed compute and fault tolerant, and Elixir being created to provide “something better” for Rubyists.

## Time for Change
Working with the CoreAPI was starting to become very difficult, we did have parts of the CoreAPI that were written in a much cleaner way since we realised the default “rails” way had some approaches that we now consider to be bad practice including but not limited to, too much logic in models, using a lot of includes for code separation only, model callbacks, and no default “service” layer. But the majority of the code was very tightly coupled and had a lot of “magic” such as model callbacks that would send a realtime broadcast whenever a model changed.

Although we could have stayed with Rails, we had some exposure to both Typescript and Elixir and considered both as better options than to forward develop using Ruby on Rails, primarily because of the NestJS (Typescript) framework and the Phoenix (Elixir) framework.

![Hello Nest](/posts/hello-nest.png)

We took the decision to start to adopt NestJS as a general use language in 2019, although there was a strong preference from the backend team to use Elixir / Phoenix, there were a few things we considered that made our final choice to be NestJS:

* Elixir / Phoenix issues
  * Still relatively new
  * Devs will need to learn functional programming
  * Availability of talent
* NestJS benefits
  * Typescript is then used across the full stack
  * Modular approach with dependency injection from the start
  * Developer experience is amazing in vscode because of typescript support

Once we started to use NestJS for a few projects, like our restrictions gRPC service, and our Invoicer integration, we had really good results, although some parts of using Typescript / Node.JS can be a developer hell (like tracing an async exception). Increased productivity, concurrency, code reusability and modularity all meant that the overall NestJS had been a good choice.

## Event Sourcing
For a long time we had been wanting to move to a event sourcing for a large amount of our business logic, it makes a lot of sense for Quiqup as we are dealing with real world events all the time, so mapping them 1-to-1 to software would make it easy to create a common language across business, product and technology.

Given some of the problems from our past, the idea of command query responsibility segregation (CQRS) and the flexibility of being able to create projections (read models) as and when needed was also very appealing.

We had been researching and learning about Event Sourcing for some time before we decided to bring it into a project. Whilst the benefits were clear, so were the complexities of adopting this approach. We had a proof of concept that was created to test a basic chain of custody system when we were working with one of our first big e-commerce clients. This was before we had adopted NestJS as general use. We built a PoC in Elixir using a framework called Commanded. This framework had been built for event sourcing, and has some very elegant methods to solve some key problems. One such example is that it keeps aggregates in running processes on the erlang VM called GenServers. This meant that when you want to work with an aggregate, you do not need to always load the stream of events.

Although the PoC was not developed further as we stopped working with the client, the experience of working with Commanded and building our first real event sourced service only strengthened our resolve that event sourcing made sense for Quiqup.

## Courier Incentives Initiative
Towards the end of 2020 we had started on an initiative to break courier compensation into two parts, a flat hourly fee and an order based bonus scheme, the system would also be able to track penalties issues to drivers and would take over the financial responsibility of calculating courier / agency pay, and therefore costs.

This project was an ideal candidate to move to an event sourced system, it could be separate enough from the CoreAPI to stand by itself, it fit well into one of our identified domains (Fleet Management) and it was relatively small in scope. We did some initial technical qualification on using NestJS and NestJS CQRS module to build our system using our general use language / framework and the knowledge we had of event sourcing.

The project took some time to get off the ground as we had to bring the team into the world of event sourcing, we were also learning a lot how to design the system using event modelling. During development we hit a few problems that sucked a lot of dev time to solve, and when we were solving these problems (specifically around event sourcing) we found ourselves looking toward the Elixir Commanded framework to see how they solved it. At some point along this journey it became apparent we would need a larger team to “roll our own” equivalent of what already existed in Commanded, and we would then have to maintain and develop that framework, alongside future work. It was at this point that we considered if we should just be using Elixir, Phoenix and commanded.

One of our lead engineers created a PoC for one module of our courier incentive system using Elixir, the result was great, he was able to port a lot of logic very quickly because commands, events and projections are very portable by nature.

It became apparent that using Elixir for courier incentive we could end up with a stable solution much faster, so we re-created the core areas of the service over 2 weeks.

During this time two of our typescript engineers who had been working with NestJS also got involved with the conversation, and we were very surprised to find that they were able to pick up and start moving with Elixir very fast, with little guidance.

## Elixir, Phoenix and Commanded

![Hello Commanded](/posts/hello-commanded.png)

Now that we are running multiple services in Elixir and we have explored options away from Elixir we are confident at this point in the evolution of Quiqup, Elixir, Phoenix and Commanded make a lot of sense for our core services. We also learned that porting an event sourced system is less complicated than alternatives, largely due to the responsibility of code in a command, event, aggregate and projection are clearly defined.

Making a choice to use and language or framework becomes a lot easier when you know that porting to other options is simple.

The productivity and stability gains we have experienced have been very pleasing, and using the excellent ElixirLS VSCode extension has resulted in a developer experience compatible to that of typescript (with WAY better stack traces) it’s for that reason that we are focused on using Elixir for immediate future at least.