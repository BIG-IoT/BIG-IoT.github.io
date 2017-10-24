---
layout: single
title: More BIG IoT Lib Functionality
sidebar: 
  nav: "docs"
---

###  ***BETA***    Complex types - Provider perspective 
In this section, we want to describe an example of **complex types**. Complex types are used, so that providers can semantically describe complex datatypes that can contain nested structures. This is possible in the latest version of the BIG IoT SDK.
**Please be aware that complex types are not yet supported by the marketplace, nevertheless they can already be used in the BIG IoT SDK.**

```java
RegistrableOfferingDescription offeringDescription = provider.createOfferingDescription("parkingSpotFinder")
	.withInformation("Demo Parking Offering", new RDFType("bigiot:Parking"))
	.addInputData("areaSpecification", new RDFType("schema:GeoCircle"), IOData.createMembers()
		.addInputData("geoCoordinates", new RDFType("schema:geoCoordinates"), IOData.createMembers()
			.addInputData("longitude", new RDFType("schema:longitude"), ValueType.NUMBER)
			.addInputData("latitude", new RDFType("schema:latitude"), ValueType.NUMBER))
		.addInputData("radius", new RDFType("schema:geoRadius"), ValueType.NUMBER))
	.addOutputData("geoCoordinates", new RDFType("schema:geoCoordinates"), IOData.createMembers()
		.addOutputData("longitude", new RDFType("schema:longitude"), ValueType.NUMBER)
		.addOutputData("latitude", new RDFType("schema:latitude"), ValueType.NUMBER))
	.addOutputData("distance", new RDFType("datex:distanceFromParkingSpace"), ValueType.NUMBER)
	.addOutputData("status", new RDFType("datex:parkingSpaceStatus"), ValueType.TEXT)
	.inCity("Barcelona")
	.withPrice(Euros.amount(0.001))
	.withPricingModel(PricingModel.PER_ACCESS)
	.withLicenseType(LicenseType.OPEN_DATA_LICENSE) 
	//Below is actually Offering specific	
	.withAccessRequestHandler(accessCallbackDummy);
```

Let’s expand our previous example with a more complex parking offering. We want to register a parking information offering on the marketplace with the complex type **areaSpecification**, which is described as the semantic RDF type schema:GeoCircle. It consists of the two nested types: geoCoordinates and geoRadius.
We also add the complex type geoCoordinates to the list of output parameters.  

### ***BETA***  Complex types - Consumer perspective 
Now that we’ve added complex types to our provider’s offering, we want to add complex type support to our consumer. In order to do that we make use of complex access parameters. 
As you can see in the following example, this is quite similar to the way, we use complex types in offering descriptions. We use the AccessParameters.create method in order to create the access parameter for the areaSpecification. 

```java
AccessParameters accessParameters = AccessParameters
  .create()
  .addNameValue("areaSpecification", AccessParameters
  .create()
  .addNameValue("geoCoordinates", AccessParameters
    .create()
    .addNameValue("latitude", 50.22)
    .addNameValue("longitude", 8.11))
  .addNameValue("radius", 777));
```

### Using integration mode 3
Up till now, we have only considered the case that **you** provide the callback code to access your IoT platforms. But what if you cannot or do not want to start additional services in your environment? In this example, you learn how to register and access offerings that uses the *BIG IoT integration mode 3*, which enables a completely consumer lib - driven access to your IoT platform. 

Suppose, we want to integrate air quality information using the public available Web API OpenAQ (https://openaq.org). In order to do that via integration mode 3 we first register our offering:

```java
RegistrableOfferingDescription airQualityOffering = provider
	.createOfferingDescription("Open_AQ_AirQualityOffering")
	.withInformation(new Information("OpenAQ_AirQuality", "bigiot:AirQuality"))
	.inRegion(Region.city("Berlin"))
	.withPrice(Euros.amount(0.01))
	.withPricingModel(PricingModel.PER_ACCESS)
	.withLicenseType(LicenseType.OPEN_DATA_LICENSE) 
// The following properties are different to other integration modes
	.addInputDataInQuery("coordinates", new RDFType("schema:location"), ValueType.TEXT)
	.addInputDataInQuery("radius", new RDFType("schema:geoRadius"), ValueType.TEXT)
	.addInputDataInQuery("from", new RDFType("schema:date"), ValueType.TEXT)
	.addInputDataInQuery("limit", new RDFType("schema:value"), ValueType.NUMBER)
	.asHttpGet()
	.acceptsJson()
	.producesJson()
	.onExternalEndpoint("https://api.openaq.org/v1/measurements");

```
The BIG IoT Lib supports different ways, how to transmit input parameters when calling a service function of a Web API: parameters can be encoded in the query (...?param1=…&param2=…) via *addInputDataInQuery*, in the path (…/paramValue1/paramValue2) via *addInputDataInPath*, in the message body e.g. as JSON via *addInputDataInBody*, or a combination of all three. 
In this example, the parameters are added as query parameters in the URL. 

If your offering requires complex parameter encoding, maybe even with many static parts, you can check out *addInputDataInTemplate*. 
The output parameter definition works analogous to the input parameter definition. 

We specify that we access the Web API through an HTTP GET request, using the *asHttpGet* method, followed by a specification of what content types shall be used. 

The endpoint of the Web API is added to the registration in the last step. Note, that the method *onExternalEndpoint* is specifically provided by the BIG IoT Lib for integration mode 3. 

Now, OpenAQ can be used through BIG IoT. However, it is important to note, that currently the lib does not support supplying input parameters as RDF Types as in the previous example. Here is an example, how a consumer can specify parameters to access the OpenAQ offering: 

```java
/* Create a hashmap to hold parameters for filtering access*/
AccessParameters accessParameters = AccessParameters.create()
	.addNameValue("radius", "2500")
	.addNameValue("from", "2017-08-25")
	.addNameValue("limit", "10000");
```

### Using Proxy support

If you access the Internet through a proxy, for example because you are in a corporate environment, you can use the BIG IoT Lib’s proxy support. You can use the *setProxy* method to set the host address and the port of your proxy server. If you want to add a host to your proxy exception list, use *addProxyBypass* (*removeProxyBypass* for removal). 

```java
ProviderSpark provider = new ProviderSpark(PROVIDER_ID, MARKETPLACE_URI, "localhost", 9002);
provider.setProxy("127.0.0.1", 3128);
provider.addProxyBypass("172.17.17.100");
```

### Using Swagger support to describe offerings

Another feature of the BIG IoT Lib allows you to automatically evaluate your offering with Swagger UI. This is useful, for example when you implement a consumer and you want to try different calls. Just add the following line to your provider setup code:

```java
SwaggerGenerator.deploySwaggerFileFrom(offeringDescription, "hi@bigiot.org");
```
Make sure you have added the correct dependencies to your gradle or maven file (group: 'org.bigiot.lib', name: 'bigiot-lib-swagger', version: '0.9.3'). The lib will automatically configure a Spark webserver hosting a dynamic web page with Swagger UI preconfigured with your offering.

