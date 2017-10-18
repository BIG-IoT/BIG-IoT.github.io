---
layout: single
title: Release Notes
sidebar: 
  nav: "docs"
---

### 0.9.6

*Highlighted features and changes since 0.8.0*

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
* Introduced `OfferingSelector` for defining a rule set to select a single offering response of a discovery
* Map the access response to your POJO 
  * Automatic mapping: `List<MyPojo> list  = res.get().map(MyPojo.class)`
  * Source code annoated remapping: `List<MyPojoAnnoated> list  = res.get().map(MyPojoAnnotated.class)`
  * Programmatic remapping: `List pojo = res.get().map(MyCustomPojo.class, OutputMapping.create().addTypeMapping("schema:geoCoordinates", "myCoordinate")/*..*/)`  
* Introduced HTTP proxy configuration (e.g. `provider.setProxy("127.0.0.1",3128)`, `provider.addProxyBypass("172.17.17.100")`)
* Supported automatic renewal of offerings
* Specification of route for offering registration is now optional
* Enabled offering queries including a subset of search parameters (search for all type of offerings in Barcelona)
* Added `inCity` for creating offering descriptions and offering queries
