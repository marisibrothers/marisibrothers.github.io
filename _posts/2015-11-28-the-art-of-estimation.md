---
layout: post
title: The Art of Estimation
date: '2015-11-28T18:00:00.000Z'
author: luciano_marisi
tags: 
permalink: /2015/11/the-art-of-estimation.html
---

**TL;DR - Estimating software is hard, to be reasonably accurate it is best to split the requirements in very small stories, and naturally experience does help**

Estimation in the software environment is more an art than a science. In most cases anything that is estimated takes longer than originally planned. There are generally two outcomes; the deadline is moved and the budget is increased or the scope is reduced. Software estimation is a huge subject, so I only intend to explain some ideas that have helped me throughout the projects I've been involved in.

Naturally, somebody that has already done a similar task before will be able to provide  a better estimate. For example, specifically to iOS/Mac development, the setting up of a CoreData stack will usually be more accurately estimated by a senior developer as opposed to a junior developer. However, by being meticulous about estimates a reasonable level of accuracy can be achieved.

## Importance of requirements

Before estimating software, it is good too start with very high-level epic stories. An epic story is a high-level requirement, for example, in a recipe app, one of the epics would be to be able to show the details of a recipe (i.e. ingredients, steps, summary and so forth). When defining this it's important to focus on the most valuable ones firsts. This is easier said that done as Oscar Wilde said _"Nowadays people know the price of everything and the value of nothing.”_. One you have your epics, then you can work your way down to individuals stories and the small tasks necessary to complete the story.  

Requirements and estimation are tightly linked, naturally if the requirements change so will the estimates. Growing requirements lead to scope-creep and will make your project fail if they are not controlled. Hence, unstable requirements lead to poor estimates. This tends to occur when the value of what we are building is not clear.

## Keep estimations small

The bigger the task the harder it's to estimate. The reason for this is that variables, edge-cases and requirements are too many to get a real grasp of. As a rule of thumb if a task is estimated to take more that two days, it should be broken down to a simpler form. As an oversimplified example, it is harder to estimate a task that is:

- Load the recipe data onto the app

Than these tasks:

- Define and document the specification of the data
- Create a model representing a recipe
- Define relevant images for a 	recipe
- Parse JSON (or relevant data type) and map it to the create model
- Write unit tests to prove that the JSON has been parsed correctly into the models

By keeping estimates small and concise, it's easy to compare the result of how long something was estimated to take and how long it took. This is key to fine tune the ability to estimate small tasks in the future.

## Dealing with the unknown

Sometimes we need to estimate something that we have never done before. You may think that if the implementation of a task is unknown we should give a very high estimate. However, _"Work expands so as to fill the time available for its completion"_ as defined by Parkinson's law. Among other things, this suggests that overestimating is not the solution since the task will just take longer without necessarily adding more value.

In this case, it's best not to give an estimate as the time you think it will take will be as good as throwing a dice. Therefore, for these stories it's best to spend a small amount doing a [spike](https://en.wikipedia.org/wiki/Spike_%28software_development%29). A spike is about spending a small amount of time to investigate the unknown before estimating the story. For example, before implementing a new SDK or library, it can be used in a sample project for one or two days to know how it works. This will give you much more information to later provide a more accurate estimate.

## Conclusion

Requirements will change over time as discussed [previously](http://www.marisibrothers.com/2015/11/designing-reusable-components-part-1.html). Hence, it's important to understand and plan for high-level estimates to change as the project evolves. Naturally, the allowance for changes will depend on your time and budget constraints.

Underestimating leads to low team moral, increased costs, more bugs and ultimately projects failing. On the other hand, overestimating will mean that the relationship between the value of a project and the cost will be diminished as a lot of time will be spent procrastinating.