---
layout: single
title: More Examples
sidebar: 
  nav: "docs"
---
Besides the explained core functionalities of registering provider and consumer, the BIG IoT Lib also supports more sophisticated features, which will be explained here in more details.

###	Complex types – Provider perspective

Our example above foresees only flat parameter sets. However, in reality, you find nested types very often. For example, instead of using parameters longitude and latitude to model a position, one could use also the semantical type `schema:geoMidpoint` with latitude and longitude as subparameters. Taking the parking example from above, an offering could look as follows: 

```java
RegisteredOffering offering = provider
	.createOfferingDescription("ComplexParkingSpotProvider")
	.addInputData("radius", "schema:geoRadius", ValueType.NUMBER)
	.addInputData("center", "schema:geoMidpoint",
		ObjectParameter.create()
			.addMember("latitude","schema:latitude",
				NumberParameter.create(-90, 90),true)
			.addMember("longitude","schema:longitude",
				NumberParameter.create(-180, 180),true),
				true)
	.addOutputData("location", "schema:geoMidpoint",
		ObjectParameter.create()
		.addMember("latitude", "schema:latitude", 
			NumberParameter.create(-90.0, 90.0),true)
		.addMember("longitude", "schema:longitude", 
			NumberParameter.create(-180.0, 180.0),true),
		true)
	.addOutputData("distance", "datex:distanceFromParkingSpace", ValueType.NUMBER)
	.addOutputData("status", "datex:parkingSpaceStatus", ValueType.TEXT)
	//...
```

Consumer accessing this offers create accordingly the access parameters.

```java
AccessParameters accessParameters = AccessParameters
	.create()
	.addNameValue("radius", 500)
	.addNameValue("center", AccessParameters
		.create()
		.addNameValue("latitude", 48.10)
		.addNameValue("longitude", 11.23));
```

Accessing the offering is the same as in the example above 

```java
offering.accessOneTime(accessParameters);
```

###	Automated mapping to POJO

When we receive data, we have different options for processing the data. AccessResponse gives access to a JSON container, with which we can traverse the response. But maybe it would be more convenient if we could map it directly to a POJO. 


```java
CompletableFuture<AccessResponse> response = offering.accessOneTime(accessParameters);
//Mapping the response automatically to your POJO
List<ParkingResultAnnotated> parkingResult = response.get()
						.map(ParkingResultAnnotated.class);
```

For the access request, you use the map method, which accepts an annotated POJO class. The lib will now map the response data from the parking provider to the POJO `ParkingResultAnnotated`. 

```java
public class ParkingResultAnnotated {
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

In order to map semantic types to your POJO’s types, you use the `ResponseMappingType` class, which is parameterized with the semantic type you want to map. In this case, we would map the complex type `geoCoordinates` from the Complex Parking Offering to the `Coordinate` class. 
Another option, instead of using an automated mapping approach is, to do the mapping manually. You can see how this works in the next example (note that we use the unannotated version of the `ParkingResult`).

```java
CompletableFuture<AccessResponse> response = offering.accessOneTime(accessParameters);
List parkingResult = response.get()
		.map(ParkingResult.class, OutputMapping
			.create()
			.addTypeMapping("schema:geoCoordinates", "coordinate")
			.addTypeMapping("datex:distanceFromParkingSpace", "distance")
			.addTypeMapping("datex:parkingSpaceStatus", "status"));
```

To provide the mapping manually, you use the `addTypeMapping` method, for each semantic type from the provider’s output data elements so that the lib can match it to your POJO.
A third option is, to provide your own mapping which means to cherry-pick the required fields from the access response. In the example, we map latitude from the complex type `geoCoordinates` to the field coordinate of the POJO. Also, we map the field distance to the POJO field `meters`.

```java
CompletableFuture<AccessResponse> response = offering.accessOneTime(accessParameters);
List parkingResult = response.get()
		.map(AlternativeParkingPojo.class, OutputMapping
			.create()
			.addNameMapping("geoCoordinates.latitude", "coordinates.latitude")
			.addNameMapping("geoCoordinates.longitude","coordinates.longitude")
			.addNameMapping("distance", "meters"));
