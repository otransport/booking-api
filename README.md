# Booking-API

Booking API

Booking API provides clients the ability to, from other platforms, create and retrieve information into/from our system about bookings. It will allow client platforms to book rides.

### Introduction

Clients should configure all the _Settings_ are first on our platform. Fleets, Fares, Operational Areas shoud be configured inside ShiftDispatch.

All the requests should obey the format: ```https://booking-api.shiftdispatch.com/:resource[/:action[?:params]]```
 
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
    ```...?format=csv```
  - ```Content-Type```: Content-Types for PUT / POST requests. We support application/json:
    ```Content-Type: application/json```
  - ```Version```: we are supporting v1 only at the present time. This is not mandatory, so if not present last version will be used. Example:
    ```Version: v1```

### Authentication

We only accept *HTTPS* requests.

Authentication is done on every request by sending a token as an Authorization header:
  ```curl https://booking-api.shiftdispatch.com \
     -H "API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1" \
     -H "Authorization: QjFsnEBXMVejoet0GBOQSZKMk03jfnf3v9u=="```

Authorization token can be requested via programmatically using basic authentication through ```/auth```.

### Operations

#### POST /auth

Requests an authorization token by posting using _API-Key_. Upon successful authentication a new token will be returned which should be used for future requests. Old tokens will not be usable anymore. Access tokens shouldn’t by default expire but it might become invalid over time. Using an invalid token in requests will result in ```401 Unauthorized``` responses and you should in that case re-issue a request for a new token.

Example request:

  ```curl -X POST -d ‘’ https://booking-api.shiftdispatch.com/auth \
     -H "API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1" \
     -H 'User-Agent: MyProBookings.com’ \
     -H 'From: admin@myprobookings.com’ \
     -H 'Accept: application/json ’ \
     -H 'Content-type: application/json’```
 
Example response:

  ```HTTP/1.0 201 Created Content-Type: application/json; charset=utf-8  
  {
    "token": "QjFsnEBXMVejoet0GBOQSZKMk03jfnf3v9u==",
    "created_at": "2015-10-01T09:00:00Z",
    " email":"admin@myprobookings.com" 
  }```
 
#### GET /endpoint

Description... 

  - Request parameters
  All parameters are optional unless specified otherwise.

  - Response attributes
  Description…..

  - Ref
  Description

  - Example request

  ```curl -X POST -d ‘’ https://booking-api.shiftdispatch.com/ \
     -H "API-Key: 00379cdef1274bd2dd3cb6d9fccab3e9e5c6ac8594dd8ae5b2839843dc0ca1c1" \
     -H 'User-Agent: MyProBookings.com’ \
     -H 'From: admin@myprobookings.com’ \
     -H 'Accept: application/json ’ \
     -H 'Content-type: application/json’```

  - Example response

  ```HTTP/1.0 200 OK Content-Type: application/json; charset=utf-8 

    {
      "": "",
    }```


### end

Use of the shiftdispatch API is conditioned by acceptance of the terms of use ().
Questions? Please email: api@shiftdispatch.com

Revised Thu Nov 1 15:12:04 2015.
