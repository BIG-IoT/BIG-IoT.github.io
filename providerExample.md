---
layout: single
title: Provide IoT Offerings
sidebar: 
  nav: "docs"
---

Let’s go through the steps of developing the example parking provider. Assuming, you have an IoT platform with a parking data service. Using the BIG IoT Provider Lib, you can advertise  your services via offerigns at the BIG IoT Marketplace. In order to achieve that, the Provider Lib has the following main functionalities:
* Registration of offerings
* Activation and deactivation of offerings
* Renewing offerings
* Providing access callbacks

**The Java code of this example available [here](https://github.com/BIG-IoT/example-projects/blob/master/more-java-examples/src/main/java/org/eclipse/bigiot/lib/examples/ExampleProvider.java).** 

Note: You can also clone/download a complete Java project for an example provider that you can build and run directly (using gradle) [here](https://github.com/BIG-IoT/example-projects/tree/master/java-example-provider).  


### Creation of a BIG IoT Provider and Authentication on the Marketplace

Before you register your parking data offering on the marketplace, you need to create your provider lifecycle object and authenticate at the marketplace. 

```java
ProviderSpark provider = ProviderSpark
	.create(
		/*your provider id*/, "https://market.big-iot.org", "localhost", 8080)
	.authenticate(/*your provider secret*/);

```

Big IoT offers multiple integration modes for your services. In this example we choose server side integration (integration mode 2), where the BIG Provider lib has to provide a gateway service. Using the ProviderSpark class is the recommended easy way to instantiate a provider with an embedded Spark webserver. How you connect your data services, you’ll learn below.
The provider object will be used for all subsequent interactions with the marketplace. 
Of course, there is also the option for a properties file based initialization of a provider.

```java
ProviderSpark provider = ProviderSpark
        		.create("example.properties")
        		.authenticate();
```

### Create an Offering

Now, you should create an offering. Offerings are incrementally built, starting with a fresh offering.

```java
RegisteredOffering offering = provider.createOfferingDescription("DemoPark")
```

You can omit and reorder parameters. However, your offering should comprise at least a name and a category.

```java
offering.withName("Demo Parking Offering")
offering.withCategory("urn:big-iot:ParkingSpaceCategory");
```

**NOTE 1: A full list of already defined and supported semantic categories is available [here](https://big-iot.github.io/categories/). Via the Marketplace user interface, you can also create new categories during creation of an offering. Those '"proposed"' types can then also be used in your code.**

**NOTE 2: Using the right semantic catgory for your offering is very important for consumers to find it. So, try to use existing and well established categories whenever there is a good fit. If not, propose a new category that is meaningful.**

Our offering expects latitude, longitude and range as search location for the input. It returns parking locations. We have to define the parameters, so that consumer can see, how the offering can be used. In this example, our offering has flat parameters. Each parameter has a RDF URI describing its semantical type and a value type specifying how it is encoded. Below, you learn how complex parameters are used.

**NOTE 3: New semantic types for input and output data can be directly created via the code. Just use the keyword '"proposed"' in front of your new type (e.g. '"proposed:parkingSpaceType"').**

**NOTE 4: Using well estabished semantic types your input and output data is very important for consumers to find the relevant offerings and to process the data correctly. Try to use existing and well established types whenever possible. If not, propose a new type that is meaningful.**

```java
offering
	.addInputData("longitude", "schema:longitude", ValueType.NUMBER)
	.addInputData("latitude", "schema:latitude", ValueType.NUMBER)
	.addInputData("radius", "schema:geoRadius", ValueType.NUMBER)
	.addOutputData("longitude", "schema:longitude", ValueType.NUMBER)
	.addOutputData("latitude", "schema:latitude", ValueType.NUMBER)
	.addOutputData("dist", "datex:distanceFromParkingSpace", ValueType.NUMBER)
	.addOutputData("status", "datex:parkingSpaceStatus", ValueType.TEXT);
```

To complete our offering description we should add more metadata. BIG IoT also supports the monetarization of offerings. Therefore, we add price related information plus the licensing type.  We can also specify a region, for which we provide data. Since our parking spaces are located in Barcelona, we define a geo body comprising roughly the city limits of Barcelona.

```java
offering
	.inRegion(BoundingBox
		.create(Location.create(41.361, 2.082), Location.create(41.45, 2.252)))
	.withPrice(Euros.amount(0.02))
	.withPricingModel(PricingModel.PER_ACCESS)
	.withLicenseType(LicenseType.CREATIVE_COMMONS);

```

In case an offering also offers historic data, the offering description can also include a time period for which the offered data is available. This information can be added as follows:

```java
offering.withTimePeriod(new DateTime(("2017-01-01T0:00Z"), DataTime.now());
```

or

```java
	offering.withTimePeriod(TimePeriod.create(24134020384, 0L));
```

Here the first parameter denotes from which point in time the data are available. The second parameter defines until when. If the data are live data, the latter can also be set in the future, or set to 0, to denote that these are live data. 
Above, we said, we want to create a server side integrated offering in integration mode 2. Therefore, we have to implement an adapter. 

```java
offering.withAccessRequestHandler(new AccessRequestHandler() {
						
@Override
public BigIotHttpResponse 
	processRequestHandler(OfferingDescription oferingDescription, 
		Map<String, Object> input, String subscriptionId, String consumerInfo){

			String jsonArray = /*Get the data from somewhere*/;
			return BigIotHttpResponse.okay().withBody(jsonArray);	
});
```

The interface function gets an `OfferingDescription` in the corresponding field, a map for the request parameters plus strings for subscription id and consumer info. In this example, we assume that we read the parking data from somewhere, for example by a web service call to the orginal web service. The results are returned as body of `BigIotHttpResponse`. Please keep in mind that responses have to be JSON arrays.
`AccessRequestHandler` is a functional interface. If you prefer, you can provide the adapter also as a lambda function.
Alternatively, providers can also assign an `AccessRequestHandler` to a list of endpoints and use this during the offering registration. 

```java
Endpoints endpoints = Endpoints
			.create(offeringDescription)
                 .withAccessRequestHandler(new AccessRequestHandler() {/*..*/});
	
  RegisteredOffering offering = provider.register(offeringDescription, endpoints);
```

This allows a provider to reuse the offeringDescription object and register offerings on different endpoints.

### Registration of Offerings
Now, that you created the offering description, you probably want to register it at the Marketplace, so that other developers can find it. Use the register method for that.

```java
RegisteredOffering offering = offeringDescription.register();
```

If you want to remove your offering from the marketplace, just call the deactivate method on the offering.

```java
offering.deregister();
```

**Congratulations!** You have created your first offering on the BIG IoT Marketplace. Continue with the [consumer](../consumerExample) example**