```

###	Data stream supports for HTTP/HTTPS based access protocols

In the context of IoT, data are often sensed, distributed and processed in form of streams. I.e. new sensor observations are added to a stream as they are measured or obtained, allowing consumers to process all the sensor observations in stream manner (one after the other). 
Without data stream support on the provider end, consumers have to poll with high frequency in order to ensure that they are not missing any sensor observations. To overcome this shortcoming, special streaming protocols (e.g. WebSockets) or message brokers (e.g. MQTT or AMQP) have been developed. 
However, for constrained consumers that want to access data via a simple HTTP request, these solutions are not suitable. 
To overcome this limitation, the BIG IoT Provider Lib supports data streaming for HTTP/HTTPS based access protocols on A1. 
We describe here how the data stream features can be used by Providers and Consumers:

1. Instead of defining an `AccessRequestHandler` function during the offering description creation and registration, a Provider can simple write newly arriving data records – whenever they are obtained – to a data stream queue using the following command:

```java
offering.queue(jsonObject);
```

NOTE: It is assumed here that the `jsonObject` passed to the queue method contains a new output data record that is compliant with the offering description’s output data specification. 

2. A consumer who wants to access the data in a stream fashion will simply use a continuous `AccessFeed` as follows: 

```java
AccessFeed accessFeed = offering
					.accessContinuous(accessParameters,
						Duration.standardSeconds(60), 
						(o,r) -> System.out.println("feed result:" + r), 
						(o,r) -> System.out.println("accessed failed"));
```

For stream based offerings, each periodic access request will return only the data records that have occurred since the last access. This way, consumers obtain each data record only once and do not need to handle duplications manually. 
NOTE: The first access request will return all the data records in the Provider queue. Providers can control the size of the streaming queue with the command:  `offering.flush()`. Upon this call, the queue will be emptied for all new sessions.
The Provider Lib takes care of the Consumer session management under the hood. So, for each new access feed, the Provider Lib creates a new streaming session for the respective Consumer. In subsequent access requests (part of the same `AccessFeed`), a Consumer will thus only obtain data records that have arrived/occurred on the Provider end since the last access. 
Since a Consumer uses the `AccessFeed` as normal, it is the Provider who decides if this features should be used. For data offerings with streaming type of data, this is a convenient and efficient way for Providers to offer such data, as data transmissions are automatically reduced to a minimum. 

### Using integration mode 3

Up till now, we have only considered that you run a BIG IoT Gateway Service, where you provide the callback code to access your IoT platforms. But what if you cannot start additional services in your environment? In this example, you learn how to register and access offerings that uses the BIG IoT integration mode 3, which is enables a completely Consumer Lib driven access to your IoT platform. To learn more about integration mode 3, please refer
to section 4.4.
Suppose, we want to add air quality information to our application and we want to integrate the public available Web API OpenAQ [https://openaq.org](https://openaq.org). In order to integrate the platform using integration mode 3, first register your offering:

```java
RegistrableOfferingDescription airQualityOffering = provider
		.createOfferingDescription("Open_AQ_AirQualityOffering")
		.withName("OpenAQ_AirQuality")
		.withCategory("bigiot:AirQuality")
		.inRegion(Region.city("Berlin"))
		.withPrice(Euros.amount(0.01))
		.withPricingModel(PricingModel.PER_ACCESS)
		.withLicenseType(LicenseType.CREATIVE_COMMONS) 
		// The following properties are different to other integration modes
		.addInputDataInQuery("coordinates", "schema:location", ValueType.TEXT)
		.addInputDataInQuery("radius", "schema:geoRadius", ValueType.TEXT)
		.addInputDataInQuery("from", "schema:date", ValueType.TEXT)
		.addInputDataInQuery("limit", "schema:value", ValueType.NUMBER)
		.asHttpGet()
		.acceptsJson()
		.producesJson()
		.onExternalEndpoint("https://api.openaq.org/v1/measurements");
```

To understand what is happening here. Since there is not BIG IoT Gateway Service on server side, the lib is directly accessing the original web API of the IoT platform. Therefore, every syntactical element has to be described. For example, when calling a service function of a Web API: parameters can be encoded in the query (`...?param1=…&param2=…`), in the path (`.../paramValue1/paramValue2`) or in the message body e.g. as JSON. Or, as a combination of all three. In this example, the parameters are added as query parameters in the URL (`addInputDataInQuery`). But you can also try: `addInputDataInPath`, `addInputDataInBody`. If your offering requires complex parameter encoding, maybe even with many static parts, you can check out also `addInputDataInTemplate`. 
The same add methods for input parameters exist, of course, also for output parameters. 
Now, we specify that we access the API through an `HTTP GET` request, using the `asHttpGet` method, followed by a specification of what content types shall be used. The endpoint of the API is added to the registration in the last step. Note, that the method `onExternalEndpoint` is specifically provided by the BIG IoT Lib for integration mode 3.
Now, everyone on the marketplace can use OpenAQ through BIG IoT, including our application. Our consumer application can retrieve the data through BIG IoT access. Here is an example for a consumer:

```java
/* Create a hashmap to hold parameters for filtering access*/
AccessParameters accessParameters = AccessParameters.create()
		.addNameValue("coordinates", location)
		.addNameValue("radius", "2500")
		.addNameValue("from", "2017-08-25")
		.addNameValue("limit", "10000");
/* Execute one time access and print the result */
AccessResponse response = offering.accessOneTime(accessParameters).get();

response.thenAccept((r) -> dashboard.displayAirQuality(r));
```