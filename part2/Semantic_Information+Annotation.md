# Semantic Information / Semantic Annotation [Wenbin Li, Hamza Baqa]
Existing syntax information can be enriched with semantics and transformed to semantic information via semantic annotation.  Semantic annotation is the process of linking existing syntax information with specific ontologies to provide both machine understandable and human readable descriptions, while the source information can be structured document, services, functions, metadata of image and videos, etc. Ontologies are the interoperable and standard means to provide semantics to existing data and furthermore link different information together via predefined relations. The semantic annotation bridges the gap between syntax and semantics and evolves the existing information for advanced data use and analytics. 

The general semantic process can be briefly summarized into the three following steps 
1. Preparation of source information to be annotated; 
2. Identification or definition of ontologies to be used; 
3. Manual or automatic link between source information to ontologies.

Semantic annotations are widely used in various data-centric domains. Here we present an example following our smart home scenario, in which a room with an energy limit profile is equipped with a temperature sensor providing temperature measurement and a washing machine providing washing machine with states and remote washing services to turn on/off and switch mode. The description of the rooms is serialized in JSON as presented below. 

```
{	"id": "room1001",
	"type": "Room",
	"energyProfile": "/Limit",
	"devices": [{	"id": "ts001",
			"type": "TemperatureSensor",
			"value": "25"},
		       {	"id": "wm002",
			"type": "WashingMachine",
			"state": "/state",
			"service": /switch"    }]
}
```

The JSON description of the Room1001 is the source information to be annotated, and the main ontologies we use for semantic annotation are SAREF and SAREF4ENER, which is introduced in previous chapters.  
Starting from the general terms, we need to map the general terms “id” and “type” to JSON-LD node identifier and RDF type concept.    


```
	"id": "@id",
	"type": "rdf:type"

```

so that all ids in the JSON descriptions are defined as an object node with an URI as identifier, while all thing types are further linked as a type of an ontology class. 
In the following, all type targets in the JSON are mapped to different SAREF4ENER, except that we use the schema ontology to annotate the type of Room which is not defined in SAREF or SAREF4ENER. 


```
  "TemperatureSensor": "saref:TemperatureSensor",
	"WashingMachine": "saref:WashingMachine",
	"Room": "schema:Room"

```
At last, we link the energyProfie, sensingValue, state and services with SAREF and SAREF4ENER concepts, 

```
"energyProfile":"saref4ener:hasEnergy",
"/Limit":"saref4ener:energyMax",
"value":"saref:hasValue",
"service": "saref:offers",
"wm002/switch": "saref:Service",
"state ": "saref:hasState",
"wm002/state": "saref:State"

```
At the of the annotation, all terms used in JSON are linked to semantic concepts, for example now we know that the resource "wm002/switch" is a device service defined by SAREF, and the "/Limit" is a resource describing the maximum energy consumption as specified in SAREF4ENER. 
The final serialization formats of the semantic annotation are rather flexible and can be formatted as with all semantic formats such as triples, JSON-LD, RDF/XML, etc. One convenient way for JSON document is to use JSON-LD to serialize the semantic annotation result, as we only need to add an extra field of “@conext” at the top of existing JSON which contains all previously defined mappings. 

```
	{	"@context": {
			"id": "@id",
			"wm002/state": "saref:State",
			… …
		},
		"id": "room1001",
		"type": "Room", 
		… …
	}

```
Here follows a sample of semantic annotation result based on triples format, 

```
Room1001 	rdf:type 		"schema:Room",
Room1001	saref4ener:hasEnergy 	"/Limit",
"/Limit" 	rdf:type 		"saref4ener:energyMax",
… …

```
Other examples: 
- Markus Stocker provides an example in his post of representing metadata about the LI-7500 sensing device used for monitoring of CO2 and H2O fluxes. http://markusstocker.com/sensing-devices-and-their-metadata/
- An example of the SSN-XG sensor ontology used to describe the Vaisala WM30 sensing device which measures wind speed and wind direction. https://www.w3.org/2005/Incubator/ssn/ssnx/meteo/WM30 
- A typical design of semantic annotation framework can be found in, https://www.researchgate.net/publication/319036946_Towards_a_Semantics_Extractor_for_Interoperability_of_IoT_Platforms

