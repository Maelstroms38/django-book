---
title: Introduction
seoTitle: Django Development
seoDescription: Intro to Django
isFree: true

---

# Intro to Django

## What is Django?

Welcome to Intro to Django!

Django is a high-level Python web framework that enables rapid development of secure and maintainable websites. Built by experienced developers, Django takes care of much of the hassle of web development, so you can focus on writing your app without needing to reinvent the wheel. It is free and open source, has a thriving and active community, and great documentation.

## Goals of the Django Framework

### Complete
Django follows the "Batteries included" philosophy and provides almost everything developers might want to do "out of the box". Because everything you need is part of the one "product", it all works seamlessly together, follows consistent design principles, and has extensive and up-to-date documentation.

### Versatile
Django can be (and has been) used to build almost any type of website — from content management systems and wikis, through to social networks and news sites. It can work with any client-side framework, and can deliver content in almost any format (including HTML, RSS feeds, JSON, XML, etc). 

Internally, while it provides choices for almost any functionality you might want (e.g. several popular databases, templating engines, etc.), it can also be extended to use other components if needed.

### Secure
Django helps developers avoid many common security mistakes by providing a framework that has been engineered to "do the right things" to protect the website automatically. For example, Django provides a secure way to manage user accounts and passwords, avoiding common mistakes like putting session information in cookies where it is vulnerable (instead cookies just contain a key, and the actual data is stored in the database) or directly storing passwords rather than a password hash.

A password hash is a fixed-length value created by sending the password through a cryptographic hash function ( bycrypt!!!)  Django can check if an entered password is correct by running it through the hash function and comparing the output to the stored hash value. However due to the "one-way" nature of the function, even if a stored hash value is compromised it is hard for an attacker to work out the original password.

Django enables protection against many vulnerabilities by default, including SQL injection, cross-site scripting, cross-site request forgery and clickjacking.

### Scalable
Django uses a component-based “shared-nothing” architecture (each part of the architecture is independent of the others, and can hence be replaced or changed if needed). Having a clear separation between the different parts means that it can scale for increased traffic by adding hardware at any level: caching servers, database servers, or application servers. Some of the busiest sites have successfully scaled Django to meet their demands.

### Maintainable
Django code is written using design principles and patterns that encourage the creation of maintainable and reusable code. In particular, it makes use of the Don't Repeat Yourself (DRY) principle so there is no unnecessary duplication, reducing the amount of code. Django also promotes the grouping of related functionality into reusable "applications" and, at a lower level, groups related code into modules (along the lines of the Model View Controller (MVC) pattern).

### Portable
Django is written in Python, which runs on many platforms. That means that you are not tied to any particular server platform, and can run your applications on many flavours of Linux, Windows, and Mac OS X. Furthermore, Django is well-supported by many web hosting providers, who often provide specific infrastructure and documentation for hosting Django sites.

### What can Django do for me?

When considering Django at first, I found out a lot of features, like authention and permissions were built in. By seeing many common use cases for Python, developers are able to create more robust applications.

The preceding sections show the main features that you'll use in almost every web application: URL mapping, views, models and templates. Just a few of the other things provided by Django include:

- **Forms**: HTML Forms are used to collect user data for processing on the server. Django simplifies form creation, validation, and processing.

- **User authentication and authorization**: Django includes a robust user authentication and authorization system that has been built with security in mind. It requires no external libraries and is natively supported by the framework.

- **Caching**: Creating content dynamically is much more computationally intensive (and slow) than serving static content. Django provides flexible caching so that you can store all or part of a rendered page so that it doesn't get re-rendered except when necessary.

- **Administration site**: The Django adminstration site is included by default when you create an app using the basic skeleton. It makes it trivially easy to provide an admin page for site administrators to create, edit, and view any data models in your site.

- **Serializing data**: Django makes it easy to serialise and serve your data as JSON. This can be useful when creating a web service (a web site that purely serves data to be consumed by other applications or sites, and doesn't display anything itself), or when creating a website in which the client-side code handles all the rendering of data.

### What are we building?

We are building a miniature clone of the [Libby Library App](https://libbyapp.com).

Our app will feature the following functionality:
- Create a lending library for books and articles.
- Users can login or signup and browse the catalog.

Here are some other examples of online digital libraries:
https://archive.org/details/books
https://books.google.com/

Before getting started, here is another excellent tutorial to get familiar with Django:

[Django Local Library](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django) (mozilla.org)

## Python Installation

Before getting started, please [download](https://www.python.org/downloads/) a Python version of at least `3.6` and above. For full Python and Django version support, refer to this [document](https://docs.djangoproject.com/en/3.0/faq/install/#what-python-version-can-i-use-with-django).

## References

- [Install Python for Mac or Linux](https://www.codingforentrepreneurs.com/blog/install-django-on-mac-or-linux)
- [Install Python for Windows](https://www.codingforentrepreneurs.com/blog/install-python-django-on-windows)

Ready? Let's get started.