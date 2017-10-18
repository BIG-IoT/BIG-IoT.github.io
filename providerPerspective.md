---
layout: single
title: Provide IoT Offerings
sidebar: 
  nav: "docs"
---

Assuming, you have developed an IoT platform for parking data. Now you want to make this data public. How do you connect your platform to the BIG IoT Marketplace in order to provide this data to other developers so they can use your data in their applications?

Using the BIG IoT Provider Lib, you can manage your data on the BIG IoT Marketplace in terms of offerings. The Provider Lib offers the following main functionalities:

* Registration of offerings
* Activation and deactivation of offerings
*	Renewing offerings
*	Providing access callbacks


### Authentication on the Marketplace

Before you can register your parking sensor data on the marketplace as an offering, you need to authenticate with the marketplace. 

```java
String MARKETPLACE_URL = "https://market.big-iot.org";
String PROVIDER_ID 	= "TestOrganization-DemoProvider";
String PROVIDER_SECRET = "****";
ProviderSpark provider = new ProviderSpark(PROVIDER_ID, MARKETPLACE_URL, "localhost", 9020); 
provider.authenticate(PROVIDER_SECRET);

```

First of all, you create a Provider instance, passing a Provider ID and a Marketplace URL you want to connect to. In our example, we use a *ProviderSpark* object, which is an easy way to create a provider with an embedded Spark webserver. The webserver is started on the given URL and port, in this case localhost on port 9020. However, you can also use the standard *Provider* class, and connect it to an existing webserver (Tomcat, Jetty, etc.).  
When creating a provider at the marketplace, you receive a unique ID and a token. You pass this data to the provider objects authenticate function in order to finish your authentication on the marketplace. The Provider object will be used for all subsequent interactions with the marketplace. 

### Create an Offering

Now, that you are authenticated you can create an offering for your parking data. The next code block shows how you can use the *Offering* class to do this:

```java
RegistrableOfferingDescription offeringDescription = provider
	.createOfferingDescription("parking_info_offering")
	.withInformation("Demo Parking Offering", new RDFType("bigiot:Parking"))
	.addInputData("longitude", new RDFType("schema:longitude"), ValueType.NUMBER)
	.addInputData("latitude", new RDFType("schema:latitude"), ValueType.NUMBER)
	.addInputData("radius", new RDFType("schema:radius"), ValueType.NUMBER)
	.addOutputData("longitude", new RDFType("schema:longitude"), ValueType.NUMBER)
	.addOutputData("latitude", new RDFType("schema:latitude"), ValueType.NUMBER)
	.addOutputData("status", new RDFType("datex:parkingSpaceStatus"), ValueType.TEXT)
	.inCity("Barcelona")
	.withPrice(Euros.amount(0.001))
	.withPricingModel(PricingModel.PER_ACCESS)
	.withLicenseType(LicenseType.OPEN_DATA_LICENSE) 
	// Below is actually Offering specific	
	.withRoute("parking")
	.withAccessRequestHandler(accessCallbackParking);
```

The *OfferingDescription* class provides a builder method that lets you easily create a new offering. For your offering to be visible on the marketplace, you have to provide a name for it. Also it is important to define a type for the offering so that consumers can automatically find it (see Consumer section).  

Additionally, you can define input parameters that can be set by your service’s consumers when accessing your platform. Here, we provide an example how to add location based filters as input elements to your offering.  

You provide the output of your offering through the *addOutputData* method which are the parking spot coordinates and the parking spot status in this example. The consumer of your data will have the possibility to query for data within a specific area. He will retrieve data that conforms to the schema of the type http://schema.org/parking.  

Both input and output elements use the *RDFType class* which makes it easy to semantically annotate your data. **This is very important for consumers to find your offerings, so be thoughtful with your annotations!**  

Providing a region, a price, a license type and a route completes the offering description. The route defines the endpoint URL to your platform. Using the *withAccessRequestHandler* method, you specify an access callback (see chapter 3.3.4) that will be called automatically, once a consumer accesses your offering.

### Registration of Offerings

Now that you created the offering description, it should be registered on the Marketplace so that other developers can find it. You will use the register method for that, which returns an instance of *RegisteredOffering* to control the offering state:

```java
RegisteredOffering offering = offeringDescription.register();
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



