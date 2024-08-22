---
layout: post
read_time: true
show_date: true
title: Installing IIIF to Omeka S (DRAFT)
description: How to get Omeka S and IIIF running in production
date: 2024-08-14
img: posts/installing-omekas-iiif.jpg
tags: [ omeka s, iiif, docker ]
author: Maarten Coonen
---

In our [previous post](./getting-started-with-iiif-omekas.html) we talked about a quick way to get IIIF and Omeka S
running locally with Docker. In this post we describe the steps to take when installing IIIF to your Omeka S production
environment. 

### Architecture
There are many ways to work with IIIF and Omeka S and there is a lot of free software available. We chose for an 
architecture in which all IIIF components are located within Omeka S. This has its pros and cons (more on
that in another blog post), but the main argument is because of simplicity in data management. Omeka S is already used 
in our library and our collection managers are familiar with its data entry and data management flows. Introducing an
additional "moving part" to the infrastructure would make things more complicated with little added value (at least for 
the scale we're using Omeka S on).

Our chosen architecture looks as follows:

![architecture 1](assets/img/posts/iiif-architecture-1-web.jpg)

![architecture 2](assets/img/posts/iiif-architecture-2-web.jpg)

This blog describes the actions to take to get this architecture installed on your Omeka S production environment.

**(TODO: ADD STEPS HERE)**