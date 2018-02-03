---
layout: single
title: Provide IoT Offerings
sidebar: 
  nav: "docs"
---

Assuming, you have developed an IoT platform for parking data. Now you want to make this data public. How do you connect your platform to the BIG IoT Marketplace in order to provide this data to other developers so they can use your data in their applications?

Using the BIG IoT Provider Lib, you can manage your data on the BIG IoT Marketplace in terms of offerings. The Provider Lib offers the following main functionalities:

* Creation of a BIG IoT Provider and Authentication on the Marketplace
* Definition of an Offering Description
   * Based on a pre-defined one via the Marketplace Web portal
   * Based on program code 
* Creation of Endpoints (by providing an access callback function)
* Registration of the Offering Description with its Endpoints

### Creation of a BIG IoT Provider and Authentication on the Marketplace

Before you can register your parking sensor data on the marketplace as an offering, you need to sign-up on the marketplace [here](https://market.big-iot.org/). Once you are logged in, you can create a new Organization for yourself (just click on "New Organization"). In a next step, you can create a new Provider instance (click on "MyProviders" and then "+Provider"). After you created a new Provider, you can copy the unique Provider ID and Secret into your program in order to associate your  Provider application with the newly created Provider instance.

```java
String MARKETPLACE_URL = "https://market.big-iot.org";
String PROVIDER_ID 	= "TestOrganization-TestProvider";
String PROVIDER_SECRET = "***************************";

ProviderSpark provider = ProviderSpark.create(PROVIDER_ID, MARKETPLACE_URL, "localhost", 9876)
                                   .authenticate(PROVIDER_SECRET);
```

With this in place, you can now create a new Provider instance in the code by providing the Provider ID and Marketplace URL. In this example, we use a *ProviderSpark* object, which is an easy way to create a provider with an embedded Spark Webserver. The webserver is started on the given DNS name or IP address and port, in this case "localhost" and port 9876. However, you can also use the standard *Provider* class, and connect it to an existing webserver (Tomcat, Jetty, etc.).  
Once you have created a provider, you also need to authenticate this on the marketplace. For this, you need to provide your Provider Secret. This Provider object will be used for all subsequent interactions with the marketplace. 

### Definition of an Offering Description

Now, that you have created a Porvider instance and authenticated it on the Marketplace, you can create an offering description for your parking data.

#### Based on a pre-defined one via the Marketplace Web portal

In case you have already created an Offering Description via the Marketplace Web poral, you can simply create a local instance in your code with the following call:

```java
RegistrableOfferingDescription offeringDescription = 
	provider.createOfferingDescriptionFromOfferingId("TestOrganization-TestProvider-DemoParkingOffering")
		// optional modify the offering description in your code, e.g. 
		.withPrice(Euros.amount(0.02))
		.withPricingModel(PricingModel.PER_ACCESS)
		.withLicenseType(LicenseType.CREATIVE_COMMONS);
``` 

If you want to alter that offering description in your code, you do this as illustrated above. With calls like `.withPrice()`, or `.withLicenseType()`, `.inRegion()`, `withTimePeriod()`, etc. you can alter the offering decription according to your needs.

#### Based on program code 

Similarly to the above, you can also create your offering description from scratch in your appliation. For example:

```java
RegistrableOfferingDescription offeringDescription = 
	OfferingDescription.createOfferingDescription("DemoParkingOffering")
		.withName("Demo Parking Offering")
		.withCategory("urn:big-iot:ParkingSpaceCategory")
		.withTimePeriod(new DateTime(2017, 1, 1, 0, 0, 0), new DateTime())
		.inRegion(BoundingBox.create(Location.create(42.1, 9.0), Location.create(43.2, 10.0)))
		// .inCity("Barcelona")
		.addInputData("longitude", new RDFType("schema:longitude"), ValueType.NUMBER)
		.addInputData("latitude", new RDFType("schema:latitude"), ValueType.NUMBER)
		.addInputData("radius", new RDFType("schema:geoRadius"), ValueType.NUMBER)
		.addOutputData("lon", new RDFType("schema:longitude"), ValueType.NUMBER)
		.addOutputData("lat", new RDFType("schema:latitude"), ValueType.NUMBER)
		.addOutputData("status", new RDFType("datex:parkingSpaceStatus"), ValueType.TEXT)
		.withPrice(Euros.amount(0.02))
		.withPricingModel(PricingModel.PER_ACCESS)
		.withLicenseType(LicenseType.CREATIVE_COMMONS);
```

The *OfferingDescription* class provides a builder method that lets you easily create a new offering description. For your offering to be visible on the marketplace, you have to provide an ID (here `"DemoParkingOffering"`) and a name (here `"Demo Parking Offering"`)for it. Also it is important to define a semantic type or category (here `"urn:big-iot:ParkingSpaceCategory"`) so that consumers can find it if they are interested in these type of offerings. 

**NOTE 1: A full list of already defined and supported semantic categories is available [here](https://big-iot.github.io/categories/). Via the Marketplace user interface, you can also create new categories during creation of an offering. Those '"proposed"' types can then also be used in your code.**

**NOTE 2: Using the right semantic catgory for your offering is very important for consumers to find it. So, try to use existing and well established categories whenever there is a good fit. If not, propose a new category that is meaningful.**
 
Additionally, you can define input parameters that can be set by your service’s consumers when accessing your platform (see `.addInputData()`). Here, we provide an example how to add a circle based filter (define by a location - i.e. a longitude and latitude of type `"schema:longitude"` and `"schema:latitude"` respectively - and radius of type `"schema:geoRadius"`) as input elements to your offering.  

You also have to provide information what your Offering will provide. This is specified my means of the `.addOutputData()` method. In our example, this are the parking space coordinates and the parking space status. The consumer of your offering will have the possibility to query for data within a specific area. He will obtain the data in form of a JSON Array, where each element consists of one parking space, consisting of at least a property with the name `"lon"` (of type `"schema:longitude"`), `"lat"` (of type `"schema:latitude"`) and `"status"` (of type `"datex:parkingSpaceStatus"`). 

Both input and output elements use the *RDFType class* in order to semantically annotate your data. 

**NOTE 3: New semantic types for input and output data can be directly created via the code. Just use the keyword '"proposed"' in front of your new type (e.g. '"proposed:parkingSpaceType"').**

**NOTE 4: Using well estabished semantic types your input and output data is very important for consumers to find the relevant offerings and to process the data correctly. Try to use existing and well established types whenever possible. If not, propose a new type that is meaningful.**

Providing a region, a price, a license type and time period (optionally) completes the offering description. 

### Creation of Endpoints

In the next step, you have to create the Endpoint, via which your offering is accessible by consumers. 


```java
Endpoints endpoints = Endpoints.create(offeringDescription)
                               .withAccessRequestHandler(accessCallback);
```

The route defines the endpoint URL to your platform. Using the *withAccessRequestHandler* method, you specify an access callback (see chapter 3.3.4) that will be called automatically, once a consumer accesses your offering.

### Registration of Offerings

Now that you created the offering description, it should be registered on the Marketplace so that other developers can find it. You will use the register method for that, which returns an instance of *RegisteredOffering* to control the offering state:

```java
RegisteredOffering offering = provider.register(offeringDescription, endpoints);
```

### Implementing the access Callback

Every time someone uses your offering, the BIG IoT Consumer Lib will forward the request to the provided callback function. We will show a simple example of how such a callback function implementation could look like:

```java
AccessRequestHandler accessCallbackParking = new AccessRequestHandler(){
	@Override
	public BigIotHttpResponse processRequestHandler{
		OfferingDescription offeringDescription, Map<String,Object> inputData) {
			double longitude, latitude, radius;

			BigIotHttpResponse errorResponse = BigIotHttpResponse.error()
			.withBody("{\"status\":\"error\"}").withStatus(422).asJsonType();

			if (!inputData.containsKey("longitude")) return errorResponse;
			longitude = new Double((String)inputData.get("longitude"));

			if (!inputData.containsKey("latitude")) return errorResponse;
			latitude = new Double((String)inputData.get("latitude"));


			if (!inputData.containsKey("radius")) return errorResponse;
			radius = new Double((String)inputData.get("radius"));

			Random r = new Random();
			int n = (int) (r.nextFloat()*10.0 + 10.0);
			String s= "[";
				for (int i = 0; i < n; i++) {
					if(i>0)s+=",\n";
					s+=String.format(Locale.US, 
					"{\"geoCoordinates\":{\n\"latitude\": %.4f,\n\"longitude\": %.4f},\n\"distance\":
					%.2f,\n\"status\":\"available\"\n}",
					r.nextFloat()*0.01+latitude, r.nextFloat()*0.01+longitude, r.nextFloat()*radius);
				}
			s+="]";
			return BigIotHttpResponse.okay().withBody(s).asJsonType();
		};	
	}
};

```

The *AccessRequestHandler* class is an abstract class that requires you to override the *processRequestHandler* method. This method receives the OfferingDescription object that we created earlier and the inputData Map which contains the input filters that the consumer has specified during his access request.  
If any of the possible filters have been used, we pass them to a service function which, in this tutorial, just simulates parking data. Usually, here you would call your database backend to retrieve real IoT data.

*AccessRequestHandler* is implemented as a functional interface. If you prefer, you can also provide the application logic in the Lambda notation.

### Offering deactivation
If you want to remove your offering from the marketplace, just call the *deactivate* method on the offering.

```java
offering.deregister();
```

Congratulations! You have created your first offering on the BIG IoT Marketplace.



