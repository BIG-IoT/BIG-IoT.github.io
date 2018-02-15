---
layout: single
title: BIG IoT Offering Categories (15.02.2018)
sidebar: 
  nav: "docs"
--- 
<p>
  <ul id="root" style="font-size: 0.7em;">
  </ul>
</p>

<script type="text/javascript">
// TODO fetch from SPARQL endpoint?
// query: https://virtuoso-dev.....org/sparql
  // PREFIX bigiot-core: <http://schema.big-iot.org/core/>
  // PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
  // PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
  // SELECT ?parent ?child
  // FROM <http://big-iot.eu/ontologies_dev#>
  // WHERE {
  // ?parent a bigiot-core:OfferingCategory;
  //      skos:narrower ?child.
  // FILTER NOT EXISTS{?parent a bigiot-core:ProposedOfferingCategory.}
  // FILTER NOT EXISTS{?child a bigiot-core:ProposedOfferingCategory.}
  // }
// Old Query:  
  // select ?parent ?child where { graph <http://big-iot.eu/ontologies#> { ?parent <http://www.w3.org/2004/02/skos/core#narrower> ?child . } }
  
var sparql = { "head": { "link": [], "vars": ["parent", "child"] },
  "results": { "distinct": false, "ordered": true, "bindings": [
    { "parent": { "type": "uri", "value": "urn:big-iot:AirPollutionIndicatorCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:O3Category" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:AirPollutionIndicatorCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:GHGCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:AirPollutionIndicatorCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:SO2Category" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:AirPollutionIndicatorCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:COCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:AirPollutionIndicatorCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:PMCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:AirPollutionIndicatorCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:NO2Category" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:ChargingCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:ChargingPointCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:ChargingCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:ChargingStationCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:EnvironmentalIndicatorCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:PollutionIndicatorCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:GHGCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:N2OCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:GHGCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:CH4Category" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:GHGCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:CO2Category" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:MobilityFeatureCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:ParkingCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:MobilityFeatureCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:TransportationCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:MobilityFeatureCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:PeopleDensityEstimationCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:MobilityFeatureCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:TrafficCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:MobilityFeatureCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:ChargingCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:MobilityFeatureCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:TravelTimeCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:PMCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:PM25Category" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:PMCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:PM10Category" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:ParkingCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:ParkingSpaceCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:ParkingCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:ParkingSiteCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:PollutionIndicatorCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:AirPollutionIndicatorCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:PollutionIndicatorCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:WaterPollutionIndicatorCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:PollutionIndicatorCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:NoisePollutionIndicatorCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:TrafficCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:TrafficObstacleCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:TrafficCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:TrafficDataCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:TrafficCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:AccidentCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:TransportationCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:BikeSharingStationCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:TransportationCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:CarDiagnosticsCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:TransportationCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:LocationTrackingCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:TransportationCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:BusCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:allOfferingsCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:WeatherIndicatorCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:allOfferingsCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:EnvironmentalIndicatorCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:allOfferingsCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:MobilityFeatureCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:PeopleDensityEstimationCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:PeopleDensityOnBusCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:PeopleDensityEstimationCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:PeopleMobilityWithinAreaCategory" }},
    { "parent": { "type": "uri", "value": "urn:big-iot:PeopleDensityEstimationCategory" }	, "child": { "type": "uri", "value": "urn:big-iot:PeopleDensityInAreaCategory" }} ] } };
</script>

<script type="text/javascript">
  function element(category) {
    var label = category.replace('urn:big-iot:', '')
                        .replace('Category', '')
                        .replace(/([A-Z])([a-z]+)/g, " $1$2");

    var e = document.createElement("li");
    e.innerHTML = label + ': <code>' + category + "</code><ul></ul>";
    return e;
  }

  var elements = {};

  sparql.results.bindings.forEach(b => {
    var parent = b.parent.value;
    var child = b.child.value;

    if (!elements[parent]) {
      elements[parent] = element(parent);
    }

    if (!elements[child]) {
      elements[child] = element(child);
    }

    elements[parent].querySelector("ul").appendChild(elements[child]);
  });

  var root = document.getElementById("root");
  root.appendChild(elements["urn:big-iot:allOfferingsCategory"]);
</script>
