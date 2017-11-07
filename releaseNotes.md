---
layout: single
title: Release Notes
sidebar: 
  nav: "docs"
---

### 0.9.7

*Highlighted features and changes since 0.9.6:*

* Extended Consumer Lib to also access Offerings with an HTTP endpoint (before only HTTPS was supported)
* Extended AccessRequestHandler to also provide the Subscriber Id and Consumer information in the callback function
   * `public BigIotHttpResponse processRequestHandler (OfferingDescription offeringDescription, Map<String,Object> inputData)`   --> 
   * `public BigIotHttpResponse processRequestHandler (OfferingDescription offeringDescription, Map<String,Object> inputData, String subscriberId, String consumerInfo)`
* Introduces Provider Lib functionality for acess streams: 
   * This allows a provider to simply queue stream data in an `RegisteredOffering` with the `queue()` method
   * For such offerings, the Provider Lib will then directly stream those queued data to consumers upon an access requests - without the access callback function
   * `offering.flush()` will flush queued data in the access stream
* Extended Consumer Lib trust manager to trust besides the Provider Lib endpoints also any Provider end point with a certificate that is trusted by the Java VM 
* Improved stability of Marketplece interactions, e.g. re-registration will happen after Marketplace crash
* Bug fixes:
   * `.inCity()` or `.inRegion()` is now optional for Offering Descriptions
   * `.withPricingModel(PricingModel.FREE)` is now also supported by the Provider Lib supported
   * `.withPrice()` and `.withPriceModel()` are now optional and a Free Pricing Model is chosed by default
   * the access response `.map()` function works now also when the response includes unexpected property names - those are simnply ignored
   * `BigIotHttpResponse.okay().withBody()` was extended so also allow the passing of JSON Objects or JSON Arrays

### 0.9.6

*Highlighted features and changes since 0.8.0. Checkout [more-java-examples](https://github.com/BIG-IoT/example-projects/tree/master/more-java-examples) on GitHub*

* Renamed package prefix from `org.bigiot.lib` to `org.eclipse.bigiot.lib`
* Renamed `addInputDataElement` to `addInputData` in `RegistrableOfferingDescription`
* Renamed term `AccoutingType` to `PricingModel` in all methods
* Introduced Complex Types
  * On provider side, use `addInputData(/*..*/`, `IOData.createMembers()./*..*/)` to configure your offering
  * On consumer side, use `AccessParameters.create().addNameValue("myComplex", AccessParameters.create()./*..*/)`
* Added support for consumer side integrated offerings (integration mode 2) with the following degrees of freedom
  * HTTP POST, GET, PUT and COAP  
  * XML and JSON encoding
  * Parameters in path, query, body and body template
* Introduced `OfferingSelector` for defining a rule set to select a single offering out of the discovery response
* Map the access response to your POJO 
  * Automatic mapping: `List<MyPojo> list  = res.get().map(MyPojo.class)`
  * Source code annoated remapping: `List<MyPojoAnnoated> list  = res.get().map(MyPojoAnnotated.class)`
  * Programmatic remapping: `List pojo = res.get().map(MyCustomPojo.class, OutputMapping.create().addTypeMapping("schema:geoCoordinates", "myCoordinate")/*..*/)`  
* Introduced HTTP proxy configuration, use `provider.setProxy("127.0.0.1",3128)` and `provider.addProxyBypass("172.17.17.100")`)
* Supported automatic renewal of offerings
* Specification of route for offering registration is now optional
* Enabled offering queries including a subset of search parameters (search for all type of offerings in Barcelona)
* Added `inCity` for creating offering descriptions and offering queries
