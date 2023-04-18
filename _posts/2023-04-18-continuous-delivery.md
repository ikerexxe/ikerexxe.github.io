---
layout:     post
title:      "Continuous Delivery"
date:       2023-04-18 12:00:00 +0100
categories: review
---

# Book information

* Title: [Continuous Delivery: Reliable Software Releases Through Build, Test, and Deployment Automation](https://www.goodreads.com/book/show/8686650-continuous-delivery)
* Author(s):  Humble Jez, Farley David
* ISBN: 978-0321601919
* Date of publication: August 2010

# Target audience

This book is aimed at anyone involved in the software delivery process (developers, testers, operations engineers).

# Brief summary

The book presents the foundations and principles of continuous delivery. It starts by setting a common ground on what is continuous delivery and the most common mistakes organization make. Then, it explaing the foundations, such as configuration management, continuous integration, and testing. Next, the authors present a high-level picture of a deployment pipeline and how to begin building it. Finally, the book explains some advanced topics like managing dependencies and a maturity level framework for the delivery process.

# Finding

The main objective is to explain the principles and technical practices needed to deliver software without pain and risk and in a timely manner. The way to do that is to automate all the delivery workflow and by tracking all the resources involved in a version control system.

Besides, the ideal deployment pipeline is defined, and a way to start building it is explained. The deployment ecosystem is also presented in general terms.

## Strengths

* The principles of software delivery are explained from a high perspective in chapter 1.
* The second part of the book contains a good overview of what a deployment pipeline should look like. Moreover, the figures help a lot in understanding it.
* The authors were ahead of their time when defining the commit stage. This is actualy being done in some git hosted environments, where the patches go through a series of CI checks before being merged.
* The advanced version control chapter gives a good explanation of branching strategies.

## Limitations

* Although some parts of the book were ahead of their time I think that in general the book is outdated. Several parts of the book focus on technologies that aren't in use nowadays (i.e. SVN, puppet).
* Not only that, but the progress in certain areas of the software delivery process have left some practices outdated: don’t check in on a broken build, never go home on a broken build, etc.
* I'd like to strees the fact that most of the explanations regarding version control system are outdated.
* For my taste too many references to their employer.
* The chapter about nonfunctional requirements isn't very helpful. I was expecting a clearer definition of how to deal with such scenarios

# Rating

## **3/5**
