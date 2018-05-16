---
layout: single
title: Startup Guide
sidebar: 
  nav: "docs"
---

# Startup Guide and Example Code

## Step-by-step instructions - to get started  

- For a BIG IoT [**Consumer application** (Java)](/StartUp-Guide-Consumer.md)
- For a BIG IoT [**Provider application** (Java)](/StartUp-Guide-Provider.md)
- For a BIG IoT [**Android Consumer application** (Java)](/StartUp-Guide-AndroidConsumer.md)
- For a BIG IoT [**Node.js Consumer application** (JavaScript)](https://github.com/flowhub/bigiot-js/blob/master/example/consumer.js)

## The Basics

**Java Example project for [download](/download.md):** This includes both, a BIG IoT Data Provider and Data Consumer project in Java, including all build files to build and run it direclty (`gradlew run`) or import it into your IDE. 

### How to develop a BIG IoT Consumer?

- **Simple Consumer example for accessing a known Offering** (full example code is available [here](https://github.com/BIG-IoT/example-projects/blob/master/more-java-examples/src/main/java/org/eclipse/bigiot/lib/examples/ExampleConsumerSubscriptionById.java)):
```java
// Initialize Consumer with Consumer ID and marketplace URL
Consumer consumer = new Consumer("Your Consumer ID - get it from Marketplace", "https://market.big-iot.org")
       .authenticate("Your Consumer SECRET - get it from Marketplace");

// Subscribe to Offering by OfferingId
Offering offering = consumer.subscribeByOfferingId("OfferingId - find it on Marketplace").get();

// Define Input Data as access parameters
AccessParameters accessParameters = AccessParameters.create();
       // e.g. .addRdfTypeValue("schema:latitude", 42.0).addRdfTypeValue("schema:longitude", 9.0);

// Access Offering one-time with Access Parameters (input data) --> response includes JSON results
AccessResponse response = offering.accessOneTime(accessParameters).get();
```
- **To get stated**, you can **use** our [**Java Example Consumer project**](https://github.com/BCX18ConnectedLife/big-iot/tree/master/java-template-consumer) **as template** for your own project. It is part of the [GitHub example project](https://github.com/BCX18ConnectedLife/big-iot) mentioned above and **contains everything** to get started!
- A _detailed_ Java **developer tutorial** for a Consumer can be found [here](https://big-iot.github.io/consumerPerspective/).

### How to develop a BIG IoT Provider?

- **Simple Provider example - register a new Offering on the Marketplace** (full example code is available [here](https://github.com/BIG-IoT/example-projects/blob/master/more-java-examples/src/main/java/org/eclipse/bigiot/lib/examples/ExampleProviderWithMarketplaceOfferingDescription.java)):
```java
// Initialize provider with provider id and Marketplace URI
ProviderSpark provider = ProviderSpark.create("Your Provider ID - get it from Marketplace", 
                                              "https://market.big-iot.org", "IP address of your node", 6789)
        .authenticate("Your Consumer SECRET - get it from Marketplace");

// Create an Offering Description based on a stored descripton on the Marketplace
RegistrableOfferingDescription offeringDescription = 
        provider.createOfferingDescriptionFromOfferingId("OfferingId - get it from Marketplace");

// Define an Endpoint for your Offering
Endpoints endpoints = Endpoints.create(offeringDescription)
        // provide an AccessRequestHandler - it is called each time a Consmer accesses your offering
        .withAccessRequestHandler(accessCallback);

// Register the offering - from now on it will be discoverable, subscribable and accessible to consumers
provider.register(offeringDescription, endpoints);
```
- **To get stated**, you can **use** our [**Java Example Provider project**](https://github.com/BCX18ConnectedLife/big-iot/tree/master/java-template-provider) **as template** for your own project. It is part of the [GitHub example project](https://github.com/BCX18ConnectedLife/big-iot) mentioned above and **contains everything** to get started!
- A _detailed_ Java **developer tutorial** for a Provider can be found [here](https://big-iot.github.io/providerPerspective/).

Further Java example applications for consumers and providers are available [here](https://github.com/BIG-IoT/example-projects/tree/master/more-java-examples/src/main/java/org/eclipse/bigiot/lib/examples).

### Download SDK

Information on how to [download](https://big-iot.github.io/download/) and use our SDK directly in your porject is provided [here](https://big-iot.github.io/download/). 


## Background Information

Background information and details on the BIG IoT architecture and vision are available [here](https://big-iot.github.io/tutorial/).
