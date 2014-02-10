---
layout: post
title: Simplicity (in software design)
date: 2011-08-15 07:19
disqus_identifier: 8948448440
---

> A designer knows he has achieved perfection not when there is nothing left to add, but when there is nothing left to take away.
> <cite>Antoine de Saint-Exupery</cite>

Recently I have been rethinking a lot about the software that automates things here at FatBox, specifically in terms of its simplicity — or rather lack there-of these days. I feel that a lot of our data model has become too strict and too complex and it’s beginning to inhibit new innovation because we’ve become obsessed on controlling the little things and it’s starting to getting in the way of the big things.

This led to an undertaking this past week to put this rethinking into practice and change as much as possible in how our business process is implemented in code, without changing too much of our code base or disrupting anything. Summed up:

* Remove as much complexity from the existing implementations
* Write as little code as possible
* Zero impact to day-to-day operations

This started with a review of our sales, order and billing process and ended with generating brand new user stories for the core of how we handle new business.

The biggest change is that we’re decoupling the data that we use to bill for our services from the data we use to manage our services, and while this seems like a small or subtle change this one action is going to have profound impact on how complicated our data model has become.

With a plan in place we started a new branch in our repository and started to simplify. As the weekend drew to a close we had only needed to write about 1,000 new lines of code, of which about 65% were either tests or boilerplate, but the best part is that the entire process that was becoming bloated is now gone, replaced with new, and simple, code that will be easier to maintain and less restrictive moving forward.

> Simplicity is the ultimate sophistication.
> <cite>Leonardo da Vinci</cite>
