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

Enter sessions, a gift to all techkind. Unlike cookies, which are client-side (your browser) files that contain user information, sessions are server-side (such as Facebook) files that contain user information. Because sessions are stored server-side, a user can no longer easily manipulate their content like they could a cookie. 

Let's break down how they work and why sessions allow maintaining state to be more secure.

**WHAT ARE SESSIONS?**

We've discussed above that HTTP is a stateless protocol, with cookies providing a way to maintain some sort of state (such as the user being logged in, the contents of a user's shopping cart, etc.) between the client and the server. A session is essentially a type of cookie in that it also behaves like a hash, storing user pertinent information in a key referred to as a "session id". The session id usually contains a 32-character string, which is used to identify the hash.

The first time you log in to Facebook, a session id is created on the server side. This session id is used to identify and authenticate you during the time you are browsing their site. Thus, your "logged in" state will be maintained with Facebook until you either logout, or until the session expires (some websites have a timeframe in which a session will expire on its own if you have not explicitly logged out), whichever comes first.

**WHY THEY ARE USEFUL**

Now when you send a request to Facebook to go to another page or look at a different post, every cookie sent to Facebook's server will include your unique session id with which to authenticate you. Facebook will then send back the contents of your request for your browser to render, and will send back your session id with it. This exchange allows a state to be maintained between the client and server without having any revealing information passed through your cookies. Your session id variables can only be seen on the server side.

Let's use a real life example to illustrate what is going on.

**GIMME THAT SHACKSTACK**

After hours of successfully procrastinating on Facebook, you realize that all this hard work has made you pretty darn hungry. There's no food in the fridge, so you decide to go out to the nearest Shake Shack. 

This Shake Shack is so poppin' that they provide every customer a buzzer and receipt number that is used to identify you when your food is ready. You stand on line, order, pay with your credit card, sign the receipt, and the employee hands you a buzzer and an order number. Your number is 33.

![login_sessions](http://i67.tinypic.com/2h5o07s.jpg)

After 180 seconds of hunger-induced torture, your buzzer vibrates. Your heart skips a beat and you bring your buzzer and receipt up to the counter. An employee takes a look at the number on your receipt and matches it up to the receipt attached the glorious brown bag containing your order. You don't need to show your credit card, or sign the receipt with your signature again to verify that you are indeed the proud owner of the food baby to be. That was already validated when you first paid for your order, and the ticket number is all you need to prove who you are. You take your food and blast Hall & Oates "You make my dreams come true" on your way home. Well done.

In the example illustrated above, paying and signing your receipt is like logging in with your username and password. The receipt is like the session id that is created by Facebook when you first log in, and that receipt is what is used to identity you for the remainder of your visit. You never have to show your credit card or sign the receipt again - the number on the receipt (like a session id) is all you need for a back and forth exchange between yourself (the client) and the employee (the server). This way, you don't run the risk of identity theft by exposing your credit card numbers multiple times (like a cookie being passed back and forth between requests), and you can access your food with the number issued by Shake Shack.

**CONCLUSION**

As you can see, cookies and sessions are a critical part in maintaining state and allowing for great user experience during your interaction with an application. In the next post, we will discuss how we can use these newly learned concepts to roll our own authentication logic using Rails.
