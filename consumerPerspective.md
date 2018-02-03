---
layout: single
title: Consume IoT Offerings
sidebar: 
  nav: "docs"
---

Assuming, you want to create a new parking app for the city of Barcelona that is supposed to display parking data on a web dashboard for different places in the city. Using the BIG IoT Consumer Lib, it is very easy to make your service query the BIG IoT Marketplace relevant data sources and access them.

The Consumer Lib offers the following main functionalities:
1. Creation of a BIG IoT Consumer and authentication on the Marketplace
2. Discovering Offerings on the Marketplace according to a Query specification
3. Subscribing to Offerings of interest 
   * Based on known Offering ID
   * Based on discovered Offerings
4. Accessing Offerings based on a one-time request or in a continuous fashion

The complete java program of this example is available [here](https://github.com/BIG-IoT/example-projects/blob/master/more-java-examples/src/main/java/org/eclipse/bigiot/lib/examples/ExampleConsumer.java).


### 1. Creation of a BIG IoT Consumer and Authentication on the Marketplace

Before you can consume offerings available on the marketplace, you need to sign-up on the marketplace [here](https://market.big-iot.org/). Once you are logged in, you can create a new Organization for yourself (just click on “New Organization”). In a next step, you can create a new Consumer instance (click on “MyConsumers” and then “+Consumer”). After you created a new Consumer, you can copy the unique Consumer ID and Secret into your program in order to associate your Consumer application with the newly created Consumer instance on the marketplace.

```java
String MARKETPLACE_URL	= "https://market.big-iot.org";
String CONSUMER_ID 	= "TestOrganization-TestConsumer";
String CONSUMER_SECRET 	= "***************************";

Consumer consumer = Consumer.create(CONSUMER_ID, CONSUMER_SECRET)
			    .authenticate(CONSUMER_SECRET);
```

With this in place, you can now create a new Consumer instance in your application by providing the Consumer ID and Marketplace URL. In a next step, you need to authenticate this on the marketplace using your Consumer Secret.

If you want to connect to multiple marketplaces, just repeat the previous steps and create another Consumer instance. 


### 2. Discovering Offerings on the Marketplace according to a Query specification

Now, that you are authenticated at the marketplace, you can search for relevant parking sensor data to feed into your service.  
#### 2.1 Based on an Offering Query created via the Marketplace Web portal

In case you have already created an Offering Query via the Marketplace Web poral, you can simply call the `discoverById()` method by using the Query ID, as shown here:

```java
  List<SubscribableOfferingDescription> offeringDescriptions = 
                consumer.discoverById("TestOrganization-TestConsumer-DemoParkingQuery").get();
```

In return, the Marketplace provides a list of all matching offerings. 

#### 2.2. Based on an Offering Query created in program code

Alternatively, you can also create your Offering Query from scratch in your appliation. The query gets constructed using a builder pattern which first, creates the empty OfferingQuery object that is completed with additional query filters, such as a specific search region, a desired accounting type, a maximum price, etc. The marketplace will later retrieve all matching offerings for this query. In this example, the query is quite simple however it can be more complex in other situations.

```java
OfferingQuery query = OfferingQuery.create("DemoParkingQuery")
                .withName("Demo Parking Query")
                .withCategory("urn:big-iot:ParkingSpaceCategory")
                .withTimePeriod(TimePeriod.create(new DateTime(1999, 1, 1, 0, 0, 0), new DateTime()))
                .inRegion(BoundingBox.create(Location.create(40.0, 8.0), Location.create(45.0, 12.0)))
		// .inCity("Barcelona")
                .addInputData(new RDFType("schema:longitude"), ValueType.NUMBER)
                .addInputData(new RDFType("schema:latitude"), ValueType.NUMBER)
                .addOutputData(new RDFType("schema:longitude"), ValueType.NUMBER)
                .addOutputData(new RDFType("schema:latitude"), ValueType.NUMBER)
                .addOutputData(new RDFType("datex:parkingSpaceStatus"), ValueType.TEXT)
                .withPricingModel(BigIotTypes.PricingModel.PER_ACCESS)
		.withMaxPrice(Euros.amount(0.5))
                .withLicenseType(LicenseType.CREATIVE_COMMONS);
```

When creating the Offering Query, you need to provide an ID (here "DemoParkingQuery") and a name (here "Demo Parking Query") for it. Also it is important to define a semantic type or category (here "urn:big-iot:ParkingSpaceCategory") in order for the marketplace to return only relevant types for your consumer application. 

***NOTE 1: A full list of already defined and supported semantic categories is available here. Via the Marketplace user interface, you can also create new categories during creation of an offering. Those ‘“proposed”’ types can then also be used in your code.

Offerings queries can use temporal (`withTimePeriod()`) and spatial (`withRegin()` or `inCity()`) filters that ensures that only relevant offerings are returned. Also the pricing model and a maximum price, as well as the desired license type can be defined. The Consumer Lib offers you Enum classes that you can consult to see, which other licenses or accounting types are available.

Finally, you can also define input and output parameters that you expect in the offerings. 


### Querying the Marketplace

To execute the query on the marketplace, the Consumer object provides multiple options.  
The dedicated method for this is *discover*, which has different signatures to take different programming preferences into account.

```java
CompletableFuture<List<SubscribableOfferingDescription>> discover(IOfferingQuery query);
void discover(IOfferingQuery query, DiscoverResponseHandler onSuccess, DiscoverResponseErrorHandler onFailure);
void discover(IOfferingQuery query, DiscoverResponseHandler onSuccess);
```

The first version uses a *CompletableFuture* as a return type, which is a promise on a list of *OfferingDescriptions*, which is part of the functional programming styles introduced in Java 8. 
The other two variants are using callback mechanisms. The following code shows how to discover offerings getting them as a CompletableFuture on the list of OfferingDescriptions:

```java
CompletableFuture<List<SubscribableOfferingDescription>> listFuture = consumer.discover(query);
List<SubscribableOfferingDescription> offerings = listFuture.get();
Offering offering = offerings.get(0).subscribe().get();
```

The discover call is actually non-blocking. So, you could do something in between, e.g. handing over the CompletableFuture object to your management thread. Or alternatively, you can directly receive the list by calling the get method. This call is blocking and will wait on the list of OfferingDescriptions. The motivation of using CompletableFuture here is, that you can come easily from an asynchronous behavior to a blocking behavior and further you can utilize reactive concepts if you want. For example by calling thenApply as a monad on the CompletableFuture object allows you to apply functions once the list of offering descriptions is received. 

```java
listFuture.thenApply(SubscribableOfferingDescription::showOfferingDescriptions);
```

Before you can utilize an offering, you have to subscribe to the OfferingDescription object. Subscription is done through the correspondent *subscribe* method which returns an Offering object. The offering object provides different access methods as described later.
You can alternatively also use callbacks for discovering offerings. Here is an example how to achieve that:

```java
query.discover((r,l) -> { 
  SubscribableOfferingDescription.showOfferingDescriptions(l) 
});
```

The callback function in this example again just prints the returned offering descriptions, however usually you would provide your offering selection logic here, that selects the appropriate offerings for your use case. The example utilizes the functional programming features introduced in Java 8. With lambdas you can express functionality without much boilerplate code. Alternatively every other instance of DiscoverResponseHandler is accepted by *discover*. 

As a side note: You can reuse your query object for subsequent queries. Only if you want to change something regarding the query you have to create a new *OfferingQuery* object.

### Using SelectionCriteria

By using the *SelectionCriteria* class, you can specify a rule which is used by the BIG IoT Lib to filter offerings. You can define the selection criteria based on you service logic needs, by creating a new SelectionCriteria instance. To create a new selection, you use the *OfferingSelector* class, which accepts an arbitrary amount of *SelectionCriteria* objects. In the example below, we discover a list of offerings and apply an *OfferingSelector*, that selects based on cheapest offerings that have the most permissive license.

```java
consumer.discover(query)
.thenApply((list) -> OfferingSelector
    	.create()
   	.cheapest()
    	.mostPermissive()
	.select(list));
```

### Accessing Offerings

Before we describe how to access an offering that was retrieved from the marketplace, it makes sense that you look at the different access concepts provided. The *IOffering* interface provides the following signatures for access:

```java
void accessOneTime(AccessParameters parameters, AccessResponseSuccessHandler onSuccess);  
void accessOneTime(AccessParameters parameters, AccessResponseSuccessHandler onSuccess, AccessResponseFailureHandler onFailure);   
CompletableFuture<AccessResponse> accessOneTime(AccessParameters parameters);
IAccessFeed accessContinuously(AccessParameters parameters, Duration lifetime, FeedNotificationSuccessHandler onSuccess, FeedNotificationFailureHandler onFailure);
```

To access offerings, we distinguish between two types: *one-time access* and *continuous access*. *One-time access* means that you execute an access request every time you want to get new data. *Continuous access* refers to the reception of data as a feed.
For one-time access, the Consumer Lib supports again different programming styles. You can either use callback functions for pure asynchronous access or you can use a CompletableFuture to do reactive programming or even having a blocking call. 

In either case, you have to provide an *AccessParameters* object for the access call. In includes the parameters, which will be passed on to the provider. Typically they are needed to filter the output or configure the access. 

Here is an example, how to access the parking offering we retrieved earlier:

```java
/* Create a hashmap to hold parameters for filtering access*/
AccessParameters accessParameters = AccessParameters.create()
.addRdfTypeValue(new RDFType("schema:longitude"), 12.3)
.addRdfTypeValue(new RDFType("schema:latitude"), 42.73);

/* Execute one time access and print the result */
CompletableFuture<AccessResponse> response = offering.accessOneTime(accessParameters);
response.thenAccept((r) -> log("One time Offering access: " + r.asJsonNode().size() + " elements received. "));

```

As you can see, accessing an offering can be that simple. We use the *accessOneTime* method and pass the parameters object that restricts the access to the specified longitude and latitude coordinates. Since we use *accessOneTime* returning a CompletableFuture, we can apply a function on the result. Here we simply output the response content to the console. Note that the response object is of the type JsonNode, which already includes the parsed response message and provides functionality for traversing the response.

### Continuous Access of Offerings

Since we want to show the returned parking data in real time , it would be even nicer if we could access the parking data continuously.
Here we describe how this can be done:

```java
Duration feedDuration = Duration.standardHours(1);
Duration feedInterval = Duration.standardSeconds(2);

AccessFeed accessFeed = offering
  .accessContinuous(
     	accessParameters, 
	feedDuration.getMillis(), 
	feedInterval.getMillis(), 
	(f,r)->log("Incoming feed data: "+ r.asJsonNode().size() + " elements received. "),
	(f,r)->log("Feed operation failed")
	);
```

You notice that the procedure is very similar to the access on a per-request base. We use the *accessContinuous* method of the OfferingQuery object which requires the *accessParameters* object, a duration and feed interval, a success callback and a failure callback in case something went wrong. *accessContinuous* creates a feed, which requires a lifecycle management. The *accessFeed* object has functionality for stopping (stop), resuming (resume) and getting the status of a feed subscription (status), which we don’t want to use now. 

If you want to stop accessing an offering, you can unsubscribe accordingly.

```java
offering.unsubscribe();
consumer.terminate();
```

Make sure to always call the terminate method of the consumer object before stopping your application in order to terminate any open network connections. 

### Automated mapping to POJO

The following example will show you, how you can access offerings and let the BIG IoT Lib automatically match the output parameters to a Parking POJO.

```java
CompletableFuture<AccessResponse> response = offering.accessOneTime(accessParameters);
//Mapping the response automatically to your POJO
List<ParkingResultAnnotated> parkingResult = response.get().map(MyParkingResultPojoAnnotated.class);
```

For the access request, you use the map method, which accepts an annotated POJO class. The lib will now map the response data from the parking provider to the POJO MyParkingResultAnnotated. 

```java
public class MyParkingResultAnnotated {
	public static class Coordinate {
		public double latitude;
		public double longitude;
	}
	@ResponseMappingType("schema:geoCoordinates")
	public MyParkingResultPojoAnnotated.Coordinate coordinate;
	@ResponseMappingType("datex:distanceFromParkingSpace")
	public double distance;
	@ResponseMappingType("datex:parkingSpaceStatus")
	public String status;	
}
```

In order to map semantic types to your POJO’s types, you can use the *ResponseMappingType* class, which is parameterized with the semantic type you want to map. In this case, we would map the complex type **geoCoordinates** from the Complex Parking Offering to the Coordinate class.  

Another option, instead of using an automated mapping approach is to do the mapping manually. You can see how this works in the next example (note that we use the non-annotated version of the ParkingResult).

```java
CompletableFuture<AccessResponse> response = offering.accessOneTime(accessParameters);
List parkingResult = response.get()
  .map(MyParkingResultPojo.class, OutputMapping
  .create()
  .addTypeMapping("schema:geoCoordinates", "coordinate")
  .addTypeMapping("datex:distanceFromParkingSpace", "distance")
  .addTypeMapping("datex:parkingSpaceStatus", "status"));
```

To provide the mapping manually, you use the *addTypeMapping* method, for each semantic type from the provider’s output data elements so that the lib can match it to your POJO.  

A third option is to provide your own mapping which means to cherry-pick the required fields from the access response. In the example, we map **latitude** from the complex type **geoCoordinates** to the field **coordinate** of our parking result POJO. Also, we map the field **distance** to the POJO field **meters**.

```java
CompletableFuture<AccessResponse> response = offering.accessOneTime(accessParameters);
List parkingResult3 = response.get().map(AlternativeParkingPojo.class, OutputMapping.create()
	.addNameMapping("geoCoordinates.latitude", "coordinates.latitude")
	.addNameMapping("geoCoordinates.longitude", "coordinates.longitude")
	.addNameMapping("distance", "meters"));
```


**That's it!** You have just learned how to use the BIG IoT Library as a data provider as well as a data consumer. 


