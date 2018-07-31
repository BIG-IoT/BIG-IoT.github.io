---
layout: single
title: Consumer Examples
sidebar: 
  nav: "docs"
---

Now, let’s assume, you want to use the data of the just created offering with your new parking app. Using the BIG IoT Consumer Lib, it is very easy to make your service use the BIG IoT Marketplace to query relevant data sources and access them.
The Consumer Lib offers the following main functionalities:

* Discovering offerings at the Marketplace according to a search specification
* Subscribing to offerings
* Accessing offerings in per request or in a continuous fashion

**The Java code of this example available [here](https://github.com/BIG-IoT/example-projects/blob/master/java-example-consumer/src/main/java/org/bigiot/examples/ExampleConsumer.java).** 

### Create a consumer object
Creating the lifecycle object for consumer operation is very similar to the provider case, described above. That’s why we don’t want to repeat the explanation.
To get started with the Consumer Lib, you have to create an instance of the `Consumer` class. The constructor requires your specific consumer ID and a marketplace URI. 
To authenticate with the marketplace, you have to use the authenticate method provided by the ´consumer` object. This method requires the marketplace API key and a consumer certificate, which you both received when you registered on the BIG IoT Marketplace portal.

```java
Consumer consumer = Consumer
	.create(
		/*your provider id*/, "https://market.big-iot.org")
	.authenticate(/*your provider secret*/);
```

Like for the instantiation of providers, there is also the option for a properties file based initialization of a provider.

```java
Consumer consumer = Consumer
        		.create("example.properties")
        		.authenticate();
```

If you want to connect to a different marketplace, just create another `consumer` object using another marketplace URL. 

###	Query for offerings
Now, that you are authenticated at the marketplace, you can search for relevant parking sensor data to feed into your service. To do that, you query the marketplace using the `OfferingQuery` object. The query gets constructed using a builder pattern which first, creates the empty `OfferingQuery` object that is completed with additional query filters, such as a specific search region, a desired accounting type, a maximum price, etc. The marketplace will return all matching offerings for this query. In this example, we search for everything what is possible and therefore possibly missing other offerings which are not so similar. Of course, you can reorder and omit attributes. Removing search parameters widens the query.

```java
OfferingQuery query = OfferingQuery
	.create("DemoParkingQuery")
	.withName("Demo Parking Query")
	.withCategory("urn:big-iot:ParkingSpaceCategory")
	.withTimePeriod(
		TimePeriod.create(new DateTime("2018-01-01T0:00Z"), DateTime.now()))
	.inRegion(
		BoundingBox.create(Location.create(40.0,8.0),Location.create(45.0,12.0)))
	.addInputData("schema:longitude", ValueType.NUMBER)
	.addInputData("schema:latitude", ValueType.NUMBER)
	.addOutputData("schema:longitude", ValueType.NUMBER)
	.addOutputData("schema:latitude", ValueType.NUMBER)
	.addOutputData("datex:parkingSpaceStatus", ValueType.TEXT)
	.withPricingModel(BigIotTypes.PricingModel.PER_ACCESS)
	.withMaxPrice(Euros.amount(0.5))
	.withLicenseType(LicenseType.CREATIVE_COMMONS);
```

*NOTE 1: A full list of already defined and supported semantic categories is available [here](https://big-iot.github.io/categories/). Via the Marketplace user interface, you can also create new categories during creation of an offering. Those ‘“proposed”’ types can then also be used in your code.*

To execute the query on the marketplace, the `consumer` object provides multiple discover methods. They have different signatures taking different programming preferences into account.
The first variant uses a `CompletableFuture` as a return type, which is a promise on a list of `OfferingDescriptions` introduced in Java 8. The following code shows how to discover offerings getting them as a `CompletableFuture` on the list of `OfferingDescriptions`.

```java
CompletableFuture<List<SubscribableOfferingDescription>> listFuture = 
											consumer.discover(query);
/*do something in-between*/
List<SubscribableOfferingDescription> offerings = listFuture.get();
Offering offering = offerings.get(0).subscribe().get(); //just take the first
```

The discover call is non-blocking. So, you could do something in-between, e.g. handing over the `CompletableFuture` object to your management thread. Of course, you can directly wait for the result by calling the get method. The motivation of using `CompletableFuture` here is, that you can come easily from an asynchronous behavior to a blocking behavior and further you can utilize reactive concepts if you want. For example by calling `thenAppl`y as a monad on the `CompletableFuture` object allows you to apply functions once the list of offering descriptions is received. 

```java
listFuture.thenApply(SubscribableOfferingDescription::showOfferingDescriptions);
```

The discovery is just returning offering descriptions, before you can utilize the offering behind, you have to subscribe to them. This authorizes you und is a precondition if charging applies. Therefore a corresponding subscribe method exists, which returns an `Offering` object. The `Offering` object provides different access methods as described later.
You can also use callbacks for discovering offerings. Here is an example for this.

```java
query.discover((r,l) -> { 
				SubscribableOfferingDescription.showOfferingDescriptions(l) 
	});
```

The callback function in this example again just prints the returned offering descriptions, however usually you would provide your offering selection logic here, that selects the appropriate offerings for your use case. The example again utilizes the functional programming features introduced in Java 8. With lambdas you can express functionality without much boilerplate code. Alternatively every other instance of `DiscoverResponseHandler` is accepted by discover. 
As a side note: Of cause you can reuse your query object for subsequent queries. Only if you want to change something regarding the filtering you have to create a new 
`OfferingQuery` object.

### Select an offering
When your discovery finds multiple offerings, you might wonder, which one to choose. Maybe you performed a wide search in order to find as much hits as possible. But you probably have selecion criteria to identify the best result. The lib supports also the automatic application of multiple selection criteria on the result of a discovery. Start with the creator method of `OfferingSelector` and build your selection.

```java
consumer
	.discover(query)
	.thenApply((list) -> OfferingSelector
						.create()
						.cheapest()
						.mostPermissive()
						.select(list));
```

### Access offerings
Now, we want to read data from the offering. First, we have to specify with which parameter we want to access the offering.

```java
AccessParameters accessParameters = 
		AccessParameters
				.create()
				.addRdfTypeValue("schema:longitude"), 12.3)
				.addRdfTypeValue("schema:latitude"), 42.73);
