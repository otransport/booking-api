# Booking-API

Booking API provides clients the ability to, from other platforms, create and retrieve information into/from our system about bookings. It will allow client platforms to book rides.

### Introduction

Clients should configure all the _Settings_ are first on our platform. Fleets, Fares, Operational Areas shoud be configured inside ShiftDispatch.

All the requests should obey the format: 
```
  https://booking-api.shiftdispatch.com/:resource[/:action[?:params]]
```
 
All timestamps are represented as _ISO 8601_ format _YYYY-MM-DDTHH:MM:SSZ_.

Example requests are constructed using _cURL_ (https://en.wikipedia.org/wiki/CURL) commands via command line interface. There are online tools like https://www.hurl.it/ where one can generate complete HTTP requests also.

We support two types of Bookings at the moment: *Regular* and *noDropOff*.

  - *Regular* Bookings require a Passenger name and phone or email address. They also require a Pickup and Drop Off address.
  - *noDropOff* Bookings require a Passenger name and phone or email address. They also require a Pickup address. This is similar to a conventional taxi mode.

Pricing will be calculated based on Fare definitions (including Operational Areas). Only after a booking is successfully created, a Dispatch can occur. 

### Headers

  - ```User-Agent```: your platform name. Example: 

    ```User-Agent: MyProBookings.com```

  - ```API-Key```: this is the secret API key. A client can get his API-Key on the settings area inside our platform. Once an API-Key is created, will not change. Example: 

    ```API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1```

  - ```Authorization```: authentication credential for HTTP authentication. You have some control over your Authorization, namely you can reset it so that a new one will be sent to the email. Example: 

    ```Authorization: QjFsnEBXMVejoet0GBOQSZKMk03jfnf3v9u==```

  - ```From```: the email address of the client making the request. Example:

    ```From: user@example.com```

  - ```Accept```: Content-Types that are acceptable for the response. Non mandatory. We support application/json:

    ```Accept: application/json```

    We may however return other formats, if the request accepts them. The header will then be updated accordingly. Example: 

    [endpoint]?format=csv

  - ```Content-Type```: Content-Types for PUT / POST requests. We support application/json:

    ```Content-Type: application/json```

  - ```Version```: we are supporting v1 only at the present time. This is not mandatory, so if not present last version will be used. Example:

    ```Version: v1```

### Authentication

We only accept *HTTPS* requests.

Authentication is done on every request by sending a token as an Authorization header:
```
  curl https://booking-api.shiftdispatch.com \

    -H "API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1" \

    -H "Authorization: QjFsnEBXMVejoet0GBOQSZKMk03jfnf3v9u=="
```

Authorization token can be requested via programmatically using basic authentication through ```/auth```.

### Operations

#### POST /auth

Requests an authorization token by posting using _API-Key_. Upon successful authentication a new token will be returned which should be used for future requests. Old tokens will not be usable anymore. Access tokens shouldn’t by default expire but it might become invalid over time. Using an invalid token in requests will result in ```401 Unauthorized``` responses and you should in that case re-issue a request for a new token.

Example request:

```
  curl -X POST -d ‘’ https://booking-api.shiftdispatch.com/auth \
    -H 'API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1' \
    -H 'User-Agent: MyProBookings.com’ \
    -H 'From: admin@myprobookings.com’ \
    -H 'Accept: application/json’ \
    -H 'Content-type: application/json’
```
 
Example response:

```
  HTTP/1.0 201 Created; Content-Type: application/json; charset=utf-8  

	{
		"token": "QjFsnEBXMVejoet0GBOQSZKMk03jfnf3v9u==",
		"created_at": "2015-10-01T09:00:00Z",
		"email":"admin@myprobookings.com" 
	}
```
 

### GET /fleet

Retrieve the Fleet information.

  - Request parameters
	Request (application/json)
```
		Header
			API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1
			Authorization: QjFsnEBXMVejoet0GBOQSZKMk03jfnf3v9u==
			User-Agent: MyProBookings.com
			From: admin@myprobookings.com
			Accept: application/json
			Content-type: application/json
		Body
```

  - Response attributes
	Response (application/javascript)
```
		Header
		Body
			{
			    "person": {},
			    "stripe": {},
			    "id": 5112317826039808,
			    "name": "MyProBookings",
			    "description": "MyProBookings Fleet",
			    "email": "admin@myprobookings.com",
			    "mobile_phone_number": "+1234567890",
			    "work_phone_number": "+1234567890",
			    "address": "Fleet Address 1",
			    "address2": "Fleet Address 2",
			    "city": "London",
			    "state": "",
			    "postal_code": "EC1A1AH",
			    "country": "UK",
			    "location_lat": 52.5167,
			    "location_lng": 13.3833,
			    "timezone": "CET",
			    "payment_method_id": 0,
			    "vat": "20",
			}
```

  - Example request

```
  curl -X GET https://booking-api.shiftdispatch.com/fleet \
    -H 'API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1' \
    -H 'Authorization: QjFsnEBXMVejoet0GBOQSZKMk03jfnf3v9u==' \
    -H 'User-Agent: MyProBookings.com’ \
    -H 'From: admin@myprobookings.com’ \
    -H 'Accept: application/json ’ \
    -H 'Content-type: application/json’
```

  - Example response

```
  HTTP/1.0 200 OK; Content-Type: application/json; charset=utf-8 

	{
		"": "",
	}
```

### Bookings

Create a new Booking. This action allows a booking to be created and a quote and estimated times to be sent in the response.

#### POST /quotes

The fields "origin" and "destination" are both strings, they can contain either an address which Google Direction API can understand, or a pair of longitude and latitude like "12.3456789,98.7654321".

  - Request parameters
	Request (application/json)
´´´
		Header
			API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1
			Authorization: QjFsnEBXMVejoet0GBOQSZKMk03jfnf3v9u==
			User-Agent: MyProBookings.com
			From: admin@myprobookings.com
			Accept: application/json
			Content-type: application/json
		Body
			{
				"pickup_time":"2015-07-13T11:14:25.434105125+02:00",
				"dropoff_time":"2015-07-13T11:14:25.434105159+02:00",
				"origin":"London Borough of Hillingdon, United Kingdom",
				"destination":"London Heathrow Airport, Longford, United Kingdom",
				"type": "regular",
				"passenger": 23456789101,
				"full_name": "John F. Doe",
				"email": "johnfdoe@example.com",
				"phone_number":"+1234567890",
				"payment":{
					"type":"pre",
					"method":"credit",
					"options":{
						"cash":{
							"pay_to":""
						},"credit":{
							"provider":"braintree",
							"last4":"1234"
						}
					}
				}
			}
´´´

  - The response will contain the list of available drivers, the driver's vehicles, and quotes for each driver+vehicle pair.

  - Response attributes
	Response (application/javascript)
´´´
			{
				"booking_id":6078271003295744,
				"ride_id":4590253276921856,
				"candidates":
					[{
						"driver":{
						    "id":5631986051842048,
						    "name":"Benjamin Westrup",
						    "logo_url":"https://storage.googleapis.com/shift-driver-api-dev-drivers-photo/photo_driver_5631986051842048.jpg?v=1436780731",
						    "logo_url_small":"https://lh3.googleusercontent.com/afafXUesTx_8I7WXM51Ts3oCh-PgrX_47zjRQQPL6uIoU5lpH7Q1VsCPAfMwfFjsf1OE1yP4DBRtMxyAz7YQRfM4QSeOx2JjVg=s200",
						    "mobile_phone_number":"0118923442",
						    "work_phone_number":"12342345",
						    "location_lat":54.123,
						    "location_lng":56.123,
						    "distance_mins":120,
						},
						"vehicles":[{
						    "vehicle_id":5644406560391168,
						    "owner":5649050225344512,
						    "make":"BMW",
						    "model":"5 Series",
						    "energy_type":1,
						    "vehicle_type":1,
						    "photo_url_small":"http://...",
						    "photo_url_medium":"http://...",
						    "photo_url_big":"http://...",
						    "quote":{
						    "fleet":5649050225344512,
						    "energy_type":1,
						    "vehicle_type":1,
						    "organisation":0,
						    "team":0,
						    "passenger":5642554087309312,
						    "kms":0,
						    "hours":0,
						    "hours_waiting":0,
						    "hiring":false,
						    "fare": {
						        "minimum_price":0,
						        "starting_price":0,
						        "per_km_price":0,
						        "per_hour_price":0,
						        "per_hour_waiting_price":0,
						        "per_hour_hiring_price":0,
						        "tax":0,
						        "energy_type_markups":null,
						        "vehicle_type_markups":null,
						        "Cancellation":{
						                        "type":"",
						                        "value":0
						                       }
						    },
						    "currency":"EUR",
						    "total":25.588
						    }
						}]
					}],
				"quote":{
				    "fleet":5672749318012928,
				    "energy_type":0,
				    "vehicle_type":0,
				    "organisation":0,
				    "team":0,
				    "passenger":23456789101,
				    "kms":540.951,
				    "hours":5.283333333333333,
				    "hours_waiting":0,
				    "hiring":false,
				    "fare":{"minimum_price":0,"starting_price":0,"per_km_price":0,"per_hour_price":0,"per_hour_waiting_price":0,"per_hour_hiring_price":0,"tax":0,"energy_type_markups":null,"vehicle_type_markups":null,"Cancellation":{"type":"","value":0}},
				    "currency":"EUR",
				    "total":100.1234
				}
			}
```

  - To confirm a booking created this way use the PUT /bookings/:booking_id/:status endpoint. Only "pending" status is accepted at the moment.



### Dispatch

Dispatch assigns the chosen driver and car to the booking (and ride), and notifies the driver (through the dispatch MS).

#### POST /dispatch

  - Request parameters
	Request (application/json)
```
		Header
			API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1
			Authorization: QjFsnEBXMVejoet0GBOQSZKMk03jfnf3v9u==
			User-Agent: MyProBookings.com
			From: admin@myprobookings.com
			Accept: application/json
			Content-type: application/json
		Body
			{
				"driver_id": 5634612826996736,
				"booking_id": 5354664308506624,
				"ride_id": 6580146253332480,
				"vehicle_id": 5717271485874176
			}
```

#### PUT /rides/:id/undispatch

Undispatches a Ride. This endpoint may be removed in future since this function should be performed by the backend.

  - Request parameters
	Request (application/json)
```
		Header
			API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1
			Authorization: QjFsnEBXMVejoet0GBOQSZKMk03jfnf3v9u==
			User-Agent: MyProBookings.com
			From: admin@myprobookings.com
			Accept: application/json
			Content-type: application/json
		Body
			{
			}
```

The response will return the Ride information, and a HTTP 200OK:

  - Response attributes
	Response (application/javascript)
```
			{
			    booking = 5354664308506624
			    created = "2015-10-23T15:24:01.820423Z"
			    currency = EUR
			    dispatch_type = ondemand
			    distance = 0
			    duration = 0
			    pickup =     {
			        delay = 0
			        lat = "46.141143"
			        lng = "1.249336"
			        location = "D1, 87290 Châteauponsac, France"
			        time = "2015-10-23T15:39:00.794Z"
			    }
			    dropoff =     {
			        delay = 0
			        lat = "46.1277419"
			        lng = "1.3212558"
			        location = "D203, 87290 Châteauponsac, France"
			        time = "2015-10-23T15:46:10.794Z"
			    }
			    "energy_type" = 0
			    "estimated_distance" = 5992
			    "estimated_duration" = 7
			    fare =     {
			        Cancellation =         {
			            type = fixed
			            value = 50
			        }
			        "energy_type_markups" = "<null>"
			        "minimum_price" = 5
			        "per_hour_hiring_price" = 0
			        "per_hour_price" = 55
			        "per_hour_waiting_price" = 0
			        "per_km_price" = 0
			        "starting_price" = 0
			        tax = 0
			        "vehicle_type_markups" =         (
			                        {
			                markup = 200
			                "vehicle_type" = 2
			            }
			        )
			        "waiting_buffer" = 0
			    }
			    fleet = 7833314331366460
			    flight_number = ""
			    handling_fleet = 0
			    hiring = 0
			    id = 6580146253332480
			    modified = "2015-10-23T15:24:16.065097578Z"
			    pob =     {
			        delay = 0
			        time = "0001-01-01T00:00:00Z"
			    }
			    status = pending
			    "status_ongoing_stage" = ""
			    "taken_by" =     {
			        email = "anna_brown@mail.com"
			        "full_name" = "Anna Brown"
			        organisation = 0
			        passenger = 6196444344090624
			        "phone_number" = 0712345678
			        team = 0
			    }
			    total = "6.416666666666667"
			    "vehicle_type" = 0
			}
```


### Driver Vehicle Status

#### GET /drivers?ride_id=6580146253332480

It is possible to retrieve a new list of drivers and quotes for an existing ride knowing the ride's ID.
ride_id can not be empty or an invalid value for parsing as int64.

  - Request parameters
	Request (application/json)
```
		Header
			API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1
			Authorization: QjFsnEBXMVejoet0GBOQSZKMk03jfnf3v9u==
			User-Agent: MyProBookings.com
			From: admin@myprobookings.com
			Accept: application/json
			Content-type: application/json
		Body
```
	Response (application/javascript)
```
		Header
		Body
			[{
				"driver": {
					"id": 4899148398592000,
					"name": "John Driver Runway",
					"logo_url": "",
					"logo_url_small": ""
				},
				"vehicles": [
					{
						"vehicle_id": 5671831268753408,
						"make": "Volkswagen",
						"model": "Phaeton",
						"energy_type": 1,
						"vehicle_type": 8,
						"quote": {
							"fleet": 5081359164899328,
							"energy_type": 1,
							"vehicle_type": 8,
							"passenger": 5674368789118976,
							"kms": 25.702,
							"hours": 34.21666666666667,
							"hours_waiting": 0,
							"currency": "EUR",
							"total": 38.782799999999995
						}
					}
				]
			  }]
```

#### GET  /status/vehicle/:vehicle_id/driver/:driver_id 

Gets the present status and location for a drivers vehicle

  - Request parameters
	Request (application/json)
```
		Header
			API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1
			Authorization: QjFsnEBXMVejoet0GBOQSZKMk03jfnf3v9u==
			User-Agent: MyProBookings.com
			From: admin@myprobookings.com
			Accept: application/json
			Content-type: application/json
		Body
```

  - Response attributes
	Response (application/javascript)
```
		Header
		Body
			{  
				"vehicle_id":5646239437684736,
				"driver_id":5139717033033728,
				"channel":"5139717033033728,5139717033033728.Driver",
				"timestamp":1439911775,
				"lat":52.5132748,
				"lng":13.4201159,
				"driver_available":true,
				"status":2,
				"name":"Mat Driver Jonhson",
				"logo_url":"https://storage.googleapis.com/shift-driver-api-drivers-photo/photo_driver_5139717033033728?v=1439899064",
				"logo_url_small":"https://lh3.googleusercontent.com/n2KSvsZSbV59oSFUdHfMSYSQbocDz3MljW5LPKg_SDogYaFRABwsKMA7iWBBFucUtmbd5RMqkmxznZmoDa5dr7MFtVRUcmQFZA=s200"
			}
```


### Passengers 

#### GET /passengers

Retrieve a list of the passengers

	Request (application/json)
```
		Header
			API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1
			Authorization: QjFsnEBXMVejoet0GBOQSZKMk03jfnf3v9u==
			User-Agent: MyProBookings.com
			From: admin@myprobookings.com
			Accept: application/json
			Content-type: application/json
		Body
```

	Response (application/javascript)
```
		Header
		Body
			[{
			    "id": 5664248772427776,
			    "created": "2015-02-02T15:22:24.461241Z",
			    "modified": "2015-02-11T15:22:57.712602Z",
			    "email": "merlin@cat.com",
			    "first_name": "Merlin",
			    "last_name": "Baracoa",
			    "fleet_id": 5112317826039808,
			    "organisation_id": 5720147234914304,
			    "team_id": -1,
			    "mobile_phone_number": "+361234567890"
			}, {
			    "id": 5687539843203072,
			    "created": "2015-02-02T15:22:39.854509Z",
			    "modified": "2015-02-02T15:22:39.854509Z",
			    "email": "merlin2@cat.com",
			    "first_name": "Merlin",
			    "last_name": "Baracoa Jr.",
			    "fleet_id": 5112317826039808,
			    "mobile_phone_number": "+361234567890"
			}]
```

#### GET /passengers?passenger={"field":"value"}

Retrieve a filtered list of the fleet's passengers. Takes url encoded passenger as parameter: /query/passenger?passenger={"field":"value"}

	Request (application/json)
```
		Header
			API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1
			Authorization: QjFsnEBXMVejoet0GBOQSZKMk03jfnf3v9u==
			User-Agent: MyProBookings.com
			From: admin@myprobookings.com
			Accept: application/json
			Content-type: application/json
		Body
```

	Response (application/javascript)
```
		Header
		Body
			[{
			    "id": 5664248772427776,
			    "created": "2015-02-02T15:22:24.461241Z",
			    "modified": "2015-02-11T15:22:57.712602Z",
			    "email": "merlin@cat.com",
			    "first_name": "Merlin",
			    "last_name": "Baracoa",
			    "fleet_id": 5112317826039808,
			    "organisation_id": 5720147234914304,
			    "team_id": -1,
			    "mobile_phone_number": "+361234567890"
			}, {
			    "id": 5687539843203072,
			    "created": "2015-02-02T15:22:39.854509Z",
			    "modified": "2015-02-02T15:22:39.854509Z",
			    "email": "merlin2@cat.com",
			    "first_name": "Merlin",
			    "last_name": "Baracoa Jr.",
			    "fleet_id": 5112317826039808,
			    "mobile_phone_number": "+361234567890"
			}]
```


#### GET /passengers/:id
Retrieve a single passenger

	Request (application/json)
```
		Header
			API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1
			Authorization: QjFsnEBXMVejoet0GBOQSZKMk03jfnf3v9u==
			User-Agent: MyProBookings.com
			From: admin@myprobookings.com
			Accept: application/json
			Content-type: application/json
		Body
```

	Response (application/javascript)
```
		Header
		Body
			{
			    "id": 5664248772427776,
			    "created": "2015-02-02T15:22:24.461241Z",
			    "modified": "2015-02-11T15:22:57.712602Z",
			    "email": "merlin@cat.com",
			    "first_name": "Merlin",
			    "last_name": "Baracoa",
			    "fleet_id": 5112317826039808,
			    "organisation_id": 5720147234914304,
			    "team_id": -1,
			    "mobile_phone_number": "+361234567890"
			}
```

#### POST /passengers

Create new passenger Object (for internal use). New Passenger is sent with Booking.


#### PUT /passengers/:id

Update a passenger information. The payload contains the properties to update.

	Request (application/json)
```
		Header
			API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1
			Authorization: QjFsnEBXMVejoet0GBOQSZKMk03jfnf3v9u==
			User-Agent: MyProBookings.com
			From: admin@myprobookings.com
			Accept: application/json
			Content-type: application/json
		Body
			{
				"first_name": "Merlin",
				"last_name": "Baracoa",
				"organisation_id": 5720147234914304,
				"team_id": -1,
				"mobile_phone_number": "+361234567890"
			}
```

	Response (application/javascript)
```
		Header
		Body
			{
				"id": 5664248772427776,
				"created": "2015-02-02T15:22:24.461241Z",
				"modified": "2015-02-11T15:22:57.712602Z",
				"email": "merlin@cat.com",
				"first_name": "Merlin",
				"last_name": "Baracoa",
				"fleet_id": 5112317826039808,
				"organisation_id": 5720147234914304,
				"team_id": -1,
				"mobile_phone_number": "+361234567890"
			}
```

#### DELETE /passengers/:id
Remove passenger

	Request (application/json)
```
		Header
			API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1
			Authorization: QjFsnEBXMVejoet0GBOQSZKMk03jfnf3v9u==
			User-Agent: MyProBookings.com
			From: admin@myprobookings.com
			Accept: application/json
			Content-type: application/json
		Body
			{
				"first_name": "Merlin",
				"last_name": "Baracoa",
				"organisation_id": 5720147234914304,
				"team_id": -1,
				"mobile_phone_number": "+361234567890"
			}
```
	Response (application/javascript)
```
		Header
		Body
			{}
```


### end

Use of the shiftdispatch API is conditioned by acceptance of the terms of use ().
Questions? Please email: api@shiftdispatch.com

Revised Thu Nov 1 15:12:04 2015.
