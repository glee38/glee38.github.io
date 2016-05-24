---
layout: single
title: Rails Authentication Series - Intro
categories: Development
comments: true
draft: false
---
#### HEY, IS THAT YOU?

As I continue to venture deeper into the magical land of Rails, I thought it would be a good idea to write about the different methods of authentication available for developers. The concept of authentication itself is not very difficult (figuring out if a user is really who they say they are), but I've found that proper set up and configuration is crucial to laying the foundation for great authentication flow. 

<!--more-->

I am, of course, no expert, but here are some of the options that I have learned for implementing authentication thus far:

1. Roll your own authentication logic (not the best idea - I will explain why)
2. Use Omniauth (make it someone else's problem)
3. Devise (a Rails engine that "pimps" your authentication ride)
4. Devise & Omniauth together

In the proceeding posts, I will go over all four methods of authentication. At the end of this series, I hope that both you and I will have developed a firm grasp on Rails authentication and can build an authentication system that will have every hacker wondering why they even tried.