----------------------------------------------------------------------------------------------------------



This example describes how to represent metadata and observations from a Salinity Sensor sensing device using the Semantic Sensor Network [SSN] (https://www.w3.org/2005/Incubator/ssn/ssnx/ssn) ontology. 

The Salinity Sensor is a device that can measure the salinity of liquids. The SSN ontology defines the class ssn:SensingDevice as a sub class of ssn:Sensor as well as of ssn:Device. Hence, as salinity sensor is a type of sensing device it can be represented as a subclass of ssn:SensingDevice.
```
SalinitySensor rdfs:subClassOf ssn:SensingDevice
```

In order to define a concrete instance of a Salinity Sensor, we need to name it (e.g. SENS_WAT_SAL01) and define its type:
```
SENS_WAT_SAL01 rdf:type SalinitySensor
```

Next, we need to define what the sensing device observes. This relation between the sensing device and what it observes is done through a ssn:Property.
```
SENS_WAT_SAL01 ssn:observes salinityConcentration
salinityConcentration rdf:type ssn:Property
```

We can also define the range that the sensor can measure. In our case, the Salinity Sensor can measure the entire range from 0-50 ppt (parts per thousand). The SSN ontology can be used in conjunction with the DUL ontology to manage data.
```
SENS_WAT_SAL01 ssn:hasOperatingRange sal01SalinityOperatingRange
sal01SalinityOperatingRange rdf:type ssn:OperatingRange
sal01SalinityOperatingRange dul:hasRegion aSalinityRegion
aSalinityRegion rdf:type dul:Region
aSalinityRegion dul:hasRegionDataValue "0 +50ppt"
```

Once we have represented the metadata about the sensor, we can model its observations as instances of class ssn:Observation. For each sample we have the following information: a timestamp, a value and its units. For example, the salinity observation of 26.35 ppt taken on the 1st June 2018 at 09:25 by SENS_WAT_SAL01 would be modelled as:
sample1 rdf:type ssn:Observation

An observation is the result of estimating a value of a property of a feature using a sensing method implemented by a sensor. In our case, the feature is the water quality that can be measured through different properties being one of them the salinity concentration. In this case, we would say that the observation sample1 was observed by the sensor SENS_WAT_SAL01, which observed the salinity property in order to measure the water quality feature. 
```
sample1 ssn:observedBy SENS_WAT_SAL01
sample1 ssn:observedProperty salinityConcentration
sample1 ssn:featureOfInterest waterQuality
waterQuality rdf:type ssn:FeatureOfInterest
waterQuality ssn:hasProperty salinityConcentration
```

Finally, we need to represent the value of the measurement of the sample1 observation and the time stamp when the measurement was taken. The DUL ontology is used in combination with SSN for this purpose. Each measurement is related to its value through the ssn:observationResult property and to its timestamp through the ssn:observationResultTime property. The measurement is an instance of ssn:SensorOutput with a value of type ssn:ObservationValue whereas the timestamp can be modelled as an instance of dul:TimeInterval.
```
sample1 ssn:observationResult result1
result1 rdf:type ssn:SensorOutput
result1 ssn:hasValue sample1val1
sample1val1 rdf:type ssn:ObservationValue
sample1val1 dul:hasRegionDataValue "26.35"^^xsd:double

sample1 ssn:observationResultTime result1time
result1time rdf:type dul:TimeInterval
result1time dul:hasRegionDataValue "2018-06-01T09:25:12.010"^^xsd:dateTime
```

Other examples: 
- Markus Stocker provides an example in his post of representing metadata about the LI-7500 sensing device used for monitoring of CO2 and H2O fluxes. http://markusstocker.com/sensing-devices-and-their-metadata/
- An example of the SSN-XG sensor ontology used to describe the Vaisala WM30 sensing device which measures wind speed and wind direction. https://www.w3.org/2005/Incubator/ssn/ssnx/meteo/WM30 
