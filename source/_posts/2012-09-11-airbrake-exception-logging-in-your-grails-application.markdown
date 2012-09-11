---
layout: post
title: "Airbrake exception logging in your Grails application"
date: 2012-09-11 14:34
comments: true
categories: 
- Grails
---

Recently I worked on migrating the Airbrake plugin written by [Phuong LeCong](https://github.com/plecong/grails-airbrake) to Grails version 2.1 ([repo here](https://github.com/cavneb/airbrake-grails)). After completing the plugin, I attempted to submit it to the Grails plugin repo via the [developer mailing list](http://grails.1312388.n4.nabble.com/Permission-to-publish-plugin-td4634449.html).

[Jon Palmer]([@bostanio](http://twitter.com/bostanio)) brought to my attention that there is a Java library that is being used to accomplish this task. I had searched prior for a solution that would work with Grails, but I didn't find any solid instructions on how to integrate the Java Airbrake library into my Grails apps.

So here it is.

### Installation

The Airbrake Java library exists in the Maven repo ([http://mvnrepository.com/artifact/io.airbrake/airbrake-java/2.2.0](http://mvnrepository.com/artifact/io.airbrake/airbrake-java/2.2.0)) and can be installed by simple adding the following as a runtime dependency in your `BuildConfig.groovy` file:

```
dependencies {
    runtime 'io.airbrake:airbrake-java:2.2.0'
}
```

Next time your app compiles, the Airbrake library should be installed and ready to go.

### Integration

It's nearly as simple to integrate the Airbrake exception notification process into your Grails application. This is done by adding a log4j appender into your `Config.groovy` file. Make sure that you update the appender config with the correct project API key from Airbrake.

```
log4j = {
  // Example of changing the log pattern for the default console appender:
  appenders {
    appender name: 'airbrake', new airbrake.AirbrakeAppender (
      api_key: 'API_KEY', // replace with your Airbrake API key
      env: ((Environment.current == Environment.PRODUCTION) ? 'Production' : 'Development')
    )
    ...
```

After the appender is in place, you need to let your Grails app know to use it as a global appender. Add `'airbrake'` to your root `debug` node:

```
log4j = {
  // Example of changing the log pattern for the default console appender:
  appenders {
    appender new airbrake.AirbrakeAppender (
      name: 'airbrake', 
      api_key: 'API_KEY', // replace with your Airbrake API key
      env: ((Environment.current == Environment.PRODUCTION)  ? 'production' : 'development'),
      enabled: true
    )
    ...
  }

  root {
    debug 'stdout', 'airbrake' // This can be added to any log level, not only 'debug'
  }
}
```

Once this is all in place, you should be able to test it out.

### Testing

To test the Airbrake exception notification, an exception must be thrown in your application when running. A good way to do this (although not ideal) is to create a controller with an action that throws an exception. I added this action to an existing controller, which again isn't ideal.

Here's an example controller and action:

```
class AirbrakeTestController {

  def simulateError() {
    session.myval = "Something in the Session!"
    throw new Exception("TestException ${System.currentTimeMillis()}")
  }

}
```

Once this is created, run your application and visit [http://localhost:8080/airbrakeTest/simulateError](http://localhost:8080/airbrakeTest/simulateError). You should see a page similar to this:

<div style="padding: 20px; 
      background: white; 
      margin-top: 10px;
      -webkit-border-radius: 5px;
      -moz-border-radius: 5px;
      border-radius: 5px;">
  <img src="/images/posts/airbrake-grails-exception.png" style="display: block;"/>
  <em>As you can see, I placed the action in another controller</em>
</div>

Now log into Airbrake.io and you should see your exception listed.

<div style="padding: 20px; 
      background: #DADADA; 
      margin-top: 10px;
      -webkit-border-radius: 5px;
      -moz-border-radius: 5px;
      border-radius: 5px;">
  <img src="/images/posts/airbrake-airbrake-exception-list.png" style="display: block;"/>
  <img src="/images/posts/airbrake-airbrake-exception.png" style="display: block;"/>
</div>

The data provided in the backtrace is a little weak and unformatted, and you can't see anywhere the contents of the session. It does however notify you when an exception is thrown and performs the goodness that Airbrake provides.

### Afterthoughts

The plugin I had submitted to the Grails plugin repository added a few extra goodies to the Airbrake notifications, primarily enhanced backtraces and session/request data. This isn't available in the Airbrake java library mentioned above.

After submitting the request to the mailing list, I received a couple of questions to the validity of the plugin, and then... nothing. I replied with reasons it's worth it, but there was no acceptance in place. I guess I could have pushed it to brute force my plugin into the repository, but to what end? I have a strong background with [Ruby](http://coderberry.me/blog/categories/ruby/) and [Rails](http://shop.oreilly.com/product/9780596520717.do), and am an active member of the [Ruby community](http://utruby.org/). Not once have I felt that my contribution to their community has been rejected. I can't help but feel this from my experience with the Grails community.

That being said, I have worked with some of the core developers of Grails and they share my concerns. They have very little to do with it, and have little control to change things. The new changes at [http://beta.grails.org](http://beta.grails.org/plugins) should promote more interaction in regards to plugins, but who knows.

Here are my parting words:

**Don't take the time to write a plugin which is intended for public use until AFTER you get it approved from the Grails developer mailing list.**