```

As you see, values are assigned to RDF types. This is different to assigning them to names, e.g. used in the web service request. In BIG IoT we want to boost semantical modeling of offerings. We assume that using the semantical model to access offering leverages interoperability better. But if you like, there is also the `addNameValue` method.

To access offerings, we distinguish between two access schemes: one-time access and continuous access. One-time means that you execute an access request every time you want to get new data. Continuous refers to the reception of data as a feed.
For one-time access, the Consumer Lib supports again different programming styles. You can either use callback functions for pure asynchronous access or you can use a 
`CompletableFuture` to do reactive programming or even having a blocking call. In either case, you have to provide an `AccessParameters` object for the access call. 

```java
CompletableFuture<AccessResponse> response = offering.accessOneTime(accessParameters);
response.thenAccept((r) -> System.out::println);
```

We use the one-time access method and pass the parameters object that restricts the access to the specified longitude and latitude coordinates. Since we use `accessOneTime` returning a `CompletableFuture`, we can apply a function on the result. Here we forward the result to println. 
In order to process the response in our application logic, you can use

```java
response.asJsonNode();
```

It returns the parsed response as Jackson `JsonNode` with you can traverse the data. If you prefer the raw JSON body, use

```java
response.getBody();
```

### Continuous access of offerings
Maybe we want to show the returned parking data in real time on a dashboard. It would be even nicer if we could access the parking data continuously.

```java
AccessFeed accessFeed = offering
						.accessContinuous(
							accessParameters,
							Duration.standardSeconds(60), 
							(o,r)->System.out.println("feed result = " + r), 
							(o,r)->System.out.println("Ups, something failed"));
```

You probably noticed that the call is very similar to the access on a per request base. We use the `accessContinuously` method of the `OfferingQuery` object which requires the 
`accessParameters` object, a duration, a success callback and a failure callback in case something went wrong. The difference is that, when calling `accessCalling` we create a feed, which requires a lifecycle management. The accessFeed object brings functionality for stopping (`stop`), resuming (`resume`) and getting the status of a feed subscription (status), which we don’t want to use now.
If you want to stop accessing an offering, you can unsubscribe accordingly.

```java
offering.unsubscribe();
consumer.terminate();
```

Make sure to always call the terminate method of the consumer object before stopping your application in order to terminate any open network connections.
**That's it!** You have just learned how to use the BIG IoT Library as a data provider as well as a data consumer. If you like, continue with [more](../moreExamples) examples
