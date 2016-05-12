---
layout: post
title: AuthSeries Pt. 1 - Authentication Basics
categories: Development
excerpt: "Part 1 of the Rails Authentication Series - Cookies and Sessions"
comments: true
---

Welcome to **Part 1** of AuthSeries, a series dedicated to authentication in Rails. WOO! In today's post, I will go over a few programming concepts that are crucial in understanding authentication as a whole.

#### Our goal today:
1. Learn about cookies
2. Learn about sessions

### Cookies

Imagine going into Facebook. You type `www.facebook.com` into your browser and are prompted to login. You do so happily, as you cannot wait to procrastinate for a few hours by accidentally creeping on people and reading a myriad of mundane posts.

Facebook logs you in. You are redirected to your homepage, where you see some breaking news - John's dating WHO?! You click on the post in your newsfeed only to be prompted to login again. Ugh. Again? You quickly login again because your nosiness just *needs* to know. Facebook takes you to the post. You read it, tell yourself Megan is basic anyway so who cares, and decide you want to check out some other posts. You click on the link to your newsfeed and there it is again. A prompt to login.

My goodness, you just wanted to stalk people in peace, and now this? What the http is going on?!

**WHAT IS A COOKIE?**

Cookies are used to maintain some state between the client (your browser, e.g., Google Chrome) and the origin server (where you are sending your requests to be fulfilled - in our example's case, Facebook). 

**WHY THEY ARE USEFUL**

In the scenario above, every time you click on a different link or part of the Facebook website, you are sending a new request to Facebook and therefore, being redirected to login every time. This is because HTTP requests are built to be a "stateless protocol". This means that the server processes each request separately from one another - the "state" of the first request where you logged in, is only longer maintained when you request to go to another page. 

Thus, every time you ask your browser to load a different page in Facebook, Facebook will ask you to login again because it *doesn't* remember the fact that you already logged in during a previous request.

This is where cookies come in. To solve this problem, cookies are created and stored in your browser when you visit a website that uses cookies to maintain state. The cookie is essentially a small text file that contains information about you as a user, such as your user id, what ads you click on, and more. Cookies provide a way for a website to verify who a user is once, and maintain that state for the rest of your visit on that site, or until you logout. 

Ah, finally. Now you can creep happily ever after.

**99 PROBLEMS, AND A COOKIE IS ONE**

As useful as cookies are, they also pose a huge security concern. Because they are stored as plain text files in a user's browser, it is frighteningly easy to change or manipulate the content of a cookie. 

For example, let's say you visit Facebook again, login, and are prompted to your homepage. Your browser at this point has a cookie stored containing information about your Facebook login, which was used to authenticate you as a user. Yay! You no longer have to sign in every time you request a different page. 

But what if you changed your user id in your cookie to something else? You click on the developer's console in your browser, find the Facebook cookie, change the user id, and voila! Suddenly, you find yourself logged in as the user with the newly changed user id. If you can hack into someone else's account this easily, doesn't this mean that the same could potentially happen to you? Uh oh.

### Sessions
