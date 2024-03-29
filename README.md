# Uniform Parking Tariff Protocol (UPTP-1)
## Version 0.1
|Date|Author|Version|Change log|
|:---:|:---:|:---:|:---:|
|2018-06-10|Steven Ly|0.1|Added format description as readme|
|2018-10-23|Robert Petersen|0.1|Updated name of protocol|

## Table of Contents   
* [Tariff](#tariff)
  * [Tariff Object](#tariff-object)
  * [Restrictions](#restrictions)
  * [Rate](#rate)
    * [Rate Examples](#rate-examples)
  * [Schedules](#schedules)
    * [Active Schedule](#activeschedule-object)
    * [Active Schedule Examples](#active-schedule-examples)
    * [Valid Schedule](#validschedule-object)
    * [Valid Schedule Examples](#valid-schedule-examples)
  * [Log](#log-object)
* [Location](#location)
  * [Address](#address-object)
  * [Contact](#contact-object)
  * [GeoLocation](#geospatial-location)
  * [Polygon](#polygon)
  * [Auxiliary](#auxiliary)
  * [Schedule](#schedule) 
  * [Surcharges](#surcharges)
* [Occupancy](#occupancy)
  * [Parking space](#parking-space)

# Protocol format
The protocol’s format is defined for JSON format.

The parking data included in the protocol’s format, are referred to as elements. The elements are further divided into three categories: location, occupancy, and tariff. The three categories act as root objects and hold the other elements together. The root elements are separated into three different arrays under a parent object. Arrays are used for the root element as there might be multiple occurrences of each type. Any object that has multiple occurrences, such as the root objects, include a unique identification string.

## Tariff
Tariff data is the object that describes the parking fees of a parking location. Tariff data is included in the protocol to enable fast and easy sharing of new tariffs. There exists a wide variety of how a tariff can be expressed. The proposed protocol aims to express both the simplest and the most complex tariff while maintaining a generic method of expressing tariffs. 

The tariff information is divided into three parts: restriction, rate, and schedule data. Because each location can have multiple tariff objects, each tariff must be provided with a globally unique tariff ID and mapped with a location ID.  

### Tariff Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|tariffId|String(ID)|Unique ID of the tariff|Required|
|locationId|String(ID)|ID of the location the tariff is meant for|Required|
|restriciton|Restrictions|Object containing restrictions of the tariff|Required|
|rate|Array([Rate](#rate-object))|Rate components describing the parking fees|Required<br/>Min items: 1|
|activeSchedule|Array([ActiveSchedule](#activeschedule-object))|Schedules defining the paid hours|Required<br/>Min items: 1|
|validSchedule|Array([ValidSchedule](#validschedule-object))|Schedules defining when the tariff is valid|Required<br/>Min items: 1|
|log|[Log](#log-object)|Information regarding when the tariff object was created and last updated|Required|

### Restrictions
Restrictions object includes information that limits or provide additional information regarding a tariff. 

The elements that restricts tariffs are: _argetGroup_, _vehicles_, _maxParkingTime_, _maxPaidParkingTime_, _maxFee_, and _minFee_.  All these elements are optional to include in the format. Therefore, if any of the elements are unassigned it means that the tariff is not restricted in any of the specific ways. 

The elements that provides additional information for a tariff are: _tariffType_, _prepaid_, and _resetTime_. The optional elements are prepaid, and resetTime. The prepaid element should be false by default and if no resetTime is defined, it means that the rates will never reset, unless the driver restarts the parking themselves.

#### Restriction Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|tariffType|TariffType|Defines the tariff type|Required|
|maxFee|Float|Overall maximum fee for this tariff <br/> Might be restricted by max rates instead|Min value: 0|
|minFee|Float|Minimum fee a driver need to pay|Min value: 0|
|maxParkingTime|Int|Maximum allowed parking time||
|maxPaidParkingTime|Int|Maximum allowed parking time during paid hours||
|prepaid|Boolean|Defines if the fees are paid in advance||
|resetTime|Int|The time next day begins in minutes|Min value: 0 <br/> Max value: 1440|
|targetGroup|Array(ParkingType)|The target parking groups that the tariff is valid for _only_|Required<br/>Default: PUBLIC|
|vehicles|Array(VehicleType)|The target vehicle types that the tariff is valid for _only_||

Part of [Tariff](#tariff-object).
#### TariffType Tokens
|Token Name|Description|
|----------|-----------|
|REGULAR|The tariff has stoppable rates|
|FIXED|The tariff only has fixed rates|

#### ParkingType Tokens
|Token Name|Description|
|----------|-----------|
|PRIVATE|Private tariff|
|PERSONNEL|Tariff is for personnel only|
|RESIDENTIAL|Tariff is for the residents in the area|
|INDIVIDUAL|The tariff targets a single driver|
|PUBLIC|The tariff is open to the public|

TODO: Determine all necessery vehicle types
#### VehicleType Tokens
|Token Name|Description|
|----------|-----------|
|CAR|Personal car|
|VAN||
|MC||
|TRUCK||
|ELECTRIC|Any kind of electric vehicle|
|BUS||

### Rate
The rate is the smallest component of a parking fee, which describes the price development. The rate component describes the fee as a value over a time interval in minutes. For more advanced price strategies, a tariff can consist of multiple rate components. Therefore, each rate must a number defining in which order the rates applies, which also can be used to differentiate the rates. Furthermore, the rate object should be used to define periodical maximum fees, such as daily, weekly, monthly, or 8hrs maximum fees.


#### Rate Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|order|Int|Unique number defining the priority of the rate. <br />Lower equals higher priority|Required<br/>Min value: 0|
|value|Float|Number defining the fee value|Required<br/>Min value: 0|
|interval|Int|Number defining how long the value is valid for|Required<br/>Min value: 1|
|intervals|Int|Number defining how many times the rate should be repeated at most|Required<br/>Min value: 1|
|unit|UnitType|Token defining the time unit of interval|Default: MIN|
|repeat|Boolean|Set true if the rate should be repeated until stopped|Default: false|
|max|Boolean|Set true if the rate defines a max fee|Default: false|
|countOnlyPaidTime|Boolean|Set true if the interval only should only count for paid hours|Default: false|

Part of [Tariff](#tariff-object).
#### UnitType Tokens
|Token Name|Description|
|----------|-----------|
|MIN|The rate interval is given in minutes|
|DAY|The rate interval is given in days|
|WEEK|The rate interval is given in weeks|
|MONTH|The rate interval is given in monts|
|YEAR|The rate interval is given in years|

#### Rate Examples
The simplest tariff with a constant cost per time units can be expressed using one rate component. However, a tariff can be more advanced than a constant cost per time unit, which requires more than 1 rate component to express the tariff.
For example: see **Rate example 1**, which describes a pricing strategy: the first-hour costs 10 SEK, the second hour is free, and thereafter the cost is 20 SEK per hour. Because no unit was provided in the rates, the interval is assumed to be in minutes. 

#### Rate example 1
```
{
 "rates":[
  {"order":0, "value":10, "interval":60, "intervals":1, "repeat":false},
  {"order":1, "value":0,  "interval":60, "intervals":1, "repeat":false},
  {"order":2, "value":20, "interval":60, "intervals":1, "repeat":true}
]}
```

Apart from consisting of several rate components, a complex tariff might also include both regular and fixed rates. In such case, the fixed rates must be expressed differently than the regular rates. For example: see **Rate example 2**, which describes the pricing strategy: the first hour (1 + 59 minutes) has a fixed rate of 15 SEK, and thereafter the cost is 10 SEK per hour. 

#### Rate example 2
```
{
 "rates":[
  {"order":0, "value":15, "interval":1,  "intervals":1, "repeat":false},
  {"order":1, "value":0,  "interval":59, "intervals":1, "repeat":false},
  {"order":2, "value":10, "interval":60, "intervals":1, "repeat":true}
]}
```

Apart from complex tariffs, a tariff might also have maximum fees such as daily, weekly, or monthly. However, a day, week, or month can be defined differently. Therefore, the rate object includes the element unit which allows to define the date time unit. In the table below possible date time definintions are shown. Using the rate objects, it is, therefore, possible to define different types of max fees. Max fees are recognized by the element max being set to true. For example: see **Rate example 3**, which describes the pricing strategy: 10 SEK per hour, with a daily maximum fee of 60 SEK (full day, i.e. 24h), and a weekly  maximum fee of 200 (full week, i.e. 168h). 

##### Possible date time definitions
|Value unit|  MIN  | Calendar Days | Calendar Weeks | Calendar Months | Calendar Years |
|----------|----:|----:|----:|----:|----:|
|1 Hour | 60|-|-|-|-|
|1 Day | 1 440|1|-|-|-|
|1 Week | 10 080|7|1|-|-|
|1 Month | 40 320 <br /> to 44 640|28 <br /> to 31| ~4|1|-|
|1 Year | 525 600 <br /> to 527 040| 365 <br /> to 366|~52|12|1|

#### Rate example 3
```
{
 "rates":[
  {"order":1,"value":10,"interval":60,"intervals":1,"repeat":true},
  {"order":2,"value":60,"interval":1440,"intervals":1,"max":true},
  {"order":3,"value":200,"interval":10080,"intervals":1,"max":true}
]}
```
Part of [Tariff](#tariff-object).
### Schedules
There are two types of schedule objects: active- and validSchedule. The activeSchedule is used to define when a tariff is active, and restricts a tariff based on the elapsed- and current time. An activeSchedule can be interpreted as rules to when a tariff is applicable. This is necessary to be able to define time periods when the parking is free. The activeSchedule are also used to differentiate which tariff to use when a single location has multiple tariffs. For example, parking after a specific time might have a cheaper tariff or be free. The validSchedule, on the other hand, restrict when a tariff is valid and usable based on when the parking started. The validSchedule is necessary when defining early bird tariffs, where a driver get cheaper rates if the driver starts parking inside a specific time period.

The time values (_startTime_, _endTime_, _validTimeFrom_, and _validTimeTo_) have a time accuracy in minutes, and are given in minutes past midnight. To prevent time values from clashing, some standard definitions exists. The _startTime_ and _validTimeFrom_ are defined as the exact second the minutes defines, for example, 60 (01:00) defines the start time 01:00:00. In contrast, _endTime_ and _validTimeTo_ are defined as 1 second before what the minutes defines, for example, 1380 (23:00) defines the end time 22:59:59.


#### ActiveSchedule Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|activeScheduleId|String(ID)|Unique ID|Required|
|startTime|Int|Inclusive start time of the paid hours |Required<br/>Min value: 0 <br />Max value: 1440|
|endTime|Int|Exclusive end time of the paid hours|Required<br/>Min value: 0 <br />Max value: 1440|
|days|Array(Days)|The days the tariff are active, i.e. days with parking fees |Required<br/>Only unique days|

Part of [Tariff](#tariff-object).
#### Active Schedule Examples
An activeSchedule might need more than one schedule unit to express the intention because the active time might vary per day. Therefore, each schedule requires a locally unique ID. For example, Active Schedule example shows a tariff is active during from 07:00 to 17:00 (16:59:59) weekdays and from 07:00 to 14:00 (13:59:59) Saturdays, and thus it requires at least two schedule units to describe. Note that there is no time defined for Sundays, hence, there is no parking fee during Sundays.

#### Active Schedule example
```
{"activeSchedules":[
  {"activeScheduleId":"as-0",
   "startTime":420, 
   "endTime":1020,
   "days": ["MONDAY","TUESDAY","WEDNESDAY","THURSDAY","FRIDAY"] },
  {"activeScheduleId":"as-1",
   "startTime":420, 
   "endTime":840,
   "days": ["SATURDAY"] },
]}
```

#### ValidSchedule Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|validScheduleId|String(ID)|Unique ID|Required|
|validFrom|DateTime|Date defining when the tariff starts being valid<br />i.e. can be used|Required|
|validTo|DateTime|Date defining when the tariff stop being valid<br />i.e. cannot be used anymore|Required|
|validTimeFrom|Int|The time when the tariff is valid, i.e. can be started|Required<br/>Min value: 0 <br />Max value: 1440|
|validTimeTo|Int|The time when the tariff is invalid, i.e. cannot be started|Required<br/>Min value: 0 <br />Max value: 1440|
|validDays|Arrays(Day)|The days the tariff can be started|Required<br/>Only unique days|

Part of [Tariff](#tariff-object).
#### Valid Schedule Examples

Like activeSchedules, multiple validSchedules might be necessary, and therefore locally unique IDs is required. For example: see Figure 16, which says that a tariff can be started between 1 January 2018 to 30 June 2018, Mondays to Fridays, from 00:00 to 09:00 (08:59:59), and from 21:00 to 00:00 (23:59:59). 

#### Valid Schedule example
```
{"validSchedules":[
  {"validScheduleId":"vs-0",
   "validFrom":"2018-01-01T00:00:00", 
   "validTo":"2018-06-30T23:59:59",
   "validTimeFrom":0, 
   "validTimeTo":540,
   "validDays": ["MONDAY","TUESDAY","WEDNESDAY","THURSDAY","FRIDAY"]
  },
  {"validScheduleId":"vs-1",
   "validFrom":"2018-01-01T00:00:00", 
   "validTo":"2018-06-30T23:59:59",
   "validTimeFrom":1260, 
   "validTimeTo":1440,
   "validDays": ["MONDAY","TUESDAY","WEDNESDAY","THURSDAY","FRIDAY"]
  }
]}
```

#### Day Tokens
|Token Name|Description|
|----------|-----------|
|MONDAY||
|TUESDAY||
|WEDNESDAY||
|THURSDAY||
|FRIDAY||
|SATURDAY||
|SUNDAY||
|HOLIDAY|Also known as red days|
|DAY_BEFORE_RED_DAY|Also know as day before holiday|

These tokens are also used for [Location](#schedule-object)

#### Log Object
|Element Name|Type|Description|Constraints|
|------|------|-------|-------|
|updated| DateTime|When the object was last updated|Required|
|user| String|Name/Username of the user that last updated the object|Required|
|created| DateTime|When the object was created|Required|
|creator| String|Name/Username of the creator of the object|Required|

The Log object is part of [Tariff](#tariff-object), [Location](#location-object) and [Occupancy](#occupancy-object)

## Location
The purpose of location data is to map parking-related information to a location. 
Mandatory for the location data is to include a globally unique identifier for each location.

TODO: determine all the required elements for every object under Location

### Location Object
|Element Name|Type|Description|Constraints|
|------|------|-------|-------|
|locationId|String(ID)|Unique ID|Required|
|parentLocation|String(ID)|If this area is a subarea, then this should be the ID of the parent area||
|sublocations|Array(String(ID))|If this is a parent area, then this should contain the IDs of the sub areas||
|name|String|Name or nickname of the area||
|areaNumber|String(ID)|Internal area numbering||
|address| Address|Object containing address info||
|contact| [Contact](#contact-object)|Object containing contanct info for the area||
|geoLocation|Array([GeoLocation](#geolocation-object))|Simple geospatial information regarding the area||
|polygons|Array([Polygon](#polygon-object))|Complex geospatial information regarding the area||
|auxiliary|[Auxiliary](#auxiliary-object)|Additional information of the area||
|log|[Log](#log-object)|Information regarding who created and upated the information the last|Required|

#### Address Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|street|String|Full street address including street number|Required|
|postalCode|String|Postal/zip code of the area|Required|
|city|String|City name|Required|
|region|String|State or county name||
|country|String|ISO-3166 Alpha-2 country codes|Required|

Part of [Location](#location-object).

#### Contact Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|contactName|String|Name of the contact person||
|organization|String|Name of the responsible organization|Required|
|email|String|Email to contact the responsible organization|Required|
|phoneNumber|String|Number to contact the responsible organization||

Part of [Location](#location-object).

### Geospatial Location
Locations may include geospatial coordinates, making it easier to find the location

#### GeoLocation Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|latitude|String|||
|longitude|String|||
|geoType|GeoType|Token describing what the geo-point defines||
|locationName|String|Nickname for the geo-point||

Part of [Location](#location-object).

#### GeoType Tokens
|Token Name|Description|
|----------|-----------|
|ENTRY|The point is a car entry|
|EXIT|The point is a car exist|
|ACCESS|The point is a people entrance|
|METER|The point is a parking meter|
|POINT|The point is a directional point|
|OTHER|Not in list, an update might be required!|

### Polygon
More advanced geographical data such as polygon markups in a map might be sought after. A commonly used format which provides such functionalities is KML. However, KML is a format based on XML with their own definition for file structures and might be hard to include in the JSON representation. However, some might want other formats than KML. Therefore, the format includes an object for attaching a reference providing the location of the polygon files. The polygon object also supports the use of encoded polygon data as an encrypted string.

#### Polygon Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|type|String|String defining what type of polygon this is||
|ref|String|URL to external polygon data||
|encoded|String|Encoded polygon data||
|description|String|Additional information about the polygon||

Part of [Location](#location-object).

### Auxiliary
Contains additional information regarding the parking area, such as currency, time zone, and operating hours.

#### Auxiliary Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|currency|String|ISO-4217 currency codes||
|timeZone|String|ISO 8601 Extended Notation (+\|-hh:mm)||
|public|Boolean|If the parking area is open to the public||
|paid|Boolean|If the parking area requires payment||
|locationType|LocationType|What type of parking area it is||
|operatingHours|Array(Schedule)|Operating hours of the parking area||
|surcharges|Surcharges|Additional charge information||

Part of [Location](#location-object).
#### LocationType Tokens
|Token Name|Description|
|----------|-----------|
|ABOVE_GROUND_GARAGE||
|UNDERGROUND_GARAGE||
|ON_STREET||
|SURFACE_LOT||
|PRIVATE|Privately owned parking lot|
|OTHER|Not in list, which means an update is required!|

### Schedule
Is similar to the active [schedule](#active-schedule-object) object of the tariff.

#### Schedule Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|scheduleId|String(ID)|Unique ID||
|startTime|Int|Opening time|Min value: 0 <br /> Max value: 1440|
|endTime|Int|Closing time|Min value: 0 <br /> Max value: 1440|
|days|Array([Day](#day-tokens))|Days the parking area is open||
|date|Date|Specific date the parking area is open||

### Surcharges
Might need to add more elements to surcharges

#### Surcharges Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|taxIncluded|Boolean|If taxes is included in the price||
|tax|Float|The tax percentage||
|other|Float|Additional fee amount||

Part of [Location](#location-object).

## Occupancy
Occupancy data are the information that describes the occupancy status of a parking location. Occupancy includes information about how many parking spaces there are in total (supply) and how many that are occupied (both as a percentage and a total number). Occupancy data also includes the elements: average occupancy, the probability of finding a parking space, and individual parking space information. The purpose of the average occupancy information is to give a major idea of how well a pricing strategy has performed and the occupancy status. Like average occupancy, the purpose of probability element is to give a major idea of the current occupancy status. The purpose of individual parking space data is to give an accurate occupancy status.

TODO: Determine what occupancy approach is best, those elements should be set as required.
TODO: Decide if average occupancy should be updated per day, as update frequency, or something else.
TODO: Determine what update frequency should be provided in for unit (seconds?, minutes?, or days?)

### Occupancy Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|locationI|ID|ID of the parking area||
|supply|Int|Total number of parking spaces||
|occupied|Int|Number of parking spaces that is currently occupied||
|occupiedPct|Int|Number of parking spaces occupied in percentage|Min value: 0 <br /> Max value: 100|
|average|Int|Average occupation in percentage|Min value: 0 <br /> Max value: 100|
|probability|Int|The probability of finding a parking spot|Min value: 0 <br /> Max value: 100|
|updateFrequency|Int|How often occupancy data is updated<br/> 30 -> once every 30 sec||
|detectionMethod|DetectionType|How occupancy data is collected||
|parkingSpace|Array(ParkingSpace)|Information regarding individual parking spaces||
|log|[Log](#log-object)|When this object was created and when the static info was updated last|Required|

#### DetectionType tokens
|Token Name|Description|
|----------|-----------|
|MANUALLY|The occupancy data is measured manually by workers|
|SPACE_SENSOR|Individual space sensor (many kinds)|
|VIDEO_COUNT|A camera is counting cars that passes|
|IMAGE_ANALYTIC|Still images are analyzed|
|VIDEO_ANALYTIC|Live images are analyzed|
|CROWD_SOURCING|The occupancy data is provided from multiple sources|

#### ParkingSpace Object
|Element Name|Type|Description|Constraints|
|------|------|-------|------|
|spaceId|String(ID)|Unique ID||
|spaceNumber|String(ID)|Local space numbering||
|available|Boolean|If the parking space is available||
|frequency|Int|How often the status is updated<br/> 30 -> once every 30 sec|Min value: 0|
|occupiedFrom |DateTime|When the occupation started||
|occupiedTo|DateTime|When the occupation is expected to end||
|spaceType| SpaceType|What kind of space it is||
|updated| DateTime|When static info was updated last||

Part of [Occupancy](#occupancy-object).
#### SpaceType
|Token Name|Description|
|----------|-----------|
|RESERVED||
|UNRESERVED||
|RESERVATION||
|EVENT||
|VALET||
|OTHER||

