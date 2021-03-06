---
layout: single
title: Download
sidebar: 
  nav: "docs"
---

Using our Maven repository (see [here](https://nexus.big-iot.org/content/repositories/releases/)), you can import the BIG IoT Lib into your software project.

For a really **simple start**, you can also clone our [**example project**](https://github.com/BIG-IoT/example-projects) and start from there to buid your first BIG IoT Provider or Consumer:
  * [Java Example Provider](https://github.com/BIG-IoT/example-projects/tree/master/java-example-provider)
  * [Java Example Consumer](https://github.com/BIG-IoT/example-projects/tree/master/java-example-consumer)
  * [Android Example Consumer](https://github.com/BIG-IoT/example-projects/tree/master/AndroidExampleConsumer)
  
These projects include everything you need to run them from the command line (`gradle run`) or include them into your IDEs (import as gradle projects).


## Using Maven or Gradle 

**Make sure you replace `x.y.z` with the correct version number (e.g. `0.10.M1`)!**

You'll find the most recent version number of the Big IoT Lib in the [Release Notes](/releaseNotes).

### Core Lib

The basic functionality of the BIG IoT Lib for providers and consumers. This lib is Java 7 compliant.

Import as Maven dependency: 
```xml
<dependency>
  <groupId>org.eclipse.bigiot.lib</groupId>
  <artifactId>bigiot-lib-core</artifactId>
  <version>x.y.z</version>
</dependency>
```

Import as Gradle dependency: 
```xml
compile 'org.eclipse.bigiot.lib:bigiot-lib-core:x.y.z'
```

### Advanced Lib

This lib extends the core lib and introduces lambda as well as asynchronous computation support via functional interfaces and *CompletableFuture*. It requires Java 8 or newer.

Import as Maven dependency: 
```xml
<dependency>
  <groupId>org.eclipse.bigiot.lib</groupId>
  <artifactId>bigiot-lib-advanced</artifactId>
  <version>x.y.z</version>
</dependency>
```

Import as Gradle dependency:  
```xml
compile 'org.eclipse.bigiot.lib:bigiot-lib-advanced:x.y.z'
```

### Embedded Spark

This extends the advanced lib with an implementation for deploying your offerings via an embedded Web server, called [Spark](http://sparkjava.com/). You can use this lib, if your offering should be served directly from your provider application, using an embedded Web server.

Import as Maven dependency: 
```xml
<dependency>
  <groupId>org.eclipse.bigiot.lib</groupId>
  <artifactId>bigiot-lib-embeddedspark</artifactId>
  <version>x.y.z</version>
</dependency>
```

Import as Gradle dependency:  
```xml
compile 'org.eclipse.bigiot.lib:bigiot-lib-embeddedspark:x.y.z'
```

### Android

This lib extends the *Core Lib* and adds Android-specific features. This is your choise of lib, if you want to develop an Android app as a consumer of BIG IoT platforms, or even if you develop a proxy provider based on the Android system.

Import as Maven dependency: 
```xml
<dependency>
  <groupId>org.eclipse.bigiot.lib</groupId>
  <artifactId>bigiot-lib-android</artifactId>
  <version>x.y.z</version>
</dependency>
```

Import as Gradle dependency:  
```xml
compile 'org.eclipse.bigiot.lib:bigiot-lib-android:x.y.z'
```

