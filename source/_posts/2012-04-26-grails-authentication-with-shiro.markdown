---
layout: post
title: "Grails Authentication with Shiro"
date: 2012-04-26 18:03
comments: true
categories: 
- Grails
- Authentication
---

[Shiro](http://shiro.apache.org) is a security framework that is meant to make life easier. The goal of it is to allow the developer to perform *authentication, authorization, cryptography* and *session management*. After reading over the [10 minute tutorial](http://shiro.apache.org/10-minute-tutorial.html), I trust that it is an excellent option to use in my Grails application. It should be also noted that the [Grails.org](http://www.grails.org) website also uses Shiro for all authentication.

My goal for this article is to help people understand how to implement the Shiro plugin into their Grails application as simply as possible. I will also cover some common usages of the plugin for reference.

## A bit more about Shiro

I strongly suggest going through the [documentation](http://shiro.apache.org/documentation.html) for Shiro before continuing. If not, there are a couple of tidbits that will help you understand how it works.

First, a term that Shiro uses for a *User* is *Subject*. Anytime the *Subject* is referenced, it is referring to Shiro's version of a User class.

Second, there is always a *Subject* that represents the current user. The *Subject* can be handled prior to authentication. This means that if someone was to visit your site without any authentication, there is already an instanciated class that represents them. In Java, you can obtain the *Subject* with the following:

```java
Subject currentUser = SecurityUtils.getSubject();
```

To authenticate the user, you must create a `UserPasswordToken` and call `login` on the instanciated *Subject*. The success or failure of the login are returned via [exceptions](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/AuthenticationException.html). Here is an example of code that may be used when logging in a user.

```java
if ( !currentUser.isAuthenticated() ) {
    UsernamePasswordToken token = new UsernamePasswordToken("foo", "bar");
    token.setRememberMe(true); // Use remember me if desired - all built in!

    // Attempt to authenticate. If no exception thrown, the login was successful
    try {
      currentUser.login( token );
    
    // Username wasn't in the system
    } catch ( UnknownAccountException uae ) {
      
    // Password didn't match
    } catch ( IncorrectCredentialsException ice ) {
      
    // Account with username is locked
    } catch ( LockedAccountException lae ) {

    // Unexpected error      
    } catch ( AuthenticationException ae ) {

    }
}
```

Cool! Now we have a bit of knowledge to help us know what we are doing in our Grails implementation.

## Installation

Let's start by creating a new Grails application. I am using version 2.0.3. In the code samples to follow, anything that starts with a $ means that it is a shell command.

<pre><code>$ grails create-app shiro-example
$ cd shiro-example</code></pre>

The [Shiro Plugin](http://grails.org/plugin/shiro) can be installed by using the dependency in your `BuildConfig.groovy`. Even though the plugin page shows version 1.1.4, we will use the latest snapshot release:

{% codeblock grails-app/conf/BuildConfig.groovy lang:java %}
    plugins {
        ...
        runtime ":shiro:1.2.0-SNAPSHOT"
        ...
{% endcodeblock %}

Once you have the dependency setup in the `BuildConfig.groovy` file, you must run the app to ensure the plugin is installed. After it's running, quit the app and return to the console.

Let's kickstart our app now by using the `shiro-quick-start` command. I am using the optional `--prefix=[package]`. Make sure you have a dot at the end if you are using the prefix.

<pre><code>$ grails shiro-quick-start --prefix=com.example.
| Created file grails-app/domain/com/example/User.groovy
| Created file grails-app/domain/com/example/Role.groovy
| Created file grails-app/realms/com/example/DbRealm.groovy
| Created file grails-app/controllers/com/example/AuthController.groovy
| Created file grails-app/views/auth/login.gsp
| Created file grails-app/conf/com/example/SecurityFilters.groovy</code></pre>

Whoa! What just happened? As you can see, you now have created some files:

`domain/../User.groovy` is the *domain class* for your user.

`domain/../Role.groovy` is the *domain class* for your user roles.

`realms/../DbRealm.groovy` is the class Shiro uses to handle authentication and permissions.

`controllers/../AuthController.groovy` and `views/auth/login.gsp` are the user-facing controller and view meant to handle login.

`config/../SecurityFilters.groovy` adds a [beforeInterceptor](http://grails.org/doc/2.0.3/ref/Controllers/beforeInterceptor.html) to all url's. This blocks/redirects users from accessing pages they are not authorized to visit (based on their roles).

> **What is a realm?**<br/>
> Shiro can use different *realms* for authenticating a user. A *realm* is basically a resource Shiro should use to authenticate a user. Further explanation can be found [here](http://www.brucephillips.name/blog/index.cfm/2009/4/5/An-Introduction-to-Ki-formerly-JSecurity--A-Beginners--Tutorial-Part-2).

## Configuration

## Bootstrapping

Bootstrapping users and roles is similar to how it's done with the [Spring Security Core plugin](http://grails.org/plugin/spring-security-core). Let's create a couple of roles and users to go with them.

{% include_code BootStrap.groovy lang:java %}

On line 5, we include the Shiro Security Service. This is necessary for us to encode the password into a password hash for the user.

On lines 9-15, the admin and user roles are created.

On lines 17-21, the admin user is either loaded from the database or is created. Here is where we use the Shiro Security Service for encoding the password.

On lines 23-26, the roles *ADMIN_ROLE* and *USER_ROLE* are assigned to the admin user. Note that we add both roles to this user. This allows the admin user to act as a normal user as well. This may or may not be appropriate, depending on the application.

Lines 28-36 are similar to 23-26, but for the standard user role instead of the admin role.

## Kick the Tires

Now that we have the default users created, let's run the app and test to make sure the login works. Start up the application with the `run-app` command:

<pre><code>$ grails run-app</code></pre>

Once it's running, open up the browser and go to <a href="http://localhost:8080/shiro-example/" target="_blank">http://localhost:8080/shiro-example/</a>.

{% img /images/posts/shiro-example-1.png %}

Click on the link *com.example.AuthController* and login as the admin user (username: 'admin', password: 'password').

{% img /images/posts/shiro-example-2.png %}

If you successfully logged in, you will be redirected back to the first page. This doesn't help us very much. Let's create a controller that will serve as a home page for the app.

<pre><code>$ grails create-controller com.example.Home</code></pre>

Create an `index` action in the controller and an accompanying view.

{% codeblock grails-app/controllers/com/example/HomeController.groovy lang:java %}
    plugins {
        ...
        runtime ":shiro:1.2.0-SNAPSHOT"
        ...
{% endcodeblock %}

## Creating Users

The `shiro-quick-start` command did create the login page and authentication controller, but it didn't help us a bit on creating users. Good thing it's pretty easy to do. 

We could create a controller called *Users* to handle the signup process, however, I don't want to do that. The reason is because I would like to eventually have a *Users* controller that will perform all the CRUD operations for the users and would only be accessible by administrators. 

Instead, let's create a new controller for handling the signup process. Let's call it `Signup`.

<pre><code>$ grails create-controller com.example.Signup
| Created file grails-app/controllers/com/example/SignupController.groovy
| Created file grails-app/views/signup
| Created file test/unit/com/example/SignupControllerTests.groovy</code></pre>

Now create a view called `index.gsp`. I will do so by using the *touch* shell command.

<pre><code>$ touch grails-app/views/signup/index.gsp</code></pre>

## Keeping out the Riff-Raff!

## Tag Lib Reference

## Service Methods Reference

## FAQ
