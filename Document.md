# Lok API Reference

> Version 0.28.0

# Overview

The Lok API is organized around [REST](http://en.wikipedia.org/wiki/Representational_State_Transfer),
having resource-oriented URLs, accepts and returns [JSON-encoded](http://www.json.org/) bodies,
and uses standard HTTP response codes, authentication and verbs. Dates are in [UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time) time standard.

This API allows the integration of your company systems with the rental of lockers:

- Get lockers locations and its availability
- Manage reservations on your lockers
- Get usage reports
- Manage notifications settings

# Authentication

Lok API uses [OAuth](http://tools.ietf.org/html/rfc6749) scheme, using access tokens to
 authenticate requests. When you request access to Lok API, you will get a `client_id` and a `client_secret`.
With these credentials you can [get an access token](#operation/postOauthToken) which should be
included in each request:

```shell
GET /api/v2/some_resource HTTP/2
Host: api.clicknbox.com
Authorization: Bearer youraccesstoken 
```

> Your client credentials grant access for a trusted entity, so make sure you store them securely.

All API requests must be made over [HTTPS](http://en.wikipedia.org/wiki/HTTP_Secure). Calls made
over plain HTTP will fail. API requests without authentication will also fail.

## User scopes

An access token has the scope of it's user. Each route has the following security scheme:

<!-- ReDoc-Inject: <security-definitions> -->

Every scope has the following allowed actions:

- **admin** is the Lok administrator. Has read access to all reservations and lockers.
- **owner** is the top-level user of the company. This scope grants access to all the lockers that are
managed by your company.
- **employee** grants access to one location only.
- **reporter** grants read access to the lockers and reservations managed by your company.
- **seller** similarly as an _owner_, is a top-level user who has access to Lok public network.

# Errors

Lok uses conventional [HTTP response codes](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html) to indicate the
success or failure of an API request. However, the field `status_code` is always returned
for redundancy. 

Responses with status other than `2xx` are error responses: 

- codes in the `4xx` range indicate an error related to information provided
- codes in the `5xx` range indicate an error with Lok servers.

Whenever you get an error response, it will have the following properties:

- `type` (_string_): The type of error returned.
- `message` (_string_): A human-readable message providing more details about the error.
- `data` (_object_): Some error types include additional details of the error.

**Response example**

```json
{
  "status_code": 404,
  "type": "ENTITY_NOT_FOUND",
  "message": "Location not found",
  "data": {
     "id": "11446958822f"
  }
}
```

## Error Types

### Request errors

| Type                 | Description                |
| ---------------------|----------------------------|
| `NOT_FOUND`            | Requested resource was not found                                            |
| `ENTITY_NOT_FOUND`     | Requested entity was not found                                              |
| `INVALID_REQUEST`      | Request was invalid: missing fields, wrong types or is semantically wrong |

### Authorization errors

| Type                 | Description                |
| ---------------------|----------------------------|
| `GRANT_TOKEN`          | The Lok grant for the operation is invalid       |
| `INSUFFICIENT_SCOPE`   | Authorization is insufficient |
| `INVALID_CLIENT`       | Client authentication failed |
| `INVALID_GRANT`        | Authorization is invalid, expired or revoked |
| `INVALID_TOKEN`        | Provided token is expired, revoked or invalid |
| `UNAUTHORIZED_CLIENT`  | The client is not authorized to use requested grant |

### Reservation errors

| Type                 | Description                |
| ---------------------|----------------------------|
| `INVALID_RESERVATION`  | Requested operation cannot be applied to the reservation |
| `UNIQUE_SHIPMENT`      | Provided shipment id is already used within your company |
| `BOX_INCOMPATIBLE`     | Provided package dimensions are incompatible with our box types |
| `NO_CAPACITY`          | The location doesn't have a box available at the moment|
| `LOCATION_LOCKED`      | The location is temporarily locked |
| `INVALID_DUE_DATE`     | Provided delivery date is invalid or in the past |

### Logistic errors
| Type                 | Description                |
| ---------------------|----------------------------|
| `INVALID_ADDRESS`  | Provided address is invalid, too abstract or has multiple matches |


### Lok server errors

| Type                 | Description                |
| ---------------------|----------------------------|
| `REQUEST_TIMEOUT`      | We were unable to sync the data with the locker |
| `UNKNOWN_ERROR`        | An unexpected exception on Lok servers |

### Third-party errors

| Type                 | Description                |
| ---------------------|----------------------------|
| `FAILED_DEPENDENCY`  | We were unable to complete the transaction with an external vendor |

# Webhook

A Webhook is a callback URL to which our systems send the different events related to your reservations. On each call you can
notify your user or operations team and create reports of events.

You can [manage your company's webhook](/#tag/webhook) settings and subscribe to specific events only or
enable/disable it. When adding your webhook for the first time, a secret will be generated
that you can use later to validate incoming requests.

## Set up a webhook

Data will be sent via HTTP POST method in JSON format and UTF-8 encoding. To confirm that you received a 
notification successfully, your server must return a 200 (OK) HTTP status code, any other status will be
treated as an error and the request will be retried up to 50 times in increasing intervals.

Example with javascript:

```
var data = typeof req.body == 'string' ? JSON.parse(req.body) : req.body;

if (data.status == 'picked_up') {
var reservation_id = data.reservation_id;
//...
}
```

## Notification payload

The information of the notification will have the following structure:

```
{
  "reservation_id": "5ccc9e33a97e2f8226ba36d2",
  "shipment_id": "Order 557",
  "status": “delivered”,
  "status_message": "Package received in the locker",
  "t": 1614202849096,
}
```

**reservation_id**
- Type: String
- Description: The reservation identifier that should be used in subsequent API calls

**shipment_id**
-   Type: String
-   Description: Identifier that the company assigned to the reservation

**status**
-   Type: String
-   Description: Current status of the reservation. Possible values are:
    - _reserved_: Reservation creation confirmation. This event is triggered whenever a new reservation is created or a pre-reservation is confirmed.
    - _undelivered_: Package waiting to be delivered
    - _delivered_: Package delivered in the locker
    - _unpicked_: Package waiting to be picked up
    - _reopened_: Package first alteration in the locker
    - _reopened2_: Second package alteration in the locker
    - _cancelled_: Reservation cancelled confirmation. This event is triggered either with an API cancellation or by using the cancellation token.
    - _picked_up_: Package was picked up

**status_message**
-   Type: String
-   Description: Human-readable description of the status

**t**
- Type: Number
- Description: Event triggered timestamp

> The `undelivered` and `unpicked` events are notified daily until the package is
> delivered or picked up, respectively.

## Notification signature

To verify that the request came from our servers, you can validate the body using
the `secret` granted upon webhook creation:

```js
const crypto = require('crypto');

const hmac = crypto.createHmac('sha256', YOUR_WEBHOOK_SECRET);

hmac.update(req.body);
const signature = hmac.digest('base64');
```

and compare `signature` against `Lok-Signature` header.


## Path Table

| Method | Path | Description |
| --- | --- | --- |
| GET | [/company/action_logs](#getcompanyaction_logs) | Activity log |
| GET | [/company/directories](#getcompanydirectories) | Get directories |
| POST | [/company/directories](#postcompanydirectories) | Create directory |
| GET | [/company/locations](#getcompanylocations) | Get locations |
| GET | [/company/notification_emails](#getcompanynotification_emails) | Get company notifications emails |
| PUT | [/company/notification_emails](#putcompanynotification_emails) | Edit company notifications emails |
| GET | [/company/pre_reservations](#getcompanypre_reservations) | Get all company pre-reservations |
| GET | [/company/reservations](#getcompanyreservations) | Get all company reservations |
| GET | [/company/webhook](#getcompanywebhook) | Get webhook settings |
| PUT | [/company/webhook](#putcompanywebhook) | Update webhook settings |
| GET | [/oauth/token](#getoauthtoken) | Validates token |
| POST | [/oauth/token](#postoauthtoken) | Get an access token |
| DELETE | [/oauth/token](#deleteoauthtoken) | Revoke an access token |
| GET | [/oauth/user](#getoauthuser) | Get authenticated user |
| GET | [/reservations/{reservation_id}](#getreservationsreservation_id) | Get a reservation |
| PUT | [/reservations/{reservation_id}](#putreservationsreservation_id) | Update a pre-reservation |
| DELETE | [/reservations/{reservation_id}](#deletereservationsreservation_id) | Cancel a reservation |
| GET | [/shipping/reservations](#getshippingreservations) | Get reservations with shipment |
| GET | [/company/action_logs/csv](#getcompanyaction_logscsv) | Request report of user activity |
| GET | [/company/directories/{directory_id}](#getcompanydirectoriesdirectory_id) | Get a directory |
| PUT | [/company/directories/{directory_id}](#putcompanydirectoriesdirectory_id) | Update directory |
| GET | [/company/locations/{location_id}](#getcompanylocationslocation_id) | Get a location |
| GET | [/company/reports/recent_activity](#getcompanyreportsrecent_activity) | Get company recent activity |
| GET | [/locations/{location_id}/reservations](#getlocationslocation_idreservations) | Get location reservations |
| POST | [/locations/{location_id}/reservations](#postlocationslocation_idreservations) | Create a reservation |
| GET | [/reservations/{reservation_id}/label](#getreservationsreservation_idlabel) | Get reservation shipping label |
| POST | [/shipping_order](#postshipping_order) | Create a shipping order getting a label as response |
| POST | [/locations/available](#postlocationsavailable) | Locations for pre-reservations |
| POST | [/company/directories/csv](#postcompanydirectoriescsv) | Create directory |
| POST | [/locations/{location_id}/pre_reservation](#postlocationslocation_idpre_reservation) | Create a pre-reservation |
| POST | [/locations/{location_id}/shipping](#postlocationslocation_idshipping) | Create a shipping order |
| POST | [/reservations/{reservation_id}/shipping](#postreservationsreservation_idshipping) | Request shipment |
| POST | [/reservations/{reservation_id}/shipping/quotation](#postreservationsreservation_idshippingquotation) | Quote shipment |
| PUT | [/company/webhook/status](#putcompanywebhookstatus) | Toggle webhook status |
| PUT | [/company/directories/{directory_id}/contacts](#putcompanydirectoriesdirectory_idcontacts) | Overwrite contacts |

## Reference Table

| Name | Path | Description |
| --- | --- | --- |
| Model24 | [#/components/requestBodies/Model24](#componentsrequestbodiesmodel24) |  |
| oauth2 | [#/components/securitySchemes/oauth2](#componentssecurityschemesoauth2) |  |
| Model1 | [#/components/schemas/Model1](#componentsschemasmodel1) |  |
| action_logs | [#/components/schemas/action_logs](#componentsschemasaction_logs) |  |
| Model2 | [#/components/schemas/Model2](#componentsschemasmodel2) |  |
| keys | [#/components/schemas/keys](#componentsschemaskeys) | Path of invalid parameters |
| data | [#/components/schemas/data](#componentsschemasdata) |  |
| Error | [#/components/schemas/Error](#componentsschemaserror) |  |
| Error1 | [#/components/schemas/Error1](#componentsschemaserror1) | Authorization is invalid |
| Error2 | [#/components/schemas/Error2](#componentsschemaserror2) | Authorization scope is insufficient |
| company | [#/components/schemas/company](#componentsschemascompany) |  |
| author | [#/components/schemas/author](#componentsschemasauthor) |  |
| DirectoryCompany | [#/components/schemas/DirectoryCompany](#componentsschemasdirectorycompany) |  |
| directories | [#/components/schemas/directories](#componentsschemasdirectories) |  |
| DirectoryListResponse | [#/components/schemas/DirectoryListResponse](#componentsschemasdirectorylistresponse) |  |
| Model3 | [#/components/schemas/Model3](#componentsschemasmodel3) |  |
| boxes | [#/components/schemas/boxes](#componentsschemasboxes) | Available boxes in locker |
| LocationAddress | [#/components/schemas/LocationAddress](#componentsschemaslocationaddress) | Locker host address |
| Coordinates | [#/components/schemas/Coordinates](#componentsschemascoordinates) | Host location coordinates |
| ServiceDay | [#/components/schemas/ServiceDay](#componentsschemasserviceday) |  |
| service_days | [#/components/schemas/service_days](#componentsschemasservice_days) | Available days schedule |
| LocationCompany | [#/components/schemas/LocationCompany](#componentsschemaslocationcompany) |  |
| locations | [#/components/schemas/locations](#componentsschemaslocations) |  |
| LocationListResponse | [#/components/schemas/LocationListResponse](#componentsschemaslocationlistresponse) |  |
| management_emails | [#/components/schemas/management_emails](#componentsschemasmanagement_emails) |  |
| Model4 | [#/components/schemas/Model4](#componentsschemasmodel4) |  |
| location | [#/components/schemas/location](#componentsschemaslocation) |  |
| PackageInfo | [#/components/schemas/PackageInfo](#componentsschemaspackageinfo) | Package information provided |
| Model5 | [#/components/schemas/Model5](#componentsschemasmodel5) |  |
| EventHistory | [#/components/schemas/EventHistory](#componentsschemaseventhistory) | Reservation updates history |
| shipping | [#/components/schemas/shipping](#componentsschemasshipping) |  |
| Model6 | [#/components/schemas/Model6](#componentsschemasmodel6) |  |
| boxes1 | [#/components/schemas/boxes1](#componentsschemasboxes1) | Selected boxes. Only for P2P and Mailbox models |
| BoxInfo | [#/components/schemas/BoxInfo](#componentsschemasboxinfo) | Assigned box information |
| contact_info | [#/components/schemas/contact_info](#componentsschemascontact_info) | Contact person for any issue regarding the reservation. |
| recipient_info | [#/components/schemas/recipient_info](#componentsschemasrecipient_info) | Contact details of the final user |
| Reservation | [#/components/schemas/Reservation](#componentsschemasreservation) |  |
| reservations | [#/components/schemas/reservations](#componentsschemasreservations) |  |
| ReservationListResponse | [#/components/schemas/ReservationListResponse](#componentsschemasreservationlistresponse) |  |
| events | [#/components/schemas/events](#componentsschemasevents) |  |
| webhook | [#/components/schemas/webhook](#componentsschemaswebhook) |  |
| Model7 | [#/components/schemas/Model7](#componentsschemasmodel7) |  |
| Model8 | [#/components/schemas/Model8](#componentsschemasmodel8) | Access token granted |
| company1 | [#/components/schemas/company1](#componentsschemascompany1) |  |
| user | [#/components/schemas/user](#componentsschemasuser) | Authenticated user data |
| Model9 | [#/components/schemas/Model9](#componentsschemasmodel9) |  |
| tokens | [#/components/schemas/tokens](#componentsschemastokens) | Reservation tokens with QR |
| Model10 | [#/components/schemas/Model10](#componentsschemasmodel10) |  |
| Error3 | [#/components/schemas/Error3](#componentsschemaserror3) | Resource not found |
| pickup_point | [#/components/schemas/pickup_point](#componentsschemaspickup_point) |  |
| destination_point | [#/components/schemas/destination_point](#componentsschemasdestination_point) |  |
| shipping1 | [#/components/schemas/shipping1](#componentsschemasshipping1) | Shipping information |
| Reservation1 | [#/components/schemas/Reservation1](#componentsschemasreservation1) | Reservation with shipping |
| reservations1 | [#/components/schemas/reservations1](#componentsschemasreservations1) |  |
| PaginationResult | [#/components/schemas/PaginationResult](#componentsschemaspaginationresult) | Reservations with shipping |
| Model11 | [#/components/schemas/Model11](#componentsschemasmodel11) |  |
| phones | [#/components/schemas/phones](#componentsschemasphones) |  |
| FullAddress | [#/components/schemas/FullAddress](#componentsschemasfulladdress) |  |
| Model12 | [#/components/schemas/Model12](#componentsschemasmodel12) |  |
| contacts | [#/components/schemas/contacts](#componentsschemascontacts) |  |
| locations1 | [#/components/schemas/locations1](#componentsschemaslocations1) |  |
| DirectoryCompany1 | [#/components/schemas/DirectoryCompany1](#componentsschemasdirectorycompany1) |  |
| Model13 | [#/components/schemas/Model13](#componentsschemasmodel13) |  |
| Model14 | [#/components/schemas/Model14](#componentsschemasmodel14) |  |
| Reservation2 | [#/components/schemas/Reservation2](#componentsschemasreservation2) |  |
| reservations2 | [#/components/schemas/reservations2](#componentsschemasreservations2) |  |
| ReservationListResponse1 | [#/components/schemas/ReservationListResponse1](#componentsschemasreservationlistresponse1) |  |
| Model15 | [#/components/schemas/Model15](#componentsschemasmodel15) |  |
| Error4 | [#/components/schemas/Error4](#componentsschemaserror4) |  |
| Model16 | [#/components/schemas/Model16](#componentsschemasmodel16) |  |
| items | [#/components/schemas/items](#componentsschemasitems) |  |
| PackageInfoRequest | [#/components/schemas/PackageInfoRequest](#componentsschemaspackageinforequest) | Additional package information |
| contact | [#/components/schemas/contact](#componentsschemascontact) |  |
| FullAddressShipping | [#/components/schemas/FullAddressShipping](#componentsschemasfulladdressshipping) | Courier pick up address. If the request has `destination` and `location_id` a 400 error is returned. |
| FullAddressShipping1 | [#/components/schemas/FullAddressShipping1](#componentsschemasfulladdressshipping1) | Courier delivery address |
| Model17 | [#/components/schemas/Model17](#componentsschemasmodel17) |  |
| directory | [#/components/schemas/directory](#componentsschemasdirectory) |  |
| DirectoryBody | [#/components/schemas/DirectoryBody](#componentsschemasdirectorybody) |  |
| Error5 | [#/components/schemas/Error5](#componentsschemaserror5) |  |
| Error6 | [#/components/schemas/Error6](#componentsschemaserror6) | Unable to complete request |
| coords | [#/components/schemas/coords](#componentsschemascoords) | Coordinates for which to find the closest locations. |
| Model18 | [#/components/schemas/Model18](#componentsschemasmodel18) |  |
| LocationAvailable | [#/components/schemas/LocationAvailable](#componentsschemaslocationavailable) |  |
| locations2 | [#/components/schemas/locations2](#componentsschemaslocations2) |  |
| Model19 | [#/components/schemas/Model19](#componentsschemasmodel19) |  |
| Error7 | [#/components/schemas/Error7](#componentsschemaserror7) | Unable to find a box with required size or provided zip code is invalid. |
| Error8 | [#/components/schemas/Error8](#componentsschemaserror8) | Unexpected error |
| directoriesIds | [#/components/schemas/directoriesIds](#componentsschemasdirectoriesids) |  |
| Model20 | [#/components/schemas/Model20](#componentsschemasmodel20) |  |
| PackageInfoRequest1 | [#/components/schemas/PackageInfoRequest1](#componentsschemaspackageinforequest1) | Additional package information |
| contact_info1 | [#/components/schemas/contact_info1](#componentsschemascontact_info1) | Contact person for any issue regarding the reservation. It should contain either name or email. |
| recipient_info1 | [#/components/schemas/recipient_info1](#componentsschemasrecipient_info1) | Contact details of the final user |
| PreReservationRequest | [#/components/schemas/PreReservationRequest](#componentsschemasprereservationrequest) |  |
| Model21 | [#/components/schemas/Model21](#componentsschemasmodel21) |  |
| Error9 | [#/components/schemas/Error9](#componentsschemaserror9) | The dimensions provided cannot be held by any box |
| size | [#/components/schemas/size](#componentsschemassize) | Required box size |
| PackageInfoRequest2 | [#/components/schemas/PackageInfoRequest2](#componentsschemaspackageinforequest2) | Additional package information |
| Address | [#/components/schemas/Address](#componentsschemasaddress) | Courier pick up address |
| contact1 | [#/components/schemas/contact1](#componentsschemascontact1) | Contact person |
| Model22 | [#/components/schemas/Model22](#componentsschemasmodel22) |  |
| Model23 | [#/components/schemas/Model23](#componentsschemasmodel23) |  |
| ReservationRequest | [#/components/schemas/ReservationRequest](#componentsschemasreservationrequest) |  |
| Error10 | [#/components/schemas/Error10](#componentsschemaserror10) | Insufficient scope in requested location |
| Error11 | [#/components/schemas/Error11](#componentsschemaserror11) | Shipment id already exists within the company |
| Error12 | [#/components/schemas/Error12](#componentsschemaserror12) | Unable to assign a box with required size |
| Model24 | [#/components/schemas/Model24](#componentsschemasmodel24) |  |
| tokens1 | [#/components/schemas/tokens1](#componentsschemastokens1) | Reservation tokens and shipping order id with encoded images |
| Model25 | [#/components/schemas/Model25](#componentsschemasmodel25) |  |
| Error13 | [#/components/schemas/Error13](#componentsschemaserror13) | Reservation cannot longer be updated |
| Error14 | [#/components/schemas/Error14](#componentsschemaserror14) | Reservation has missing information |
| Error15 | [#/components/schemas/Error15](#componentsschemaserror15) | Failed to create order with carrier |
| quoting | [#/components/schemas/quoting](#componentsschemasquoting) | Quoting information |
| Model26 | [#/components/schemas/Model26](#componentsschemasmodel26) |  |
| Error16 | [#/components/schemas/Error16](#componentsschemaserror16) |  |
| Model27 | [#/components/schemas/Model27](#componentsschemasmodel27) |  |
| Model28 | [#/components/schemas/Model28](#componentsschemasmodel28) |  |
| PreReservationUpdate | [#/components/schemas/PreReservationUpdate](#componentsschemasprereservationupdate) | Reservation fields to update. At least one field must be sent. |
| Error17 | [#/components/schemas/Error17](#componentsschemaserror17) |  |
| locations3 | [#/components/schemas/locations3](#componentsschemaslocations3) |  |
| directory1 | [#/components/schemas/directory1](#componentsschemasdirectory1) |  |
| DirectoryUpdateBody | [#/components/schemas/DirectoryUpdateBody](#componentsschemasdirectoryupdatebody) |  |
| Model29 | [#/components/schemas/Model29](#componentsschemasmodel29) |  |
| Model30 | [#/components/schemas/Model30](#componentsschemasmodel30) |  |
| Model31 | [#/components/schemas/Model31](#componentsschemasmodel31) |  |
| contacts1 | [#/components/schemas/contacts1](#componentsschemascontacts1) |  |
| Model32 | [#/components/schemas/Model32](#componentsschemasmodel32) |  |
| ReservationResponse | [#/components/schemas/ReservationResponse](#componentsschemasreservationresponse) |  |
| Error18 | [#/components/schemas/Error18](#componentsschemaserror18) | The reservation is in an ongoing state |

## Path Details

***

### [GET]/company/action_logs

- Summary  
Activity log

- Description  
Get last 1000 logs of user activity

- Security  
oauth2  

#### Responses

- 200 Successful

`application/json`

```ts
{
  // #/components/schemas/action_logs
  action_logs: {
    resource?: string
    status?: number
    user_email?: string
    referer?: string
    origin?: string
    timestamp?: string
    payload?: string
  }[]
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [GET]/company/directories

- Summary  
Get directories

- Description  
Returns all directories associated to the user company

- Security  
oauth2  

#### Parameters(Query)

```ts
page?: integer //default: 1
```

```ts
page_size?: integer //default: 15
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  // #/components/schemas/directories
  directories: {
    // Directory identifier
    id: string
    company: {
      // Company name
      name?: string
      // Resource identifier
      id?: string
    }
    author: {
      // Author name
      name?: string
      // Resource identifier
      id?: string
    }
    isActive: boolean
    // Directory name
    name: string
    created_at: string
    updated_at?: string
  }[]
  status_code?: enum[200]
  // Total number of results
  total?: number
  // Page number returned
  page?: number
  // Maximum number of results per page
  page_size?: number
  // Total number of pages
  total_pages?: number
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [POST]/company/directories

- Summary  
Create directory

- Description  
Create and associates a directory to the company

- Security  
oauth2  

#### RequestBody

- application/json

```ts
{
  directory: {
    // Directory name
    name: string
  }
}
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  directory: {
    // #/components/schemas/contacts
    contacts: {
      // Contact name
      name: string
      // Contact name
      last_name: string
      email?: string
      gender: enum[male, female, unspecified]
      age: integer
      // #/components/schemas/phones
      // Contact number
      phones?: string[]
      address: {
        delivery_info?: string
        city?: string
        state?: string
        country?: string //default: Mexico
        street: string
        number: string
        // Address borough/suburb
        town: string
        zip_code: string
        // Address references
        references?: string
        // Cross street
        cross_streets?: string
      }
      // Directory identifier
      directory?: string
    }[]
    // #/components/schemas/locations1
    locations: {
      // Resource identifier
      id: string
      // Locker identifier
      uuid: string
      // Locker business name
      internal_name: string
      // Locker host address
      address: {
        // Whether it has a parking space
        parking?: boolean
        city?: string
        state?: string
        country?: string //default: Mexico
        street: string
        number: string
        // Address borough/suburb
        town: string
        zip_code: string
        // Address references
        references?: string
        // Cross street
        cross_streets?: string
      }
    }[]
    isActive: boolean //default: true
    // Directory identifier
    id: string
    company: {
      // Company name
      name?: string
      // Resource identifier
      id?: string
    }
    author: {
      // Author name
      name?: string
      // Resource identifier
      id?: string
    }
    // Directory name
    name: string
    created_at: string
    updated_at?: string
  }
  status_code?: enum[200]
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 409 Conflict

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[409]
  // Error type
  type?: string
  // Human readable message
  message: string
}
```

- 424 Unable to complete request

`application/json`

```ts
// Unable to complete request
{
  // HTTP status code
  status_code?: enum[424]
  // Error type
  type?: string
  // Human readable message
  message: string
}
```

***

### [GET]/company/locations

- Summary  
Get locations

- Description  
Returns all locations where the user can operate

- Security  
oauth2  

#### Parameters(Query)

```ts
internal_name?: string
```

```ts
page?: integer //default: 1
```

```ts
page_size?: integer //default: 15
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  // #/components/schemas/locations
  locations: {
    // #/components/schemas/boxes
    boxes: {
      size: enum[s, m, l, xl]
      // Total boxes of size. Only for private lockers
      total?: number
      // Total occupied boxes of size. Only for private lockers
      occupied?: number
      // Whether the locker has boxes available in this size
      available?: boolean
    }[]
    // Number of boxes in the public network being used
    public_boxes?: number
    // Resource identifier
    id: string
    // Locker identifier
    uuid: string
    // Locker business name
    internal_name: string
    // Locker host address
    address: {
      // Whether it has a parking space
      parking?: boolean
      city?: string
      state?: string
      country?: string //default: Mexico
      street: string
      number: string
      // Address borough/suburb
      town: string
      zip_code: string
      // Address references
      references?: string
      // Cross street
      cross_streets?: string
    }
    // Host location coordinates
    coords: {
      latitude: number
      longitude: number
    }
    // Contact number
    phone?: string
    // #/components/schemas/service_days
    service_days: {
      weekday: number
      string_weekday: enum[monday, tuesday, wednesday, thursday, friday, saturday, sunday]
      // Opening hour in hh:mm format
      opening_time: string //default: 09:00
      // Closing hour in hh:mm format
      closing_time: string //default: 18:00
    }[]
    timezone: string
  }[]
  status_code?: enum[200]
  // Total number of results
  total?: number
  // Page number returned
  page?: number
  // Maximum number of results per page
  page_size?: number
  // Total number of pages
  total_pages?: number
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [GET]/company/notification_emails

- Summary  
Get company notifications emails

- Security  
oauth2  

#### Responses

- 200 Successful

`application/json`

```ts
{
  management_emails: {
    operator?: string
    administrator?: string
  }
  status_code?: number
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [PUT]/company/notification_emails

- Summary  
Edit company notifications emails

- Security  
oauth2  

#### RequestBody

- application/json

```ts
{
  operator_email?: string
  administrator_email?: string
}
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  management_emails: {
    operator?: string
    administrator?: string
  }
  status_code?: number
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [GET]/company/pre_reservations

- Summary  
Get all company pre-reservations

- Description  
Returns all pre-reservations owned by the company

- Security  
oauth2  

#### Parameters(Query)

```ts
only_active?: boolean //default: true
```

```ts
status?: enum[active, finished]
```

```ts
shipment_id?: string
```

```ts
from?: string
```

```ts
sort?: string
```

```ts
page?: integer //default: 1
```

```ts
page_size?: integer //default: 15
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  // #/components/schemas/reservations
  reservations: {
    // Reservation identifier
    id?: string
    // Custom package identifier
    shipment_id?: string
    // Whether the reservation is confirmed on the locker
    is_confirmed?: boolean
    // Whether the reservation is in local mode
    is_local?: boolean
    location: {
      // Resource identifier
      id: string
      // Locker identifier
      uuid: string
      // Locker business name
      internal_name: string
      // Locker host address
      address: {
        // Whether it has a parking space
        parking?: boolean
        city?: string
        state?: string
        country?: string //default: Mexico
        street: string
        number: string
        // Address borough/suburb
        town: string
        zip_code: string
        // Address references
        references?: string
        // Cross street
        cross_streets?: string
      }
    }
    // Package information provided
    package_info: {
      // Package width in cm
      width?: number
      // Package height in cm
      height?: number
      // Package length in cm
      length?: number
      // Package estimated value in MXN
      value?: number
      // Package weight in kg
      weight?: number
      // Package description, useful for logistic services, use a comma to split multiple descriptions
      description?: string
    }
    // Delivery date limit in ISO 8601 format
    delivery_due_date?: string
    // Pick up date limit in ISO 8601 format
    pickup_due_date?: string
    // #/components/schemas/EventHistory
    event_history: {
      status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
      // Update date in ISO 8601 format
      date?: string
    }[]
    // Additional reservation data
    additional: {
      string?: string
    }
    shipping: {
    }
    // Reservation creation date in ISO 8601 format
    created_at?: string
    // Reservation last update date in ISO 8601 format
    updated_at?: string
    // Assigned box information
    locker: {
      // Assigned box size
      size?: string
      // Assigned box identifier
      uuid?: string
      // Whether the box belongs to public network
      is_public?: boolean
      // #/components/schemas/boxes1
      boxes: {
        // Box size
        size?: string
        // Box identifier
        uuid?: string
      }[]
    }
    // Contact person for any issue regarding the reservation.
    contact_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Contact details of the final user
    recipient_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Whether recipient's notifications are enabled.
    recipient_notifications_enabled?: boolean
  }[]
  status_code?: enum[200]
  // Total number of results
  total?: number
  // Page number returned
  page?: number
  // Maximum number of results per page
  page_size?: number
  // Total number of pages
  total_pages?: number
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [GET]/company/reservations

- Summary  
Get all company reservations

- Description  
Returns all reservations owned by the company

- Security  
oauth2  

#### Parameters(Query)

```ts
only_active?: boolean //default: true
```

```ts
status?: enum[active, finished]
```

```ts
shipment_id?: string
```

```ts
from?: string
```

```ts
sort?: string
```

```ts
page?: integer //default: 1
```

```ts
page_size?: integer //default: 15
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  // #/components/schemas/reservations
  reservations: {
    // Reservation identifier
    id?: string
    // Custom package identifier
    shipment_id?: string
    // Whether the reservation is confirmed on the locker
    is_confirmed?: boolean
    // Whether the reservation is in local mode
    is_local?: boolean
    location: {
      // Resource identifier
      id: string
      // Locker identifier
      uuid: string
      // Locker business name
      internal_name: string
      // Locker host address
      address: {
        // Whether it has a parking space
        parking?: boolean
        city?: string
        state?: string
        country?: string //default: Mexico
        street: string
        number: string
        // Address borough/suburb
        town: string
        zip_code: string
        // Address references
        references?: string
        // Cross street
        cross_streets?: string
      }
    }
    // Package information provided
    package_info: {
      // Package width in cm
      width?: number
      // Package height in cm
      height?: number
      // Package length in cm
      length?: number
      // Package estimated value in MXN
      value?: number
      // Package weight in kg
      weight?: number
      // Package description, useful for logistic services, use a comma to split multiple descriptions
      description?: string
    }
    // Delivery date limit in ISO 8601 format
    delivery_due_date?: string
    // Pick up date limit in ISO 8601 format
    pickup_due_date?: string
    // #/components/schemas/EventHistory
    event_history: {
      status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
      // Update date in ISO 8601 format
      date?: string
    }[]
    // Additional reservation data
    additional: {
      string?: string
    }
    shipping: {
    }
    // Reservation creation date in ISO 8601 format
    created_at?: string
    // Reservation last update date in ISO 8601 format
    updated_at?: string
    // Assigned box information
    locker: {
      // Assigned box size
      size?: string
      // Assigned box identifier
      uuid?: string
      // Whether the box belongs to public network
      is_public?: boolean
      // #/components/schemas/boxes1
      boxes: {
        // Box size
        size?: string
        // Box identifier
        uuid?: string
      }[]
    }
    // Contact person for any issue regarding the reservation.
    contact_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Contact details of the final user
    recipient_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Whether recipient's notifications are enabled.
    recipient_notifications_enabled?: boolean
  }[]
  status_code?: enum[200]
  // Total number of results
  total?: number
  // Page number returned
  page?: number
  // Maximum number of results per page
  page_size?: number
  // Total number of pages
  total_pages?: number
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [GET]/company/webhook

- Summary  
Get webhook settings

- Security  
oauth2  

#### Responses

- 200 Successful

`application/json`

```ts
{
  webhook: {
    enabled: boolean
    secret: string
    url: string
    // #/components/schemas/events
    events?: enum[reserved, delivered, reopened, reopened2, picked_up, cancelled, undelivered, unpicked][]
  }
  status_code?: enum[200]
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [PUT]/company/webhook

- Summary  
Update webhook settings

- Security  
oauth2  

#### RequestBody

- application/json

```ts
{
  url: string
  // #/components/schemas/events
  events?: enum[reserved, delivered, reopened, reopened2, picked_up, cancelled, undelivered, unpicked][]
}
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  webhook: {
    enabled: boolean
    secret: string
    url: string
    // #/components/schemas/events
    events?: enum[reserved, delivered, reopened, reopened2, picked_up, cancelled, undelivered, unpicked][]
  }
  status_code?: enum[200]
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [GET]/oauth/token

- Summary  
Validates token

#### Responses

- 200 Access token granted

`application/json`

```ts
// Access token granted
{
  status_code?: enum[200]
  // Access token to use in future requests
  access_token: string
  // Access token expiration in UTC format
  access_token_expires_at: string
  // User scope granted for this client
  scope: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [POST]/oauth/token

- Summary  
Get an access token

- Description  
Machine-to-machine authentication

#### RequestBody

- application/x-www-form-urlencoded

```ts
{
  grant_type: enum[client_credentials]
  client_id: string
  client_secret: string
}
```

#### Responses

- 200 Access token granted

`application/json`

```ts
// Access token granted
{
  status_code?: enum[200]
  // Access token to use in future requests
  access_token: string
  // Access token expiration in UTC format
  access_token_expires_at: string
  // User scope granted for this client
  scope: string
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [DELETE]/oauth/token

- Summary  
Revoke an access token

#### Responses

- 200 Successful

`application/json`

```ts
{
  status_code?: number //default: 200
  message?: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [GET]/oauth/user

- Summary  
Get authenticated user

- Description  
Returns information about the user to whom the access token belongs to

#### Responses

- 200 Successful

`application/json`

```ts
{
  status_code?: enum[200]
  // Authenticated user data
  user: {
    id: string
    name: string
    email: string
    company: {
      // Company identifier
      id?: string
      // Company name
      name?: string
    }
    // Location identifier where the user is allowed to operate
    location?: string
    // User scope granted
    scope: string
    // User creation date
    created_at: string
    // Date of last update
    updated_at: string
  }
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [GET]/reservations/{reservation_id}

- Summary  
Get a reservation

- Description  
Returns a reservation with its tokens.

- Security  
oauth2  

#### Parameters(Query)

```ts
include_tokens?: boolean //default: true
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  reservation: {
    // Reservation identifier
    id?: string
    // Custom package identifier
    shipment_id?: string
    // Whether the reservation is confirmed on the locker
    is_confirmed?: boolean
    // Whether the reservation is in local mode
    is_local?: boolean
    location: {
      // Resource identifier
      id: string
      // Locker identifier
      uuid: string
      // Locker business name
      internal_name: string
      // Locker host address
      address: {
        // Whether it has a parking space
        parking?: boolean
        city?: string
        state?: string
        country?: string //default: Mexico
        street: string
        number: string
        // Address borough/suburb
        town: string
        zip_code: string
        // Address references
        references?: string
        // Cross street
        cross_streets?: string
      }
    }
    // Package information provided
    package_info: {
      // Package width in cm
      width?: number
      // Package height in cm
      height?: number
      // Package length in cm
      length?: number
      // Package estimated value in MXN
      value?: number
      // Package weight in kg
      weight?: number
      // Package description, useful for logistic services, use a comma to split multiple descriptions
      description?: string
    }
    // Delivery date limit in ISO 8601 format
    delivery_due_date?: string
    // Pick up date limit in ISO 8601 format
    pickup_due_date?: string
    // #/components/schemas/EventHistory
    event_history: {
      status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
      // Update date in ISO 8601 format
      date?: string
    }[]
    // Additional reservation data
    additional: {
      string?: string
    }
    shipping: {
    }
    // Reservation creation date in ISO 8601 format
    created_at?: string
    // Reservation last update date in ISO 8601 format
    updated_at?: string
    // Assigned box information
    locker: {
      // Assigned box size
      size?: string
      // Assigned box identifier
      uuid?: string
      // Whether the box belongs to public network
      is_public?: boolean
      // #/components/schemas/boxes1
      boxes: {
        // Box size
        size?: string
        // Box identifier
        uuid?: string
      }[]
    }
    // Contact person for any issue regarding the reservation.
    contact_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Contact details of the final user
    recipient_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Whether recipient's notifications are enabled.
    recipient_notifications_enabled?: boolean
  }
  // Reservation tokens with QR
  tokens: {
    delivery_token?: string
    delivery_token_qr?: string
    reopen_1_token?: string
    reopen_1_token_qr?: string
    reopen_2_token?: string
    reopen_2_token_qr?: string
    pickup_token?: string
    pickup_token_qr?: string
    cancel_token?: string
    cancel_token_qr?: string
  }
  confirmation_code?: string
  status_code?: number
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 404 Resource not found

`application/json`

```ts
// Resource not found
{
  // HTTP status code
  status_code?: enum[404]
  // Error type
  type?: string //default: ENTITY_NOT_FOUND
  data: {
  }
  // Human readable message
  message: string
}
```

***

### [PUT]/reservations/{reservation_id}

- Summary  
Update a pre-reservation

- Description  
Allows to update reservation data and/or post-pone delivery date

- Security  
oauth2  

#### RequestBody

- application/json

```ts
// Reservation fields to update. At least one field must be sent.
{
  // Your identifier for the reservation.
  shipment_id?: string
  // Additional package information
  package_info: {
    // Package estimated value in MXN
    value?: number
    // Package weight in kg
    weight?: number
    // Package description, useful for logistic services, use a comma to split multiple descriptions
    description?: string
  }
  // Custom reservation information (up to 30 keys). Any key starting with `lok` will be ignored
  additional: {
    string?: string
  }
  // Contact person for any issue regarding the reservation. It should contain either name or email.
  contact_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Contact details of the final user
  recipient_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Whether to send the pick up token to recipient's email.
  recipient_notifications_enabled?: boolean
  // Size for which to find an available box
  size?: enum[s, m, l, xl]
  // Expected delivery date in ISO 8601 format. Defaults to 24 hours from request
  delivery_date?: string
  delivery_confirmation_enabled?: boolean
  // Custom delivery token.
  delivery_token?: string
}
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  reservation: {
    // Reservation identifier
    id?: string
    // Custom package identifier
    shipment_id?: string
    // Whether the reservation is confirmed on the locker
    is_confirmed?: boolean
    // Whether the reservation is in local mode
    is_local?: boolean
    // Location identifier
    location?: string
    // Package information provided
    package_info: {
      // Package width in cm
      width?: number
      // Package height in cm
      height?: number
      // Package length in cm
      length?: number
      // Package estimated value in MXN
      value?: number
      // Package weight in kg
      weight?: number
      // Package description, useful for logistic services, use a comma to split multiple descriptions
      description?: string
    }
    // Delivery date limit in ISO 8601 format
    delivery_due_date?: string
    // Pick up date limit in ISO 8601 format
    pickup_due_date?: string
    // #/components/schemas/EventHistory
    event_history: {
      status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
      // Update date in ISO 8601 format
      date?: string
    }[]
    // Additional reservation data
    additional: {
      string?: string
    }
    shipping: {
    }
    // Reservation creation date in ISO 8601 format
    created_at?: string
    // Reservation last update date in ISO 8601 format
    updated_at?: string
    // Assigned box information
    locker: {
      // Assigned box size
      size?: string
      // Assigned box identifier
      uuid?: string
      // Whether the box belongs to public network
      is_public?: boolean
      // #/components/schemas/boxes1
      boxes: {
        // Box size
        size?: string
        // Box identifier
        uuid?: string
      }[]
    }
    // Contact person for any issue regarding the reservation.
    contact_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Contact details of the final user
    recipient_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Whether recipient's notifications are enabled.
    recipient_notifications_enabled?: boolean
  }
  // Reservation tokens with QR
  tokens: {
    delivery_token?: string
    delivery_token_qr?: string
    reopen_1_token?: string
    reopen_1_token_qr?: string
    reopen_2_token?: string
    reopen_2_token_qr?: string
    pickup_token?: string
    pickup_token_qr?: string
    cancel_token?: string
    cancel_token_qr?: string
  }
  confirmation_code?: string
  status_code?: number
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 404 Resource not found

`application/json`

```ts
// Resource not found
{
  // HTTP status code
  status_code?: enum[404]
  // Error type
  type?: string //default: ENTITY_NOT_FOUND
  data: {
  }
  // Human readable message
  message: string
}
```

- 410 Gone

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[410]
  // Error type
  type?: enum[INVALID_RESERVATION]
  // Human readable message
  message: string
}
```

- 422 The dimensions provided cannot be held by any box

`application/json`

```ts
// The dimensions provided cannot be held by any box
{
  // HTTP status code
  status_code?: enum[422]
  // Error type
  type?: enum[BOX_INCOMPATIBLE]
  // Human readable message
  message: string
}
```

***

### [DELETE]/reservations/{reservation_id}

- Summary  
Cancel a reservation

- Description  
Cancel a reservation while it has not been delivered

- Security  
oauth2  

#### Responses

- 200 Successful

`application/json`

```ts
{
  reservation: {
    // Reservation identifier
    id?: string
    // Custom package identifier
    shipment_id?: string
    // Whether the reservation is confirmed on the locker
    is_confirmed?: boolean
    // Whether the reservation is in local mode
    is_local?: boolean
    // Location identifier
    location?: string
    // Package information provided
    package_info: {
      // Package width in cm
      width?: number
      // Package height in cm
      height?: number
      // Package length in cm
      length?: number
      // Package estimated value in MXN
      value?: number
      // Package weight in kg
      weight?: number
      // Package description, useful for logistic services, use a comma to split multiple descriptions
      description?: string
    }
    // Delivery date limit in ISO 8601 format
    delivery_due_date?: string
    // Pick up date limit in ISO 8601 format
    pickup_due_date?: string
    // #/components/schemas/EventHistory
    event_history: {
      status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
      // Update date in ISO 8601 format
      date?: string
    }[]
    // Additional reservation data
    additional: {
      string?: string
    }
    shipping: {
    }
    // Reservation creation date in ISO 8601 format
    created_at?: string
    // Reservation last update date in ISO 8601 format
    updated_at?: string
    // Assigned box information
    locker: {
      // Assigned box size
      size?: string
      // Assigned box identifier
      uuid?: string
      // Whether the box belongs to public network
      is_public?: boolean
      // #/components/schemas/boxes1
      boxes: {
        // Box size
        size?: string
        // Box identifier
        uuid?: string
      }[]
    }
    // Contact person for any issue regarding the reservation.
    contact_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Contact details of the final user
    recipient_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Whether recipient's notifications are enabled.
    recipient_notifications_enabled?: boolean
  }
  status_code?: number
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 404 Resource not found

`application/json`

```ts
// Resource not found
{
  // HTTP status code
  status_code?: enum[404]
  // Error type
  type?: string //default: ENTITY_NOT_FOUND
  data: {
  }
  // Human readable message
  message: string
}
```

- 409 The reservation is in an ongoing state

`application/json`

```ts
// The reservation is in an ongoing state
{
  // HTTP status code
  status_code?: enum[409]
  // Error type
  type?: enum[INVALID_RESERVATION]
  // Human readable message
  message: string
}
```

***

### [GET]/shipping/reservations

- Summary  
Get reservations with shipment

- Security  
oauth2  

#### Parameters(Query)

```ts
only_active?: boolean //default: true
```

```ts
status?: enum[active, finished]
```

```ts
shipment_id?: string
```

```ts
from?: string
```

```ts
sort?: string
```

```ts
page?: integer //default: 1
```

```ts
page_size?: integer //default: 15
```

#### Responses

- 200 Reservations with shipping

`application/json`

```ts
// Reservations with shipping
{
  status_code?: enum[200]
  // #/components/schemas/reservations1
  // Reservation with shipping
  reservations: {
    // Reservation identifier
    id?: string
    // Custom package identifier
    shipment_id?: string
    // Whether the reservation is confirmed on the locker
    is_confirmed?: boolean
    // Whether the reservation is in local mode
    is_local?: boolean
    // Location identifier
    location?: string
    // Package information provided
    package_info: {
      // Package width in cm
      width?: number
      // Package height in cm
      height?: number
      // Package length in cm
      length?: number
      // Package estimated value in MXN
      value?: number
      // Package weight in kg
      weight?: number
      // Package description, useful for logistic services, use a comma to split multiple descriptions
      description?: string
    }
    // Delivery date limit in ISO 8601 format
    delivery_due_date?: string
    // Pick up date limit in ISO 8601 format
    pickup_due_date?: string
    // #/components/schemas/EventHistory
    event_history: {
      status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
      // Update date in ISO 8601 format
      date?: string
    }[]
    // Additional reservation data
    additional: {
      string?: string
    }
    // Shipping information
    shipping: {
      // Carrier tracking id
      tracking_id?: string
      pickup_point: {
        // Courier pick up address
        address: string
        // Courier pick up notes
        notes?: string
      }
      destination_point: {
        // Courier destination address
        address: string
        // Courier destination notes
        notes?: string
      }
      // Carrier used
      carrier: string
      // Vehicle type quoted
      vehicle_type?: string
      // Shipping cost in MXN
      fee: string
      // Carrier recollection date
      required_pickup_date?: string //default: 2020-09-08T00:26:35.817Z
      // Carrier estimated delivery date
      estimated_delivery_date?: string //default: 2020-09-08T00:28:35.817Z
    }
    // Reservation creation date in ISO 8601 format
    created_at?: string
    // Reservation last update date in ISO 8601 format
    updated_at?: string
    // Assigned box information
    locker: {
      // Assigned box size
      size?: string
      // Assigned box identifier
      uuid?: string
      // Whether the box belongs to public network
      is_public?: boolean
      // #/components/schemas/boxes1
      boxes: {
        // Box size
        size?: string
        // Box identifier
        uuid?: string
      }[]
    }
    // Contact person for any issue regarding the reservation.
    contact_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Contact details of the final user
    recipient_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Whether recipient's notifications are enabled.
    recipient_notifications_enabled?: boolean
  }[]
  // Total number of results
  total?: number
  // Page number returned
  page?: number
  // Maximum number of results per page
  page_size?: number
  // Total number of pages
  total_pages?: number
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [GET]/company/action_logs/csv

- Summary  
Request report of user activity

- Description  
Enqueues the delivery of a complete log report with provided dates

- Security  
oauth2  

#### Parameters(Query)

```ts
from?: string //default: 1653342119905
```

```ts
until?: string //default: now
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  status_code?: number //default: 200
  message?: string
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [GET]/company/directories/{directory_id}

- Summary  
Get a directory

- Description  
Return requested directory

- Security  
oauth2  

#### Responses

- 200 Successful

`application/json`

```ts
{
  directory: {
    // #/components/schemas/contacts
    contacts: {
      // Contact name
      name: string
      // Contact name
      last_name: string
      email?: string
      gender: enum[male, female, unspecified]
      age: integer
      // #/components/schemas/phones
      // Contact number
      phones?: string[]
      address: {
        delivery_info?: string
        city?: string
        state?: string
        country?: string //default: Mexico
        street: string
        number: string
        // Address borough/suburb
        town: string
        zip_code: string
        // Address references
        references?: string
        // Cross street
        cross_streets?: string
      }
      // Directory identifier
      directory?: string
    }[]
    // #/components/schemas/locations1
    locations: {
      // Resource identifier
      id: string
      // Locker identifier
      uuid: string
      // Locker business name
      internal_name: string
      // Locker host address
      address: {
        // Whether it has a parking space
        parking?: boolean
        city?: string
        state?: string
        country?: string //default: Mexico
        street: string
        number: string
        // Address borough/suburb
        town: string
        zip_code: string
        // Address references
        references?: string
        // Cross street
        cross_streets?: string
      }
    }[]
    isActive: boolean //default: true
    // Directory identifier
    id: string
    company: {
      // Company name
      name?: string
      // Resource identifier
      id?: string
    }
    author: {
      // Author name
      name?: string
      // Resource identifier
      id?: string
    }
    // Directory name
    name: string
    created_at: string
    updated_at?: string
  }
  status_code?: enum[200]
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 404 Resource not found

`application/json`

```ts
// Resource not found
{
  // HTTP status code
  status_code?: enum[404]
  // Error type
  type?: string //default: ENTITY_NOT_FOUND
  data: {
  }
  // Human readable message
  message: string
}
```

***

### [PUT]/company/directories/{directory_id}

- Summary  
Update directory

- Description  
api<br/><br/>directory

- Security  
oauth2  

#### RequestBody

- application/json

```ts
{
  directory: {
    is_active?: boolean //default: true
    // #/components/schemas/locations3
    locations?: string[]
    // Directory name
    name: string
  }
}
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  directory: {
    // #/components/schemas/contacts
    contacts: {
      // Contact name
      name: string
      // Contact name
      last_name: string
      email?: string
      gender: enum[male, female, unspecified]
      age: integer
      // #/components/schemas/phones
      // Contact number
      phones?: string[]
      address: {
        delivery_info?: string
        city?: string
        state?: string
        country?: string //default: Mexico
        street: string
        number: string
        // Address borough/suburb
        town: string
        zip_code: string
        // Address references
        references?: string
        // Cross street
        cross_streets?: string
      }
      // Directory identifier
      directory?: string
    }[]
    // #/components/schemas/locations1
    locations: {
      // Resource identifier
      id: string
      // Locker identifier
      uuid: string
      // Locker business name
      internal_name: string
      // Locker host address
      address: {
        // Whether it has a parking space
        parking?: boolean
        city?: string
        state?: string
        country?: string //default: Mexico
        street: string
        number: string
        // Address borough/suburb
        town: string
        zip_code: string
        // Address references
        references?: string
        // Cross street
        cross_streets?: string
      }
    }[]
    isActive: boolean //default: true
    // Directory identifier
    id: string
    company: {
      // Company name
      name?: string
      // Resource identifier
      id?: string
    }
    author: {
      // Author name
      name?: string
      // Resource identifier
      id?: string
    }
    // Directory name
    name: string
    created_at: string
    updated_at?: string
  }
  status_code?: enum[200]
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 409 Conflict

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[409]
  // Error type
  type?: string
  // Human readable message
  message: string
}
```

- 424 Unable to complete request

`application/json`

```ts
// Unable to complete request
{
  // HTTP status code
  status_code?: enum[424]
  // Error type
  type?: string
  // Human readable message
  message: string
}
```

***

### [GET]/company/locations/{location_id}

- Summary  
Get a location

- Security  
oauth2  

#### Responses

- 200 Successful

`application/json`

```ts
{
  status_code?: enum[200]
  location: {
    // #/components/schemas/boxes
    boxes: {
      size: enum[s, m, l, xl]
      // Total boxes of size. Only for private lockers
      total?: number
      // Total occupied boxes of size. Only for private lockers
      occupied?: number
      // Whether the locker has boxes available in this size
      available?: boolean
    }[]
    // Number of boxes in the public network being used
    public_boxes?: number
    // Resource identifier
    id: string
    // Locker identifier
    uuid: string
    // Locker business name
    internal_name: string
    // Locker host address
    address: {
      // Whether it has a parking space
      parking?: boolean
      city?: string
      state?: string
      country?: string //default: Mexico
      street: string
      number: string
      // Address borough/suburb
      town: string
      zip_code: string
      // Address references
      references?: string
      // Cross street
      cross_streets?: string
    }
    // Host location coordinates
    coords: {
      latitude: number
      longitude: number
    }
    // Contact number
    phone?: string
    // #/components/schemas/service_days
    service_days: {
      weekday: number
      string_weekday: enum[monday, tuesday, wednesday, thursday, friday, saturday, sunday]
      // Opening hour in hh:mm format
      opening_time: string //default: 09:00
      // Closing hour in hh:mm format
      closing_time: string //default: 18:00
    }[]
    timezone: string
  }
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [GET]/company/reports/recent_activity

- Summary  
Get company recent activity

- Description  
Generates a CSV with reservations from two days ago to current date in the following format:<br/><br/>  
```csv  
"Id de pedido", "Locker", "Huso horario", "Torre", "Casillero", "Reservación", "Depósito", "Recolección", "Cancelación"  
"DFUL", "Antara", "America/Mexico_City", "00010003", "00010003002", "2020-10-10T15:52:30.000Z", "2020-10-10T15:52:30.000Z", "2020-10-10T15:52:30.000Z", "2020-10-10T15:52:30.000Z"  
```  


- Security  
oauth2  

#### Responses

- 200 Successful

`text/csv`

```ts
{
  "type": "string"
}
```

- 401 Authorization is invalid

`text/csv`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`text/csv`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [GET]/locations/{location_id}/reservations

- Summary  
Get location reservations

- Description  
Returns all reservations in the location

- Security  
oauth2  

#### Parameters(Query)

```ts
only_active?: boolean //default: true
```

```ts
status?: enum[active, finished]
```

```ts
shipment_id?: string
```

```ts
from?: string
```

```ts
sort?: string
```

```ts
page?: integer //default: 1
```

```ts
page_size?: integer //default: 15
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  // #/components/schemas/reservations2
  reservations: {
    // Reservation identifier
    id?: string
    // Custom package identifier
    shipment_id?: string
    // Whether the reservation is confirmed on the locker
    is_confirmed?: boolean
    // Whether the reservation is in local mode
    is_local?: boolean
    // Location identifier
    location?: string
    // Package information provided
    package_info: {
      // Package width in cm
      width?: number
      // Package height in cm
      height?: number
      // Package length in cm
      length?: number
      // Package estimated value in MXN
      value?: number
      // Package weight in kg
      weight?: number
      // Package description, useful for logistic services, use a comma to split multiple descriptions
      description?: string
    }
    // Delivery date limit in ISO 8601 format
    delivery_due_date?: string
    // Pick up date limit in ISO 8601 format
    pickup_due_date?: string
    // #/components/schemas/EventHistory
    event_history: {
      status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
      // Update date in ISO 8601 format
      date?: string
    }[]
    // Additional reservation data
    additional: {
      string?: string
    }
    shipping: {
    }
    // Reservation creation date in ISO 8601 format
    created_at?: string
    // Reservation last update date in ISO 8601 format
    updated_at?: string
    // Assigned box information
    locker: {
      // Assigned box size
      size?: string
      // Assigned box identifier
      uuid?: string
      // Whether the box belongs to public network
      is_public?: boolean
      // #/components/schemas/boxes1
      boxes: {
        // Box size
        size?: string
        // Box identifier
        uuid?: string
      }[]
    }
    // Contact person for any issue regarding the reservation.
    contact_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Contact details of the final user
    recipient_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Whether recipient's notifications are enabled.
    recipient_notifications_enabled?: boolean
  }[]
  status_code?: enum[200]
  // Total number of results
  total?: number
  // Page number returned
  page?: number
  // Maximum number of results per page
  page_size?: number
  // Total number of pages
  total_pages?: number
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 404 Resource not found

`application/json`

```ts
// Resource not found
{
  // HTTP status code
  status_code?: enum[404]
  // Error type
  type?: string //default: ENTITY_NOT_FOUND
  data: {
  }
  // Human readable message
  message: string
}
```

***

### [POST]/locations/{location_id}/reservations

- Summary  
Create a reservation

- Security  
oauth2  

#### RequestBody

- application/json

```ts
{
  // Size for which to find an available box
  size: enum[s, m, l, xl]
  // Custom delivery token.
  delivery_token?: string
  delivery_confirmation_enabled?: boolean
  // Your identifier for the reservation.
  shipment_id?: string
  // Additional package information
  package_info: {
    // Package estimated value in MXN
    value?: number
    // Package weight in kg
    weight?: number
    // Package description, useful for logistic services, use a comma to split multiple descriptions
    description?: string
  }
  // Custom reservation information (up to 30 keys). Any key starting with `lok` will be ignored
  additional: {
    string?: string
  }
  // Contact person for any issue regarding the reservation. It should contain either name or email.
  contact_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Contact details of the final user
  recipient_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Whether to send the pick up token to recipient's email.
  recipient_notifications_enabled?: boolean
}
```

#### Responses

- 201 Created

`application/json`

```ts
{
  reservation: {
    // Reservation identifier
    id?: string
    // Custom package identifier
    shipment_id?: string
    // Whether the reservation is confirmed on the locker
    is_confirmed?: boolean
    // Whether the reservation is in local mode
    is_local?: boolean
    // Location identifier
    location?: string
    // Package information provided
    package_info: {
      // Package width in cm
      width?: number
      // Package height in cm
      height?: number
      // Package length in cm
      length?: number
      // Package estimated value in MXN
      value?: number
      // Package weight in kg
      weight?: number
      // Package description, useful for logistic services, use a comma to split multiple descriptions
      description?: string
    }
    // Delivery date limit in ISO 8601 format
    delivery_due_date?: string
    // Pick up date limit in ISO 8601 format
    pickup_due_date?: string
    // #/components/schemas/EventHistory
    event_history: {
      status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
      // Update date in ISO 8601 format
      date?: string
    }[]
    // Additional reservation data
    additional: {
      string?: string
    }
    shipping: {
    }
    // Reservation creation date in ISO 8601 format
    created_at?: string
    // Reservation last update date in ISO 8601 format
    updated_at?: string
    // Assigned box information
    locker: {
      // Assigned box size
      size?: string
      // Assigned box identifier
      uuid?: string
      // Whether the box belongs to public network
      is_public?: boolean
      // #/components/schemas/boxes1
      boxes: {
        // Box size
        size?: string
        // Box identifier
        uuid?: string
      }[]
    }
    // Contact person for any issue regarding the reservation.
    contact_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Contact details of the final user
    recipient_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Whether recipient's notifications are enabled.
    recipient_notifications_enabled?: boolean
  }
  // Reservation tokens with QR
  tokens: {
    delivery_token?: string
    delivery_token_qr?: string
    reopen_1_token?: string
    reopen_1_token_qr?: string
    reopen_2_token?: string
    reopen_2_token_qr?: string
    pickup_token?: string
    pickup_token_qr?: string
    cancel_token?: string
    cancel_token_qr?: string
  }
  confirmation_code?: string
  status_code?: number
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Insufficient scope in requested location

`application/json`

```ts
// Insufficient scope in requested location
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 404 Resource not found

`application/json`

```ts
// Resource not found
{
  // HTTP status code
  status_code?: enum[404]
  // Error type
  type?: string //default: ENTITY_NOT_FOUND
  data: {
  }
  // Human readable message
  message: string
}
```

- 409 Shipment id already exists within the company

`application/json`

```ts
// Shipment id already exists within the company
{
  // HTTP status code
  status_code?: enum[409]
  // Error type
  type?: enum[UNIQUE_SHIPMENT]
  // Human readable message
  message: string
}
```

- 422 Unable to assign a box with required size

`application/json`

```ts
// Unable to assign a box with required size
{
  // HTTP status code
  status_code?: enum[422]
  // Error type
  type?: enum[BOX_INCOMPATIBLE, NO_CAPACITY]
  // Human readable message
  message: string
}
```

- 424 Unable to complete request

`application/json`

```ts
// Unable to complete request
{
  // HTTP status code
  status_code?: enum[424]
  // Error type
  type?: string
  // Human readable message
  message: string
}
```

***

### [GET]/reservations/{reservation_id}/label

- Summary  
Get reservation shipping label

- Security  
oauth2  

#### Responses

- 200 Successful

`application/json`

```ts
{
  // Tracking guide in JPEG format
  jpeg: string
  // Tracking guide in PDF format
  pdf: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 422 Unprocessable Entity

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[422]
  // Error type
  type?: string
  // Human readable message
  message: string
}
```

***

### [POST]/shipping_order

- Summary  
Create a shipping order getting a label as response

- Security  
oauth2  

#### RequestBody

- application/json

```ts
{
  // Preffered carrier
  preferred: string
  // Additional package information
  package_info: {
    // Package width in cm
    width?: number
    // Package height in cm
    height?: number
    // Package length in cm
    length?: number
    // Package estimated value in MXN
    value?: number
    // Package weight in kg
    weight: number
    // Package description, useful for logistic services, use a comma to split multiple descriptions
    description?: string
    // #/components/schemas/items
    items: {
      // Code for SAT messurments and packing
      units_weight: string
      // Code for SAT products and services
      product_service: string
      // Code for SAT dangerous materials
      dangerous_material?: string
      // Code for SAT packing type
      pack_type?: string
      // Quantity of the same object
      units?: string
      // Value to know if the item is dangerous
      is_dangerous: boolean
    }[]
  }
  // Location identifier for delivery. If the request has `destination` and `location_id` a 400 error is returned.
  location_id?: string
  // Courier pick up address. If the request has `destination` and `location_id` a 400 error is returned.
  destination: {
    contact: {
      phone: string
      // Contact name
      name?: string
      email: string
      // Company name
      company?: string
      // Contact taxpayer number
      rfc?: string
    }
    street_type?: string
    town_type?: string
    city?: string
    state?: string
    country?: string //default: Mexico
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
  // Courier delivery address
  origin: {
    contact:#/components/schemas/contact
    street_type?: string
    town_type?: string
    city?: string
    state?: string
    country?: string //default: Mexico
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
}
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  // Tracking guide in JPEG format
  jpeg: string
  // Tracking guide in PDF format
  pdf: string
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 422 Unprocessable Entity

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[422]
  // Error type
  type?: string
  // Human readable message
  message: string
}
```

***

### [POST]/locations/available

- Summary  
Locations for pre-reservations

- Description  
Returns locations available based on a geographical search. Either a zip code or custom center coordinates are required.<br/><br/>The response includes a token that can be used later to create a pre-reservation.

- Security  
oauth2  

#### RequestBody

- application/json

```ts
{
  // Coordinates for which to find the closest locations.
  coords: {
    // Center longitude
    long: number
    // Center latitude
    lat: number
  }
  // Zip code for which to find the closest locations.
  zip_code?: string
  // The maximum distance in meters from the center point that the locations can be.
  max_distance?: number //default: 50000
  // Maximum number of locations to return.
  limit: number
  // Size for which to find an available box
  size?: enum[s, m, l, xl]
}
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  status_code?: enum[200]
  // #/components/schemas/locations2
  locations: {
    // Distance in meters from the center
    distance: number
    // Locker host name
    host?: string
    // Whether the locker has 24hrs access
    is_24?: boolean
    // Host location coordinates
    coords: {
      latitude: number
      longitude: number
    }
    // Contact number
    phone?: string
    // #/components/schemas/service_days
    service_days: {
      weekday: number
      string_weekday: enum[monday, tuesday, wednesday, thursday, friday, saturday, sunday]
      // Opening hour in hh:mm format
      opening_time: string //default: 09:00
      // Closing hour in hh:mm format
      closing_time: string //default: 18:00
    }[]
    timezone: string
    // Locker business name
    internal_name: string
    // Locker host address
    address: {
      // Whether it has a parking space
      parking?: boolean
      city?: string
      state?: string
      country?: string //default: Mexico
      street: string
      number: string
      // Address borough/suburb
      town: string
      zip_code: string
      // Address references
      references?: string
      // Cross street
      cross_streets?: string
    }
    // Resource identifier
    id: string
    // Locker identifier
    uuid: string
  }[]
  // Token granted to create a pre-reservation in the locations returned
  grant?: string
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 422 Unable to find a box with required size or provided zip code is invalid.

`application/json`

```ts
// Unable to find a box with required size or provided zip code is invalid.
{
  // HTTP status code
  status_code?: enum[422]
  // Error type
  type?: enum[BOX_INCOMPATIBLE, INVALID_ADDRESS]
  // Human readable message
  message: string
}
```

- 500 Unexpected error

`application/json`

```ts
// Unexpected error
{
  // HTTP status code
  status_code?: enum[500]
  // Error type
  type?: string //default: UNKNOWN_ERROR
  // Human readable message
  message: string
}
```

***

### [POST]/company/directories/csv

- Summary  
Create directory

- Description  
Import a csv file to set contacts in a new directory or some already created

- Security  
oauth2  

#### RequestBody

- application/json

```ts
{
  // #/components/schemas/directoriesIds
  directoriesIds?: string[]
  csv?: string
}
```

#### Responses

- 204 No Content

`application/json`

```ts
{
  status_code?: number //default: 200
  message?: string
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 409 Conflict

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[409]
  // Error type
  type?: string
  // Human readable message
  message: string
}
```

- 424 Unable to complete request

`application/json`

```ts
// Unable to complete request
{
  // HTTP status code
  status_code?: enum[424]
  // Error type
  type?: string
  // Human readable message
  message: string
}
```

***

### [POST]/locations/{location_id}/pre_reservation

- Summary  
Create a pre-reservation

- Description  
Creates a pre-reservation in a [granted location](#operation/postLocationsAvailable).

- Security  
oauth2  

#### Headers

```ts
lok-grant-token: string
```

#### RequestBody

- application/json

```ts
{
  // Size for which to find an available box
  size?: enum[s, m, l, xl]
  // Expected delivery date in ISO 8601 format. Defaults to 24 hours from request
  delivery_date?: string
  delivery_confirmation_enabled?: boolean
  // Custom delivery token.
  delivery_token?: string
  // Your identifier for the reservation.
  shipment_id?: string
  // Additional package information
  package_info: {
    // Package estimated value in MXN
    value?: number
    // Package weight in kg
    weight?: number
    // Package description, useful for logistic services, use a comma to split multiple descriptions
    description?: string
  }
  // Custom reservation information (up to 30 keys). Any key starting with `lok` will be ignored
  additional: {
    string?: string
  }
  // Contact person for any issue regarding the reservation. It should contain either name or email.
  contact_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Contact details of the final user
  recipient_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Whether to send the pick up token to recipient's email.
  recipient_notifications_enabled?: boolean
}
```

#### Responses

- 201 Created

`application/json`

```ts
{
  reservation: {
    // Reservation identifier
    id?: string
    // Custom package identifier
    shipment_id?: string
    // Whether the reservation is confirmed on the locker
    is_confirmed?: boolean
    // Whether the reservation is in local mode
    is_local?: boolean
    // Location identifier
    location?: string
    // Package information provided
    package_info: {
      // Package width in cm
      width?: number
      // Package height in cm
      height?: number
      // Package length in cm
      length?: number
      // Package estimated value in MXN
      value?: number
      // Package weight in kg
      weight?: number
      // Package description, useful for logistic services, use a comma to split multiple descriptions
      description?: string
    }
    // Delivery date limit in ISO 8601 format
    delivery_due_date?: string
    // Pick up date limit in ISO 8601 format
    pickup_due_date?: string
    // #/components/schemas/EventHistory
    event_history: {
      status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
      // Update date in ISO 8601 format
      date?: string
    }[]
    // Additional reservation data
    additional: {
      string?: string
    }
    shipping: {
    }
    // Reservation creation date in ISO 8601 format
    created_at?: string
    // Reservation last update date in ISO 8601 format
    updated_at?: string
    // Assigned box information
    locker: {
      // Assigned box size
      size?: string
      // Assigned box identifier
      uuid?: string
      // Whether the box belongs to public network
      is_public?: boolean
      // #/components/schemas/boxes1
      boxes: {
        // Box size
        size?: string
        // Box identifier
        uuid?: string
      }[]
    }
    // Contact person for any issue regarding the reservation.
    contact_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Contact details of the final user
    recipient_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Whether recipient's notifications are enabled.
    recipient_notifications_enabled?: boolean
  }
  // Reservation tokens with QR
  tokens: {
    delivery_token?: string
    delivery_token_qr?: string
    reopen_1_token?: string
    reopen_1_token_qr?: string
    reopen_2_token?: string
    reopen_2_token_qr?: string
    pickup_token?: string
    pickup_token_qr?: string
    cancel_token?: string
    cancel_token_qr?: string
  }
  confirmation_code?: string
  status_code?: number
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 404 Resource not found

`application/json`

```ts
// Resource not found
{
  // HTTP status code
  status_code?: enum[404]
  // Error type
  type?: string //default: ENTITY_NOT_FOUND
  data: {
  }
  // Human readable message
  message: string
}
```

- 422 The dimensions provided cannot be held by any box

`application/json`

```ts
// The dimensions provided cannot be held by any box
{
  // HTTP status code
  status_code?: enum[422]
  // Error type
  type?: enum[BOX_INCOMPATIBLE]
  // Human readable message
  message: string
}
```

***

### [POST]/locations/{location_id}/shipping

- Summary  
Create a shipping order

- Security  
oauth2  

#### RequestBody

- application/json

```ts
{
  // Required box size
  size: {
    // Package width in cm
    width?: number
    // Package height in cm
    height?: number
    // Package length in cm
    length?: number
  }
  // Custom package identifier
  shipment_id: string
  // Additional package information
  package_info: {
    // Package estimated value in MXN
    value?: number
    // Package weight in kg
    weight: number
    // Package description, useful for logistic services, use a comma to split multiple descriptions
    description?: string
    // Request package insurance
    request_insurance?: boolean
    // Request thermal label size pdf
    request_thermal_lable?: boolean
  }
  // Required pick up date
  pickup_date: string
  // Courier pick up address
  origin: {
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
  // Contact person
  contact: {
    phone: string
    // Contact name
    name?: string
  }
}
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  status_code?: enum[200]
  // Custom package identifier
  shipment_id?: string
  // Tracking guide identifier
  tracking_id?: string
  // Carrier used
  carrier: string
  // Shipping cost in MXN
  fee: string
  // Carrier estimated delivery date
  estimated_delivery_date?: string //default: 2020-09-08T00:28:35.817Z
  // Tracking guide in PDF format
  label: string
  // Resource identifier
  reservation_id?: string
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 422 Unprocessable Entity

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[422]
  // Error type
  type?: string
  // Human readable message
  message: string
}
```

***

### [POST]/reservations/{reservation_id}/shipping

- Summary  
Request shipment

- Description  
Requests a shipping for a pre-reservation. All logistic-related data is saved under `additional` field.<br/><br/>This operation enqueues the pre-reservation confirmation, ensuring there is an available box on delivery.

- Security  
oauth2  

#### RequestBody

- application/json

```ts
{
  // Courier pick up address
  origin: {
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
  // Contact person
  contact: {
    phone: string
    // Contact name
    name?: string
  }
}
```

#### Responses

- 201 Created

`application/json`

```ts
{
  status_code?: enum[201]
  // Reservation with shipping
  reservation: {
    // Reservation identifier
    id?: string
    // Custom package identifier
    shipment_id?: string
    // Whether the reservation is confirmed on the locker
    is_confirmed?: boolean
    // Whether the reservation is in local mode
    is_local?: boolean
    // Location identifier
    location?: string
    // Package information provided
    package_info: {
      // Package width in cm
      width?: number
      // Package height in cm
      height?: number
      // Package length in cm
      length?: number
      // Package estimated value in MXN
      value?: number
      // Package weight in kg
      weight?: number
      // Package description, useful for logistic services, use a comma to split multiple descriptions
      description?: string
    }
    // Delivery date limit in ISO 8601 format
    delivery_due_date?: string
    // Pick up date limit in ISO 8601 format
    pickup_due_date?: string
    // #/components/schemas/EventHistory
    event_history: {
      status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
      // Update date in ISO 8601 format
      date?: string
    }[]
    // Additional reservation data
    additional: {
      string?: string
    }
    // Shipping information
    shipping: {
      // Carrier tracking id
      tracking_id?: string
      pickup_point: {
        // Courier pick up address
        address: string
        // Courier pick up notes
        notes?: string
      }
      destination_point: {
        // Courier destination address
        address: string
        // Courier destination notes
        notes?: string
      }
      // Carrier used
      carrier: string
      // Vehicle type quoted
      vehicle_type?: string
      // Shipping cost in MXN
      fee: string
      // Carrier recollection date
      required_pickup_date?: string //default: 2020-09-08T00:26:35.817Z
      // Carrier estimated delivery date
      estimated_delivery_date?: string //default: 2020-09-08T00:28:35.817Z
    }
    // Reservation creation date in ISO 8601 format
    created_at?: string
    // Reservation last update date in ISO 8601 format
    updated_at?: string
    // Assigned box information
    locker: {
      // Assigned box size
      size?: string
      // Assigned box identifier
      uuid?: string
      // Whether the box belongs to public network
      is_public?: boolean
      // #/components/schemas/boxes1
      boxes: {
        // Box size
        size?: string
        // Box identifier
        uuid?: string
      }[]
    }
    // Contact person for any issue regarding the reservation.
    contact_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Contact details of the final user
    recipient_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Whether recipient's notifications are enabled.
    recipient_notifications_enabled?: boolean
  }
  // Reservation tokens and shipping order id with encoded images
  tokens: {
    delivery_token?: string
    delivery_token_qr?: string
    reopen_1_token?: string
    reopen_1_token_qr?: string
    reopen_2_token?: string
    reopen_2_token_qr?: string
    pickup_token?: string
    pickup_token_qr?: string
    cancel_token?: string
    cancel_token_qr?: string
  }
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 410 Reservation cannot longer be updated

`application/json`

```ts
// Reservation cannot longer be updated
{
  // HTTP status code
  status_code?: enum[410]
  // Error type
  type?: enum[INVALID_RESERVATION]
  // Human readable message
  message: string
}
```

- 422 Reservation has missing information

`application/json`

```ts
// Reservation has missing information
{
  // HTTP status code
  status_code?: enum[422]
  // Error type
  type?: enum[INVALID_RESERVATION]
  // Human readable message
  message: string
}
```

- 424 Failed to create order with carrier

`application/json`

```ts
// Failed to create order with carrier
{
  // HTTP status code
  status_code?: enum[424]
  // Error type
  type?: enum[LOGISTIC_ERROR]
  // Human readable message
  message: string
}
```

***

### [POST]/reservations/{reservation_id}/shipping/quotation

- Summary  
Quote shipment

- Description  
Requests a shipping quote for a pre-reservation

- Security  
oauth2  

#### RequestBody

- application/json

```ts
{
  // Courier pick up address
  origin: {
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
  // Contact person
  contact: {
    phone: string
    // Contact name
    name?: string
  }
}
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  status_code?: enum[200]
  // Quoting information
  quoting: {
    // Courier pick up address
    origin: string
    // Locker address
    destination: string
    // Carrier used
    carrier: string
    // Vehicle type quoted
    vehicle_type?: string
    // Shipping cost in MXN
    fee: string
    // Carrier recollection date
    required_pickup_date?: string //default: 2020-09-08T00:26:35.817Z
    // Carrier estimated delivery date
    estimated_delivery_date?: string //default: 2020-09-08T00:28:35.817Z
  }
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 410 Gone

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[410]
  // Error type
  type?: string
  // Human readable message
  message: string
}
```

***

### [PUT]/company/webhook/status

- Summary  
Toggle webhook status

- Security  
oauth2  

#### RequestBody

- application/json

```ts
{
  enabled: boolean
}
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  status_code?: number //default: 200
  message?: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

***

### [PUT]/company/directories/{directory_id}/contacts

- Summary  
Overwrite contacts

- Security  
oauth2  

#### RequestBody

- application/json

```ts
{
  // #/components/schemas/contacts
  contacts: {
    // Contact name
    name: string
    // Contact name
    last_name: string
    email?: string
    gender: enum[male, female, unspecified]
    age: integer
    // #/components/schemas/phones
    // Contact number
    phones?: string[]
    address: {
      delivery_info?: string
      city?: string
      state?: string
      country?: string //default: Mexico
      street: string
      number: string
      // Address borough/suburb
      town: string
      zip_code: string
      // Address references
      references?: string
      // Cross street
      cross_streets?: string
    }
    // Directory identifier
    directory?: string
  }[]
}
```

#### Responses

- 200 Successful

`application/json`

```ts
{
  // #/components/schemas/contacts1
  contacts: {
    // Contact name
    name: string
    // Contact name
    last_name: string
    email?: string
    gender: enum[male, female, unspecified]
    age: integer
    // #/components/schemas/phones
    // Contact number
    phones?: string[]
    address: {
      delivery_info?: string
      city?: string
      state?: string
      country?: string //default: Mexico
      street: string
      number: string
      // Address borough/suburb
      town: string
      zip_code: string
      // Address references
      references?: string
      // Cross street
      cross_streets?: string
    }
    // Directory identifier
    directory?: string
    created_at: string
    updated_at?: string
  }[]
}
```

- 400 Bad Request

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

- 401 Authorization is invalid

`application/json`

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 403 Authorization scope is insufficient

`application/json`

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

- 409 Conflict

`application/json`

```ts
{
  // HTTP status code
  status_code?: enum[409]
  // Error type
  type?: string
  // Human readable message
  message: string
}
```

- 424 Unable to complete request

`application/json`

```ts
// Unable to complete request
{
  // HTTP status code
  status_code?: enum[424]
  // Error type
  type?: string
  // Human readable message
  message: string
}
```

## References

### #/components/requestBodies/Model24

- application/json

```ts
{
  // Courier pick up address
  origin: {
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
  // Contact person
  contact: {
    phone: string
    // Contact name
    name?: string
  }
}
```

### #/components/securitySchemes/oauth2

```ts
{
  "type": "oauth2",
  "flows": {
    "client_credentials": {
      "tokenUrl": "/oauth/token",
      "scopes": {
        "admin": "Lok Admin",
        "owner": "Company owner",
        "employee": "Company employee",
        "reporter": "Company reporter",
        "seller": "Company seller"
      }
    }
  }
}
```

### #/components/schemas/Model1

```ts
{
  resource?: string
  status?: number
  user_email?: string
  referer?: string
  origin?: string
  timestamp?: string
  payload?: string
}
```

### #/components/schemas/action_logs

```ts
{
  resource?: string
  status?: number
  user_email?: string
  referer?: string
  origin?: string
  timestamp?: string
  payload?: string
}[]
```

### #/components/schemas/Model2

```ts
{
  // #/components/schemas/action_logs
  action_logs: {
    resource?: string
    status?: number
    user_email?: string
    referer?: string
    origin?: string
    timestamp?: string
    payload?: string
  }[]
}
```

### #/components/schemas/keys

```ts
string[]
```

### #/components/schemas/data

```ts
{
  // #/components/schemas/keys
  keys?: string[]
  // Part where validations failed
  source: enum[payload, headers, query]
}
```

### #/components/schemas/Error

```ts
{
  // HTTP status code
  status_code?: enum[400]
  // Error type
  type?: string //default: INVALID_REQUEST
  data: {
    // #/components/schemas/keys
    keys?: string[]
    // Part where validations failed
    source: enum[payload, headers, query]
  }
  // Human readable message
  message: string
}
```

### #/components/schemas/Error1

```ts
// Authorization is invalid
{
  // HTTP status code
  status_code?: enum[401]
  // Error type
  type?: enum[INVALID_TOKEN, INVALID_GRANT, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

### #/components/schemas/Error2

```ts
// Authorization scope is insufficient
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

### #/components/schemas/company

```ts
{
  // Company name
  name?: string
  // Resource identifier
  id?: string
}
```

### #/components/schemas/author

```ts
{
  // Author name
  name?: string
  // Resource identifier
  id?: string
}
```

### #/components/schemas/DirectoryCompany

```ts
{
  // Directory identifier
  id: string
  company: {
    // Company name
    name?: string
    // Resource identifier
    id?: string
  }
  author: {
    // Author name
    name?: string
    // Resource identifier
    id?: string
  }
  isActive: boolean
  // Directory name
  name: string
  created_at: string
  updated_at?: string
}
```

### #/components/schemas/directories

```ts
{
  // Directory identifier
  id: string
  company: {
    // Company name
    name?: string
    // Resource identifier
    id?: string
  }
  author: {
    // Author name
    name?: string
    // Resource identifier
    id?: string
  }
  isActive: boolean
  // Directory name
  name: string
  created_at: string
  updated_at?: string
}[]
```

### #/components/schemas/DirectoryListResponse

```ts
{
  // #/components/schemas/directories
  directories: {
    // Directory identifier
    id: string
    company: {
      // Company name
      name?: string
      // Resource identifier
      id?: string
    }
    author: {
      // Author name
      name?: string
      // Resource identifier
      id?: string
    }
    isActive: boolean
    // Directory name
    name: string
    created_at: string
    updated_at?: string
  }[]
  status_code?: enum[200]
  // Total number of results
  total?: number
  // Page number returned
  page?: number
  // Maximum number of results per page
  page_size?: number
  // Total number of pages
  total_pages?: number
}
```

### #/components/schemas/Model3

```ts
{
  size: enum[s, m, l, xl]
  // Total boxes of size. Only for private lockers
  total?: number
  // Total occupied boxes of size. Only for private lockers
  occupied?: number
  // Whether the locker has boxes available in this size
  available?: boolean
}
```

### #/components/schemas/boxes

```ts
{
  size: enum[s, m, l, xl]
  // Total boxes of size. Only for private lockers
  total?: number
  // Total occupied boxes of size. Only for private lockers
  occupied?: number
  // Whether the locker has boxes available in this size
  available?: boolean
}[]
```

### #/components/schemas/LocationAddress

```ts
// Locker host address
{
  // Whether it has a parking space
  parking?: boolean
  city?: string
  state?: string
  country?: string //default: Mexico
  street: string
  number: string
  // Address borough/suburb
  town: string
  zip_code: string
  // Address references
  references?: string
  // Cross street
  cross_streets?: string
}
```

### #/components/schemas/Coordinates

```ts
// Host location coordinates
{
  latitude: number
  longitude: number
}
```

### #/components/schemas/ServiceDay

```ts
{
  weekday: number
  string_weekday: enum[monday, tuesday, wednesday, thursday, friday, saturday, sunday]
  // Opening hour in hh:mm format
  opening_time: string //default: 09:00
  // Closing hour in hh:mm format
  closing_time: string //default: 18:00
}
```

### #/components/schemas/service_days

```ts
{
  weekday: number
  string_weekday: enum[monday, tuesday, wednesday, thursday, friday, saturday, sunday]
  // Opening hour in hh:mm format
  opening_time: string //default: 09:00
  // Closing hour in hh:mm format
  closing_time: string //default: 18:00
}[]
```

### #/components/schemas/LocationCompany

```ts
{
  // #/components/schemas/boxes
  boxes: {
    size: enum[s, m, l, xl]
    // Total boxes of size. Only for private lockers
    total?: number
    // Total occupied boxes of size. Only for private lockers
    occupied?: number
    // Whether the locker has boxes available in this size
    available?: boolean
  }[]
  // Number of boxes in the public network being used
  public_boxes?: number
  // Resource identifier
  id: string
  // Locker identifier
  uuid: string
  // Locker business name
  internal_name: string
  // Locker host address
  address: {
    // Whether it has a parking space
    parking?: boolean
    city?: string
    state?: string
    country?: string //default: Mexico
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
  // Host location coordinates
  coords: {
    latitude: number
    longitude: number
  }
  // Contact number
  phone?: string
  // #/components/schemas/service_days
  service_days: {
    weekday: number
    string_weekday: enum[monday, tuesday, wednesday, thursday, friday, saturday, sunday]
    // Opening hour in hh:mm format
    opening_time: string //default: 09:00
    // Closing hour in hh:mm format
    closing_time: string //default: 18:00
  }[]
  timezone: string
}
```

### #/components/schemas/locations

```ts
{
  // #/components/schemas/boxes
  boxes: {
    size: enum[s, m, l, xl]
    // Total boxes of size. Only for private lockers
    total?: number
    // Total occupied boxes of size. Only for private lockers
    occupied?: number
    // Whether the locker has boxes available in this size
    available?: boolean
  }[]
  // Number of boxes in the public network being used
  public_boxes?: number
  // Resource identifier
  id: string
  // Locker identifier
  uuid: string
  // Locker business name
  internal_name: string
  // Locker host address
  address: {
    // Whether it has a parking space
    parking?: boolean
    city?: string
    state?: string
    country?: string //default: Mexico
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
  // Host location coordinates
  coords: {
    latitude: number
    longitude: number
  }
  // Contact number
  phone?: string
  // #/components/schemas/service_days
  service_days: {
    weekday: number
    string_weekday: enum[monday, tuesday, wednesday, thursday, friday, saturday, sunday]
    // Opening hour in hh:mm format
    opening_time: string //default: 09:00
    // Closing hour in hh:mm format
    closing_time: string //default: 18:00
  }[]
  timezone: string
}[]
```

### #/components/schemas/LocationListResponse

```ts
{
  // #/components/schemas/locations
  locations: {
    // #/components/schemas/boxes
    boxes: {
      size: enum[s, m, l, xl]
      // Total boxes of size. Only for private lockers
      total?: number
      // Total occupied boxes of size. Only for private lockers
      occupied?: number
      // Whether the locker has boxes available in this size
      available?: boolean
    }[]
    // Number of boxes in the public network being used
    public_boxes?: number
    // Resource identifier
    id: string
    // Locker identifier
    uuid: string
    // Locker business name
    internal_name: string
    // Locker host address
    address: {
      // Whether it has a parking space
      parking?: boolean
      city?: string
      state?: string
      country?: string //default: Mexico
      street: string
      number: string
      // Address borough/suburb
      town: string
      zip_code: string
      // Address references
      references?: string
      // Cross street
      cross_streets?: string
    }
    // Host location coordinates
    coords: {
      latitude: number
      longitude: number
    }
    // Contact number
    phone?: string
    // #/components/schemas/service_days
    service_days: {
      weekday: number
      string_weekday: enum[monday, tuesday, wednesday, thursday, friday, saturday, sunday]
      // Opening hour in hh:mm format
      opening_time: string //default: 09:00
      // Closing hour in hh:mm format
      closing_time: string //default: 18:00
    }[]
    timezone: string
  }[]
  status_code?: enum[200]
  // Total number of results
  total?: number
  // Page number returned
  page?: number
  // Maximum number of results per page
  page_size?: number
  // Total number of pages
  total_pages?: number
}
```

### #/components/schemas/management_emails

```ts
{
  operator?: string
  administrator?: string
}
```

### #/components/schemas/Model4

```ts
{
  management_emails: {
    operator?: string
    administrator?: string
  }
  status_code?: number
}
```

### #/components/schemas/location

```ts
{
  // Resource identifier
  id: string
  // Locker identifier
  uuid: string
  // Locker business name
  internal_name: string
  // Locker host address
  address: {
    // Whether it has a parking space
    parking?: boolean
    city?: string
    state?: string
    country?: string //default: Mexico
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
}
```

### #/components/schemas/PackageInfo

```ts
// Package information provided
{
  // Package width in cm
  width?: number
  // Package height in cm
  height?: number
  // Package length in cm
  length?: number
  // Package estimated value in MXN
  value?: number
  // Package weight in kg
  weight?: number
  // Package description, useful for logistic services, use a comma to split multiple descriptions
  description?: string
}
```

### #/components/schemas/Model5

```ts
{
  status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
  // Update date in ISO 8601 format
  date?: string
}
```

### #/components/schemas/EventHistory

```ts
{
  status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
  // Update date in ISO 8601 format
  date?: string
}[]
```

### #/components/schemas/shipping

```ts
{
}
```

### #/components/schemas/Model6

```ts
{
  // Box size
  size?: string
  // Box identifier
  uuid?: string
}
```

### #/components/schemas/boxes1

```ts
{
  // Box size
  size?: string
  // Box identifier
  uuid?: string
}[]
```

### #/components/schemas/BoxInfo

```ts
// Assigned box information
{
  // Assigned box size
  size?: string
  // Assigned box identifier
  uuid?: string
  // Whether the box belongs to public network
  is_public?: boolean
  // #/components/schemas/boxes1
  boxes: {
    // Box size
    size?: string
    // Box identifier
    uuid?: string
  }[]
}
```

### #/components/schemas/contact_info

```ts
// Contact person for any issue regarding the reservation.
{
  name?: string
  email?: string
  phone?: string
}
```

### #/components/schemas/recipient_info

```ts
// Contact details of the final user
{
  name?: string
  email?: string
  phone?: string
}
```

### #/components/schemas/Reservation

```ts
{
  // Reservation identifier
  id?: string
  // Custom package identifier
  shipment_id?: string
  // Whether the reservation is confirmed on the locker
  is_confirmed?: boolean
  // Whether the reservation is in local mode
  is_local?: boolean
  location: {
    // Resource identifier
    id: string
    // Locker identifier
    uuid: string
    // Locker business name
    internal_name: string
    // Locker host address
    address: {
      // Whether it has a parking space
      parking?: boolean
      city?: string
      state?: string
      country?: string //default: Mexico
      street: string
      number: string
      // Address borough/suburb
      town: string
      zip_code: string
      // Address references
      references?: string
      // Cross street
      cross_streets?: string
    }
  }
  // Package information provided
  package_info: {
    // Package width in cm
    width?: number
    // Package height in cm
    height?: number
    // Package length in cm
    length?: number
    // Package estimated value in MXN
    value?: number
    // Package weight in kg
    weight?: number
    // Package description, useful for logistic services, use a comma to split multiple descriptions
    description?: string
  }
  // Delivery date limit in ISO 8601 format
  delivery_due_date?: string
  // Pick up date limit in ISO 8601 format
  pickup_due_date?: string
  // #/components/schemas/EventHistory
  event_history: {
    status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
    // Update date in ISO 8601 format
    date?: string
  }[]
  // Additional reservation data
  additional: {
    string?: string
  }
  shipping: {
  }
  // Reservation creation date in ISO 8601 format
  created_at?: string
  // Reservation last update date in ISO 8601 format
  updated_at?: string
  // Assigned box information
  locker: {
    // Assigned box size
    size?: string
    // Assigned box identifier
    uuid?: string
    // Whether the box belongs to public network
    is_public?: boolean
    // #/components/schemas/boxes1
    boxes: {
      // Box size
      size?: string
      // Box identifier
      uuid?: string
    }[]
  }
  // Contact person for any issue regarding the reservation.
  contact_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Contact details of the final user
  recipient_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Whether recipient's notifications are enabled.
  recipient_notifications_enabled?: boolean
}
```

### #/components/schemas/reservations

```ts
{
  // Reservation identifier
  id?: string
  // Custom package identifier
  shipment_id?: string
  // Whether the reservation is confirmed on the locker
  is_confirmed?: boolean
  // Whether the reservation is in local mode
  is_local?: boolean
  location: {
    // Resource identifier
    id: string
    // Locker identifier
    uuid: string
    // Locker business name
    internal_name: string
    // Locker host address
    address: {
      // Whether it has a parking space
      parking?: boolean
      city?: string
      state?: string
      country?: string //default: Mexico
      street: string
      number: string
      // Address borough/suburb
      town: string
      zip_code: string
      // Address references
      references?: string
      // Cross street
      cross_streets?: string
    }
  }
  // Package information provided
  package_info: {
    // Package width in cm
    width?: number
    // Package height in cm
    height?: number
    // Package length in cm
    length?: number
    // Package estimated value in MXN
    value?: number
    // Package weight in kg
    weight?: number
    // Package description, useful for logistic services, use a comma to split multiple descriptions
    description?: string
  }
  // Delivery date limit in ISO 8601 format
  delivery_due_date?: string
  // Pick up date limit in ISO 8601 format
  pickup_due_date?: string
  // #/components/schemas/EventHistory
  event_history: {
    status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
    // Update date in ISO 8601 format
    date?: string
  }[]
  // Additional reservation data
  additional: {
    string?: string
  }
  shipping: {
  }
  // Reservation creation date in ISO 8601 format
  created_at?: string
  // Reservation last update date in ISO 8601 format
  updated_at?: string
  // Assigned box information
  locker: {
    // Assigned box size
    size?: string
    // Assigned box identifier
    uuid?: string
    // Whether the box belongs to public network
    is_public?: boolean
    // #/components/schemas/boxes1
    boxes: {
      // Box size
      size?: string
      // Box identifier
      uuid?: string
    }[]
  }
  // Contact person for any issue regarding the reservation.
  contact_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Contact details of the final user
  recipient_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Whether recipient's notifications are enabled.
  recipient_notifications_enabled?: boolean
}[]
```

### #/components/schemas/ReservationListResponse

```ts
{
  // #/components/schemas/reservations
  reservations: {
    // Reservation identifier
    id?: string
    // Custom package identifier
    shipment_id?: string
    // Whether the reservation is confirmed on the locker
    is_confirmed?: boolean
    // Whether the reservation is in local mode
    is_local?: boolean
    location: {
      // Resource identifier
      id: string
      // Locker identifier
      uuid: string
      // Locker business name
      internal_name: string
      // Locker host address
      address: {
        // Whether it has a parking space
        parking?: boolean
        city?: string
        state?: string
        country?: string //default: Mexico
        street: string
        number: string
        // Address borough/suburb
        town: string
        zip_code: string
        // Address references
        references?: string
        // Cross street
        cross_streets?: string
      }
    }
    // Package information provided
    package_info: {
      // Package width in cm
      width?: number
      // Package height in cm
      height?: number
      // Package length in cm
      length?: number
      // Package estimated value in MXN
      value?: number
      // Package weight in kg
      weight?: number
      // Package description, useful for logistic services, use a comma to split multiple descriptions
      description?: string
    }
    // Delivery date limit in ISO 8601 format
    delivery_due_date?: string
    // Pick up date limit in ISO 8601 format
    pickup_due_date?: string
    // #/components/schemas/EventHistory
    event_history: {
      status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
      // Update date in ISO 8601 format
      date?: string
    }[]
    // Additional reservation data
    additional: {
      string?: string
    }
    shipping: {
    }
    // Reservation creation date in ISO 8601 format
    created_at?: string
    // Reservation last update date in ISO 8601 format
    updated_at?: string
    // Assigned box information
    locker: {
      // Assigned box size
      size?: string
      // Assigned box identifier
      uuid?: string
      // Whether the box belongs to public network
      is_public?: boolean
      // #/components/schemas/boxes1
      boxes: {
        // Box size
        size?: string
        // Box identifier
        uuid?: string
      }[]
    }
    // Contact person for any issue regarding the reservation.
    contact_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Contact details of the final user
    recipient_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Whether recipient's notifications are enabled.
    recipient_notifications_enabled?: boolean
  }[]
  status_code?: enum[200]
  // Total number of results
  total?: number
  // Page number returned
  page?: number
  // Maximum number of results per page
  page_size?: number
  // Total number of pages
  total_pages?: number
}
```

### #/components/schemas/events

```ts
enum[reserved, delivered, reopened, reopened2, picked_up, cancelled, undelivered, unpicked][]
```

### #/components/schemas/webhook

```ts
{
  enabled: boolean
  secret: string
  url: string
  // #/components/schemas/events
  events?: enum[reserved, delivered, reopened, reopened2, picked_up, cancelled, undelivered, unpicked][]
}
```

### #/components/schemas/Model7

```ts
{
  webhook: {
    enabled: boolean
    secret: string
    url: string
    // #/components/schemas/events
    events?: enum[reserved, delivered, reopened, reopened2, picked_up, cancelled, undelivered, unpicked][]
  }
  status_code?: enum[200]
}
```

### #/components/schemas/Model8

```ts
// Access token granted
{
  status_code?: enum[200]
  // Access token to use in future requests
  access_token: string
  // Access token expiration in UTC format
  access_token_expires_at: string
  // User scope granted for this client
  scope: string
}
```

### #/components/schemas/company1

```ts
{
  // Company identifier
  id?: string
  // Company name
  name?: string
}
```

### #/components/schemas/user

```ts
// Authenticated user data
{
  id: string
  name: string
  email: string
  company: {
    // Company identifier
    id?: string
    // Company name
    name?: string
  }
  // Location identifier where the user is allowed to operate
  location?: string
  // User scope granted
  scope: string
  // User creation date
  created_at: string
  // Date of last update
  updated_at: string
}
```

### #/components/schemas/Model9

```ts
{
  status_code?: enum[200]
  // Authenticated user data
  user: {
    id: string
    name: string
    email: string
    company: {
      // Company identifier
      id?: string
      // Company name
      name?: string
    }
    // Location identifier where the user is allowed to operate
    location?: string
    // User scope granted
    scope: string
    // User creation date
    created_at: string
    // Date of last update
    updated_at: string
  }
}
```

### #/components/schemas/tokens

```ts
// Reservation tokens with QR
{
  delivery_token?: string
  delivery_token_qr?: string
  reopen_1_token?: string
  reopen_1_token_qr?: string
  reopen_2_token?: string
  reopen_2_token_qr?: string
  pickup_token?: string
  pickup_token_qr?: string
  cancel_token?: string
  cancel_token_qr?: string
}
```

### #/components/schemas/Model10

```ts
{
  reservation: {
    // Reservation identifier
    id?: string
    // Custom package identifier
    shipment_id?: string
    // Whether the reservation is confirmed on the locker
    is_confirmed?: boolean
    // Whether the reservation is in local mode
    is_local?: boolean
    location: {
      // Resource identifier
      id: string
      // Locker identifier
      uuid: string
      // Locker business name
      internal_name: string
      // Locker host address
      address: {
        // Whether it has a parking space
        parking?: boolean
        city?: string
        state?: string
        country?: string //default: Mexico
        street: string
        number: string
        // Address borough/suburb
        town: string
        zip_code: string
        // Address references
        references?: string
        // Cross street
        cross_streets?: string
      }
    }
    // Package information provided
    package_info: {
      // Package width in cm
      width?: number
      // Package height in cm
      height?: number
      // Package length in cm
      length?: number
      // Package estimated value in MXN
      value?: number
      // Package weight in kg
      weight?: number
      // Package description, useful for logistic services, use a comma to split multiple descriptions
      description?: string
    }
    // Delivery date limit in ISO 8601 format
    delivery_due_date?: string
    // Pick up date limit in ISO 8601 format
    pickup_due_date?: string
    // #/components/schemas/EventHistory
    event_history: {
      status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
      // Update date in ISO 8601 format
      date?: string
    }[]
    // Additional reservation data
    additional: {
      string?: string
    }
    shipping: {
    }
    // Reservation creation date in ISO 8601 format
    created_at?: string
    // Reservation last update date in ISO 8601 format
    updated_at?: string
    // Assigned box information
    locker: {
      // Assigned box size
      size?: string
      // Assigned box identifier
      uuid?: string
      // Whether the box belongs to public network
      is_public?: boolean
      // #/components/schemas/boxes1
      boxes: {
        // Box size
        size?: string
        // Box identifier
        uuid?: string
      }[]
    }
    // Contact person for any issue regarding the reservation.
    contact_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Contact details of the final user
    recipient_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Whether recipient's notifications are enabled.
    recipient_notifications_enabled?: boolean
  }
  // Reservation tokens with QR
  tokens: {
    delivery_token?: string
    delivery_token_qr?: string
    reopen_1_token?: string
    reopen_1_token_qr?: string
    reopen_2_token?: string
    reopen_2_token_qr?: string
    pickup_token?: string
    pickup_token_qr?: string
    cancel_token?: string
    cancel_token_qr?: string
  }
  confirmation_code?: string
  status_code?: number
}
```

### #/components/schemas/Error3

```ts
// Resource not found
{
  // HTTP status code
  status_code?: enum[404]
  // Error type
  type?: string //default: ENTITY_NOT_FOUND
  data: {
  }
  // Human readable message
  message: string
}
```

### #/components/schemas/pickup_point

```ts
{
  // Courier pick up address
  address: string
  // Courier pick up notes
  notes?: string
}
```

### #/components/schemas/destination_point

```ts
{
  // Courier destination address
  address: string
  // Courier destination notes
  notes?: string
}
```

### #/components/schemas/shipping1

```ts
// Shipping information
{
  // Carrier tracking id
  tracking_id?: string
  pickup_point: {
    // Courier pick up address
    address: string
    // Courier pick up notes
    notes?: string
  }
  destination_point: {
    // Courier destination address
    address: string
    // Courier destination notes
    notes?: string
  }
  // Carrier used
  carrier: string
  // Vehicle type quoted
  vehicle_type?: string
  // Shipping cost in MXN
  fee: string
  // Carrier recollection date
  required_pickup_date?: string //default: 2020-09-08T00:26:35.817Z
  // Carrier estimated delivery date
  estimated_delivery_date?: string //default: 2020-09-08T00:28:35.817Z
}
```

### #/components/schemas/Reservation1

```ts
// Reservation with shipping
{
  // Reservation identifier
  id?: string
  // Custom package identifier
  shipment_id?: string
  // Whether the reservation is confirmed on the locker
  is_confirmed?: boolean
  // Whether the reservation is in local mode
  is_local?: boolean
  // Location identifier
  location?: string
  // Package information provided
  package_info: {
    // Package width in cm
    width?: number
    // Package height in cm
    height?: number
    // Package length in cm
    length?: number
    // Package estimated value in MXN
    value?: number
    // Package weight in kg
    weight?: number
    // Package description, useful for logistic services, use a comma to split multiple descriptions
    description?: string
  }
  // Delivery date limit in ISO 8601 format
  delivery_due_date?: string
  // Pick up date limit in ISO 8601 format
  pickup_due_date?: string
  // #/components/schemas/EventHistory
  event_history: {
    status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
    // Update date in ISO 8601 format
    date?: string
  }[]
  // Additional reservation data
  additional: {
    string?: string
  }
  // Shipping information
  shipping: {
    // Carrier tracking id
    tracking_id?: string
    pickup_point: {
      // Courier pick up address
      address: string
      // Courier pick up notes
      notes?: string
    }
    destination_point: {
      // Courier destination address
      address: string
      // Courier destination notes
      notes?: string
    }
    // Carrier used
    carrier: string
    // Vehicle type quoted
    vehicle_type?: string
    // Shipping cost in MXN
    fee: string
    // Carrier recollection date
    required_pickup_date?: string //default: 2020-09-08T00:26:35.817Z
    // Carrier estimated delivery date
    estimated_delivery_date?: string //default: 2020-09-08T00:28:35.817Z
  }
  // Reservation creation date in ISO 8601 format
  created_at?: string
  // Reservation last update date in ISO 8601 format
  updated_at?: string
  // Assigned box information
  locker: {
    // Assigned box size
    size?: string
    // Assigned box identifier
    uuid?: string
    // Whether the box belongs to public network
    is_public?: boolean
    // #/components/schemas/boxes1
    boxes: {
      // Box size
      size?: string
      // Box identifier
      uuid?: string
    }[]
  }
  // Contact person for any issue regarding the reservation.
  contact_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Contact details of the final user
  recipient_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Whether recipient's notifications are enabled.
  recipient_notifications_enabled?: boolean
}
```

### #/components/schemas/reservations1

```ts
// Reservation with shipping
{
  // Reservation identifier
  id?: string
  // Custom package identifier
  shipment_id?: string
  // Whether the reservation is confirmed on the locker
  is_confirmed?: boolean
  // Whether the reservation is in local mode
  is_local?: boolean
  // Location identifier
  location?: string
  // Package information provided
  package_info: {
    // Package width in cm
    width?: number
    // Package height in cm
    height?: number
    // Package length in cm
    length?: number
    // Package estimated value in MXN
    value?: number
    // Package weight in kg
    weight?: number
    // Package description, useful for logistic services, use a comma to split multiple descriptions
    description?: string
  }
  // Delivery date limit in ISO 8601 format
  delivery_due_date?: string
  // Pick up date limit in ISO 8601 format
  pickup_due_date?: string
  // #/components/schemas/EventHistory
  event_history: {
    status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
    // Update date in ISO 8601 format
    date?: string
  }[]
  // Additional reservation data
  additional: {
    string?: string
  }
  // Shipping information
  shipping: {
    // Carrier tracking id
    tracking_id?: string
    pickup_point: {
      // Courier pick up address
      address: string
      // Courier pick up notes
      notes?: string
    }
    destination_point: {
      // Courier destination address
      address: string
      // Courier destination notes
      notes?: string
    }
    // Carrier used
    carrier: string
    // Vehicle type quoted
    vehicle_type?: string
    // Shipping cost in MXN
    fee: string
    // Carrier recollection date
    required_pickup_date?: string //default: 2020-09-08T00:26:35.817Z
    // Carrier estimated delivery date
    estimated_delivery_date?: string //default: 2020-09-08T00:28:35.817Z
  }
  // Reservation creation date in ISO 8601 format
  created_at?: string
  // Reservation last update date in ISO 8601 format
  updated_at?: string
  // Assigned box information
  locker: {
    // Assigned box size
    size?: string
    // Assigned box identifier
    uuid?: string
    // Whether the box belongs to public network
    is_public?: boolean
    // #/components/schemas/boxes1
    boxes: {
      // Box size
      size?: string
      // Box identifier
      uuid?: string
    }[]
  }
  // Contact person for any issue regarding the reservation.
  contact_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Contact details of the final user
  recipient_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Whether recipient's notifications are enabled.
  recipient_notifications_enabled?: boolean
}[]
```

### #/components/schemas/PaginationResult

```ts
// Reservations with shipping
{
  status_code?: enum[200]
  // #/components/schemas/reservations1
  // Reservation with shipping
  reservations: {
    // Reservation identifier
    id?: string
    // Custom package identifier
    shipment_id?: string
    // Whether the reservation is confirmed on the locker
    is_confirmed?: boolean
    // Whether the reservation is in local mode
    is_local?: boolean
    // Location identifier
    location?: string
    // Package information provided
    package_info: {
      // Package width in cm
      width?: number
      // Package height in cm
      height?: number
      // Package length in cm
      length?: number
      // Package estimated value in MXN
      value?: number
      // Package weight in kg
      weight?: number
      // Package description, useful for logistic services, use a comma to split multiple descriptions
      description?: string
    }
    // Delivery date limit in ISO 8601 format
    delivery_due_date?: string
    // Pick up date limit in ISO 8601 format
    pickup_due_date?: string
    // #/components/schemas/EventHistory
    event_history: {
      status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
      // Update date in ISO 8601 format
      date?: string
    }[]
    // Additional reservation data
    additional: {
      string?: string
    }
    // Shipping information
    shipping: {
      // Carrier tracking id
      tracking_id?: string
      pickup_point: {
        // Courier pick up address
        address: string
        // Courier pick up notes
        notes?: string
      }
      destination_point: {
        // Courier destination address
        address: string
        // Courier destination notes
        notes?: string
      }
      // Carrier used
      carrier: string
      // Vehicle type quoted
      vehicle_type?: string
      // Shipping cost in MXN
      fee: string
      // Carrier recollection date
      required_pickup_date?: string //default: 2020-09-08T00:26:35.817Z
      // Carrier estimated delivery date
      estimated_delivery_date?: string //default: 2020-09-08T00:28:35.817Z
    }
    // Reservation creation date in ISO 8601 format
    created_at?: string
    // Reservation last update date in ISO 8601 format
    updated_at?: string
    // Assigned box information
    locker: {
      // Assigned box size
      size?: string
      // Assigned box identifier
      uuid?: string
      // Whether the box belongs to public network
      is_public?: boolean
      // #/components/schemas/boxes1
      boxes: {
        // Box size
        size?: string
        // Box identifier
        uuid?: string
      }[]
    }
    // Contact person for any issue regarding the reservation.
    contact_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Contact details of the final user
    recipient_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Whether recipient's notifications are enabled.
    recipient_notifications_enabled?: boolean
  }[]
  // Total number of results
  total?: number
  // Page number returned
  page?: number
  // Maximum number of results per page
  page_size?: number
  // Total number of pages
  total_pages?: number
}
```

### #/components/schemas/Model11

```ts
{
  status_code?: number //default: 200
  message?: string
}
```

### #/components/schemas/phones

```ts
// Contact number
string[]
```

### #/components/schemas/FullAddress

```ts
{
  delivery_info?: string
  city?: string
  state?: string
  country?: string //default: Mexico
  street: string
  number: string
  // Address borough/suburb
  town: string
  zip_code: string
  // Address references
  references?: string
  // Cross street
  cross_streets?: string
}
```

### #/components/schemas/Model12

```ts
{
  // Contact name
  name: string
  // Contact name
  last_name: string
  email?: string
  gender: enum[male, female, unspecified]
  age: integer
  // #/components/schemas/phones
  // Contact number
  phones?: string[]
  address: {
    delivery_info?: string
    city?: string
    state?: string
    country?: string //default: Mexico
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
  // Directory identifier
  directory?: string
}
```

### #/components/schemas/contacts

```ts
{
  // Contact name
  name: string
  // Contact name
  last_name: string
  email?: string
  gender: enum[male, female, unspecified]
  age: integer
  // #/components/schemas/phones
  // Contact number
  phones?: string[]
  address: {
    delivery_info?: string
    city?: string
    state?: string
    country?: string //default: Mexico
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
  // Directory identifier
  directory?: string
}[]
```

### #/components/schemas/locations1

```ts
{
  // Resource identifier
  id: string
  // Locker identifier
  uuid: string
  // Locker business name
  internal_name: string
  // Locker host address
  address: {
    // Whether it has a parking space
    parking?: boolean
    city?: string
    state?: string
    country?: string //default: Mexico
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
}[]
```

### #/components/schemas/DirectoryCompany1

```ts
{
  // #/components/schemas/contacts
  contacts: {
    // Contact name
    name: string
    // Contact name
    last_name: string
    email?: string
    gender: enum[male, female, unspecified]
    age: integer
    // #/components/schemas/phones
    // Contact number
    phones?: string[]
    address: {
      delivery_info?: string
      city?: string
      state?: string
      country?: string //default: Mexico
      street: string
      number: string
      // Address borough/suburb
      town: string
      zip_code: string
      // Address references
      references?: string
      // Cross street
      cross_streets?: string
    }
    // Directory identifier
    directory?: string
  }[]
  // #/components/schemas/locations1
  locations: {
    // Resource identifier
    id: string
    // Locker identifier
    uuid: string
    // Locker business name
    internal_name: string
    // Locker host address
    address: {
      // Whether it has a parking space
      parking?: boolean
      city?: string
      state?: string
      country?: string //default: Mexico
      street: string
      number: string
      // Address borough/suburb
      town: string
      zip_code: string
      // Address references
      references?: string
      // Cross street
      cross_streets?: string
    }
  }[]
  isActive: boolean //default: true
  // Directory identifier
  id: string
  company: {
    // Company name
    name?: string
    // Resource identifier
    id?: string
  }
  author: {
    // Author name
    name?: string
    // Resource identifier
    id?: string
  }
  // Directory name
  name: string
  created_at: string
  updated_at?: string
}
```

### #/components/schemas/Model13

```ts
{
  directory: {
    // #/components/schemas/contacts
    contacts: {
      // Contact name
      name: string
      // Contact name
      last_name: string
      email?: string
      gender: enum[male, female, unspecified]
      age: integer
      // #/components/schemas/phones
      // Contact number
      phones?: string[]
      address: {
        delivery_info?: string
        city?: string
        state?: string
        country?: string //default: Mexico
        street: string
        number: string
        // Address borough/suburb
        town: string
        zip_code: string
        // Address references
        references?: string
        // Cross street
        cross_streets?: string
      }
      // Directory identifier
      directory?: string
    }[]
    // #/components/schemas/locations1
    locations: {
      // Resource identifier
      id: string
      // Locker identifier
      uuid: string
      // Locker business name
      internal_name: string
      // Locker host address
      address: {
        // Whether it has a parking space
        parking?: boolean
        city?: string
        state?: string
        country?: string //default: Mexico
        street: string
        number: string
        // Address borough/suburb
        town: string
        zip_code: string
        // Address references
        references?: string
        // Cross street
        cross_streets?: string
      }
    }[]
    isActive: boolean //default: true
    // Directory identifier
    id: string
    company: {
      // Company name
      name?: string
      // Resource identifier
      id?: string
    }
    author: {
      // Author name
      name?: string
      // Resource identifier
      id?: string
    }
    // Directory name
    name: string
    created_at: string
    updated_at?: string
  }
  status_code?: enum[200]
}
```

### #/components/schemas/Model14

```ts
{
  status_code?: enum[200]
  location: {
    // #/components/schemas/boxes
    boxes: {
      size: enum[s, m, l, xl]
      // Total boxes of size. Only for private lockers
      total?: number
      // Total occupied boxes of size. Only for private lockers
      occupied?: number
      // Whether the locker has boxes available in this size
      available?: boolean
    }[]
    // Number of boxes in the public network being used
    public_boxes?: number
    // Resource identifier
    id: string
    // Locker identifier
    uuid: string
    // Locker business name
    internal_name: string
    // Locker host address
    address: {
      // Whether it has a parking space
      parking?: boolean
      city?: string
      state?: string
      country?: string //default: Mexico
      street: string
      number: string
      // Address borough/suburb
      town: string
      zip_code: string
      // Address references
      references?: string
      // Cross street
      cross_streets?: string
    }
    // Host location coordinates
    coords: {
      latitude: number
      longitude: number
    }
    // Contact number
    phone?: string
    // #/components/schemas/service_days
    service_days: {
      weekday: number
      string_weekday: enum[monday, tuesday, wednesday, thursday, friday, saturday, sunday]
      // Opening hour in hh:mm format
      opening_time: string //default: 09:00
      // Closing hour in hh:mm format
      closing_time: string //default: 18:00
    }[]
    timezone: string
  }
}
```

### #/components/schemas/Reservation2

```ts
{
  // Reservation identifier
  id?: string
  // Custom package identifier
  shipment_id?: string
  // Whether the reservation is confirmed on the locker
  is_confirmed?: boolean
  // Whether the reservation is in local mode
  is_local?: boolean
  // Location identifier
  location?: string
  // Package information provided
  package_info: {
    // Package width in cm
    width?: number
    // Package height in cm
    height?: number
    // Package length in cm
    length?: number
    // Package estimated value in MXN
    value?: number
    // Package weight in kg
    weight?: number
    // Package description, useful for logistic services, use a comma to split multiple descriptions
    description?: string
  }
  // Delivery date limit in ISO 8601 format
  delivery_due_date?: string
  // Pick up date limit in ISO 8601 format
  pickup_due_date?: string
  // #/components/schemas/EventHistory
  event_history: {
    status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
    // Update date in ISO 8601 format
    date?: string
  }[]
  // Additional reservation data
  additional: {
    string?: string
  }
  shipping: {
  }
  // Reservation creation date in ISO 8601 format
  created_at?: string
  // Reservation last update date in ISO 8601 format
  updated_at?: string
  // Assigned box information
  locker: {
    // Assigned box size
    size?: string
    // Assigned box identifier
    uuid?: string
    // Whether the box belongs to public network
    is_public?: boolean
    // #/components/schemas/boxes1
    boxes: {
      // Box size
      size?: string
      // Box identifier
      uuid?: string
    }[]
  }
  // Contact person for any issue regarding the reservation.
  contact_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Contact details of the final user
  recipient_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Whether recipient's notifications are enabled.
  recipient_notifications_enabled?: boolean
}
```

### #/components/schemas/reservations2

```ts
{
  // Reservation identifier
  id?: string
  // Custom package identifier
  shipment_id?: string
  // Whether the reservation is confirmed on the locker
  is_confirmed?: boolean
  // Whether the reservation is in local mode
  is_local?: boolean
  // Location identifier
  location?: string
  // Package information provided
  package_info: {
    // Package width in cm
    width?: number
    // Package height in cm
    height?: number
    // Package length in cm
    length?: number
    // Package estimated value in MXN
    value?: number
    // Package weight in kg
    weight?: number
    // Package description, useful for logistic services, use a comma to split multiple descriptions
    description?: string
  }
  // Delivery date limit in ISO 8601 format
  delivery_due_date?: string
  // Pick up date limit in ISO 8601 format
  pickup_due_date?: string
  // #/components/schemas/EventHistory
  event_history: {
    status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
    // Update date in ISO 8601 format
    date?: string
  }[]
  // Additional reservation data
  additional: {
    string?: string
  }
  shipping: {
  }
  // Reservation creation date in ISO 8601 format
  created_at?: string
  // Reservation last update date in ISO 8601 format
  updated_at?: string
  // Assigned box information
  locker: {
    // Assigned box size
    size?: string
    // Assigned box identifier
    uuid?: string
    // Whether the box belongs to public network
    is_public?: boolean
    // #/components/schemas/boxes1
    boxes: {
      // Box size
      size?: string
      // Box identifier
      uuid?: string
    }[]
  }
  // Contact person for any issue regarding the reservation.
  contact_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Contact details of the final user
  recipient_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Whether recipient's notifications are enabled.
  recipient_notifications_enabled?: boolean
}[]
```

### #/components/schemas/ReservationListResponse1

```ts
{
  // #/components/schemas/reservations2
  reservations: {
    // Reservation identifier
    id?: string
    // Custom package identifier
    shipment_id?: string
    // Whether the reservation is confirmed on the locker
    is_confirmed?: boolean
    // Whether the reservation is in local mode
    is_local?: boolean
    // Location identifier
    location?: string
    // Package information provided
    package_info: {
      // Package width in cm
      width?: number
      // Package height in cm
      height?: number
      // Package length in cm
      length?: number
      // Package estimated value in MXN
      value?: number
      // Package weight in kg
      weight?: number
      // Package description, useful for logistic services, use a comma to split multiple descriptions
      description?: string
    }
    // Delivery date limit in ISO 8601 format
    delivery_due_date?: string
    // Pick up date limit in ISO 8601 format
    pickup_due_date?: string
    // #/components/schemas/EventHistory
    event_history: {
      status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
      // Update date in ISO 8601 format
      date?: string
    }[]
    // Additional reservation data
    additional: {
      string?: string
    }
    shipping: {
    }
    // Reservation creation date in ISO 8601 format
    created_at?: string
    // Reservation last update date in ISO 8601 format
    updated_at?: string
    // Assigned box information
    locker: {
      // Assigned box size
      size?: string
      // Assigned box identifier
      uuid?: string
      // Whether the box belongs to public network
      is_public?: boolean
      // #/components/schemas/boxes1
      boxes: {
        // Box size
        size?: string
        // Box identifier
        uuid?: string
      }[]
    }
    // Contact person for any issue regarding the reservation.
    contact_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Contact details of the final user
    recipient_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Whether recipient's notifications are enabled.
    recipient_notifications_enabled?: boolean
  }[]
  status_code?: enum[200]
  // Total number of results
  total?: number
  // Page number returned
  page?: number
  // Maximum number of results per page
  page_size?: number
  // Total number of pages
  total_pages?: number
}
```

### #/components/schemas/Model15

```ts
{
  // Tracking guide in JPEG format
  jpeg: string
  // Tracking guide in PDF format
  pdf: string
}
```

### #/components/schemas/Error4

```ts
{
  // HTTP status code
  status_code?: enum[422]
  // Error type
  type?: string
  // Human readable message
  message: string
}
```

### #/components/schemas/Model16

```ts
{
  // Code for SAT messurments and packing
  units_weight: string
  // Code for SAT products and services
  product_service: string
  // Code for SAT dangerous materials
  dangerous_material?: string
  // Code for SAT packing type
  pack_type?: string
  // Quantity of the same object
  units?: string
  // Value to know if the item is dangerous
  is_dangerous: boolean
}
```

### #/components/schemas/items

```ts
{
  // Code for SAT messurments and packing
  units_weight: string
  // Code for SAT products and services
  product_service: string
  // Code for SAT dangerous materials
  dangerous_material?: string
  // Code for SAT packing type
  pack_type?: string
  // Quantity of the same object
  units?: string
  // Value to know if the item is dangerous
  is_dangerous: boolean
}[]
```

### #/components/schemas/PackageInfoRequest

```ts
// Additional package information
{
  // Package width in cm
  width?: number
  // Package height in cm
  height?: number
  // Package length in cm
  length?: number
  // Package estimated value in MXN
  value?: number
  // Package weight in kg
  weight: number
  // Package description, useful for logistic services, use a comma to split multiple descriptions
  description?: string
  // #/components/schemas/items
  items: {
    // Code for SAT messurments and packing
    units_weight: string
    // Code for SAT products and services
    product_service: string
    // Code for SAT dangerous materials
    dangerous_material?: string
    // Code for SAT packing type
    pack_type?: string
    // Quantity of the same object
    units?: string
    // Value to know if the item is dangerous
    is_dangerous: boolean
  }[]
}
```

### #/components/schemas/contact

```ts
{
  phone: string
  // Contact name
  name?: string
  email: string
  // Company name
  company?: string
  // Contact taxpayer number
  rfc?: string
}
```

### #/components/schemas/FullAddressShipping

```ts
// Courier pick up address. If the request has `destination` and `location_id` a 400 error is returned.
{
  contact: {
    phone: string
    // Contact name
    name?: string
    email: string
    // Company name
    company?: string
    // Contact taxpayer number
    rfc?: string
  }
  street_type?: string
  town_type?: string
  city?: string
  state?: string
  country?: string //default: Mexico
  street: string
  number: string
  // Address borough/suburb
  town: string
  zip_code: string
  // Address references
  references?: string
  // Cross street
  cross_streets?: string
}
```

### #/components/schemas/FullAddressShipping1

```ts
// Courier delivery address
{
  contact: {
    phone: string
    // Contact name
    name?: string
    email: string
    // Company name
    company?: string
    // Contact taxpayer number
    rfc?: string
  }
  street_type?: string
  town_type?: string
  city?: string
  state?: string
  country?: string //default: Mexico
  street: string
  number: string
  // Address borough/suburb
  town: string
  zip_code: string
  // Address references
  references?: string
  // Cross street
  cross_streets?: string
}
```

### #/components/schemas/Model17

```ts
{
  // Preffered carrier
  preferred: string
  // Additional package information
  package_info: {
    // Package width in cm
    width?: number
    // Package height in cm
    height?: number
    // Package length in cm
    length?: number
    // Package estimated value in MXN
    value?: number
    // Package weight in kg
    weight: number
    // Package description, useful for logistic services, use a comma to split multiple descriptions
    description?: string
    // #/components/schemas/items
    items: {
      // Code for SAT messurments and packing
      units_weight: string
      // Code for SAT products and services
      product_service: string
      // Code for SAT dangerous materials
      dangerous_material?: string
      // Code for SAT packing type
      pack_type?: string
      // Quantity of the same object
      units?: string
      // Value to know if the item is dangerous
      is_dangerous: boolean
    }[]
  }
  // Location identifier for delivery. If the request has `destination` and `location_id` a 400 error is returned.
  location_id?: string
  // Courier pick up address. If the request has `destination` and `location_id` a 400 error is returned.
  destination: {
    contact: {
      phone: string
      // Contact name
      name?: string
      email: string
      // Company name
      company?: string
      // Contact taxpayer number
      rfc?: string
    }
    street_type?: string
    town_type?: string
    city?: string
    state?: string
    country?: string //default: Mexico
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
  // Courier delivery address
  origin: {
    contact:#/components/schemas/contact
    street_type?: string
    town_type?: string
    city?: string
    state?: string
    country?: string //default: Mexico
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
}
```

### #/components/schemas/directory

```ts
{
  // Directory name
  name: string
}
```

### #/components/schemas/DirectoryBody

```ts
{
  directory: {
    // Directory name
    name: string
  }
}
```

### #/components/schemas/Error5

```ts
{
  // HTTP status code
  status_code?: enum[409]
  // Error type
  type?: string
  // Human readable message
  message: string
}
```

### #/components/schemas/Error6

```ts
// Unable to complete request
{
  // HTTP status code
  status_code?: enum[424]
  // Error type
  type?: string
  // Human readable message
  message: string
}
```

### #/components/schemas/coords

```ts
// Coordinates for which to find the closest locations.
{
  // Center longitude
  long: number
  // Center latitude
  lat: number
}
```

### #/components/schemas/Model18

```ts
{
  // Coordinates for which to find the closest locations.
  coords: {
    // Center longitude
    long: number
    // Center latitude
    lat: number
  }
  // Zip code for which to find the closest locations.
  zip_code?: string
  // The maximum distance in meters from the center point that the locations can be.
  max_distance?: number //default: 50000
  // Maximum number of locations to return.
  limit: number
  // Size for which to find an available box
  size?: enum[s, m, l, xl]
}
```

### #/components/schemas/LocationAvailable

```ts
{
  // Distance in meters from the center
  distance: number
  // Locker host name
  host?: string
  // Whether the locker has 24hrs access
  is_24?: boolean
  // Host location coordinates
  coords: {
    latitude: number
    longitude: number
  }
  // Contact number
  phone?: string
  // #/components/schemas/service_days
  service_days: {
    weekday: number
    string_weekday: enum[monday, tuesday, wednesday, thursday, friday, saturday, sunday]
    // Opening hour in hh:mm format
    opening_time: string //default: 09:00
    // Closing hour in hh:mm format
    closing_time: string //default: 18:00
  }[]
  timezone: string
  // Locker business name
  internal_name: string
  // Locker host address
  address: {
    // Whether it has a parking space
    parking?: boolean
    city?: string
    state?: string
    country?: string //default: Mexico
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
  // Resource identifier
  id: string
  // Locker identifier
  uuid: string
}
```

### #/components/schemas/locations2

```ts
{
  // Distance in meters from the center
  distance: number
  // Locker host name
  host?: string
  // Whether the locker has 24hrs access
  is_24?: boolean
  // Host location coordinates
  coords: {
    latitude: number
    longitude: number
  }
  // Contact number
  phone?: string
  // #/components/schemas/service_days
  service_days: {
    weekday: number
    string_weekday: enum[monday, tuesday, wednesday, thursday, friday, saturday, sunday]
    // Opening hour in hh:mm format
    opening_time: string //default: 09:00
    // Closing hour in hh:mm format
    closing_time: string //default: 18:00
  }[]
  timezone: string
  // Locker business name
  internal_name: string
  // Locker host address
  address: {
    // Whether it has a parking space
    parking?: boolean
    city?: string
    state?: string
    country?: string //default: Mexico
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
  // Resource identifier
  id: string
  // Locker identifier
  uuid: string
}[]
```

### #/components/schemas/Model19

```ts
{
  status_code?: enum[200]
  // #/components/schemas/locations2
  locations: {
    // Distance in meters from the center
    distance: number
    // Locker host name
    host?: string
    // Whether the locker has 24hrs access
    is_24?: boolean
    // Host location coordinates
    coords: {
      latitude: number
      longitude: number
    }
    // Contact number
    phone?: string
    // #/components/schemas/service_days
    service_days: {
      weekday: number
      string_weekday: enum[monday, tuesday, wednesday, thursday, friday, saturday, sunday]
      // Opening hour in hh:mm format
      opening_time: string //default: 09:00
      // Closing hour in hh:mm format
      closing_time: string //default: 18:00
    }[]
    timezone: string
    // Locker business name
    internal_name: string
    // Locker host address
    address: {
      // Whether it has a parking space
      parking?: boolean
      city?: string
      state?: string
      country?: string //default: Mexico
      street: string
      number: string
      // Address borough/suburb
      town: string
      zip_code: string
      // Address references
      references?: string
      // Cross street
      cross_streets?: string
    }
    // Resource identifier
    id: string
    // Locker identifier
    uuid: string
  }[]
  // Token granted to create a pre-reservation in the locations returned
  grant?: string
}
```

### #/components/schemas/Error7

```ts
// Unable to find a box with required size or provided zip code is invalid.
{
  // HTTP status code
  status_code?: enum[422]
  // Error type
  type?: enum[BOX_INCOMPATIBLE, INVALID_ADDRESS]
  // Human readable message
  message: string
}
```

### #/components/schemas/Error8

```ts
// Unexpected error
{
  // HTTP status code
  status_code?: enum[500]
  // Error type
  type?: string //default: UNKNOWN_ERROR
  // Human readable message
  message: string
}
```

### #/components/schemas/directoriesIds

```ts
string[]
```

### #/components/schemas/Model20

```ts
{
  // #/components/schemas/directoriesIds
  directoriesIds?: string[]
  csv?: string
}
```

### #/components/schemas/PackageInfoRequest1

```ts
// Additional package information
{
  // Package estimated value in MXN
  value?: number
  // Package weight in kg
  weight?: number
  // Package description, useful for logistic services, use a comma to split multiple descriptions
  description?: string
}
```

### #/components/schemas/contact_info1

```ts
// Contact person for any issue regarding the reservation. It should contain either name or email.
{
  name?: string
  email?: string
  phone?: string
}
```

### #/components/schemas/recipient_info1

```ts
// Contact details of the final user
{
  name?: string
  email?: string
  phone?: string
}
```

### #/components/schemas/PreReservationRequest

```ts
{
  // Size for which to find an available box
  size?: enum[s, m, l, xl]
  // Expected delivery date in ISO 8601 format. Defaults to 24 hours from request
  delivery_date?: string
  delivery_confirmation_enabled?: boolean
  // Custom delivery token.
  delivery_token?: string
  // Your identifier for the reservation.
  shipment_id?: string
  // Additional package information
  package_info: {
    // Package estimated value in MXN
    value?: number
    // Package weight in kg
    weight?: number
    // Package description, useful for logistic services, use a comma to split multiple descriptions
    description?: string
  }
  // Custom reservation information (up to 30 keys). Any key starting with `lok` will be ignored
  additional: {
    string?: string
  }
  // Contact person for any issue regarding the reservation. It should contain either name or email.
  contact_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Contact details of the final user
  recipient_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Whether to send the pick up token to recipient's email.
  recipient_notifications_enabled?: boolean
}
```

### #/components/schemas/Model21

```ts
{
  reservation: {
    // Reservation identifier
    id?: string
    // Custom package identifier
    shipment_id?: string
    // Whether the reservation is confirmed on the locker
    is_confirmed?: boolean
    // Whether the reservation is in local mode
    is_local?: boolean
    // Location identifier
    location?: string
    // Package information provided
    package_info: {
      // Package width in cm
      width?: number
      // Package height in cm
      height?: number
      // Package length in cm
      length?: number
      // Package estimated value in MXN
      value?: number
      // Package weight in kg
      weight?: number
      // Package description, useful for logistic services, use a comma to split multiple descriptions
      description?: string
    }
    // Delivery date limit in ISO 8601 format
    delivery_due_date?: string
    // Pick up date limit in ISO 8601 format
    pickup_due_date?: string
    // #/components/schemas/EventHistory
    event_history: {
      status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
      // Update date in ISO 8601 format
      date?: string
    }[]
    // Additional reservation data
    additional: {
      string?: string
    }
    shipping: {
    }
    // Reservation creation date in ISO 8601 format
    created_at?: string
    // Reservation last update date in ISO 8601 format
    updated_at?: string
    // Assigned box information
    locker: {
      // Assigned box size
      size?: string
      // Assigned box identifier
      uuid?: string
      // Whether the box belongs to public network
      is_public?: boolean
      // #/components/schemas/boxes1
      boxes: {
        // Box size
        size?: string
        // Box identifier
        uuid?: string
      }[]
    }
    // Contact person for any issue regarding the reservation.
    contact_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Contact details of the final user
    recipient_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Whether recipient's notifications are enabled.
    recipient_notifications_enabled?: boolean
  }
  // Reservation tokens with QR
  tokens: {
    delivery_token?: string
    delivery_token_qr?: string
    reopen_1_token?: string
    reopen_1_token_qr?: string
    reopen_2_token?: string
    reopen_2_token_qr?: string
    pickup_token?: string
    pickup_token_qr?: string
    cancel_token?: string
    cancel_token_qr?: string
  }
  confirmation_code?: string
  status_code?: number
}
```

### #/components/schemas/Error9

```ts
// The dimensions provided cannot be held by any box
{
  // HTTP status code
  status_code?: enum[422]
  // Error type
  type?: enum[BOX_INCOMPATIBLE]
  // Human readable message
  message: string
}
```

### #/components/schemas/size

```ts
// Required box size
{
  // Package width in cm
  width?: number
  // Package height in cm
  height?: number
  // Package length in cm
  length?: number
}
```

### #/components/schemas/PackageInfoRequest2

```ts
// Additional package information
{
  // Package estimated value in MXN
  value?: number
  // Package weight in kg
  weight: number
  // Package description, useful for logistic services, use a comma to split multiple descriptions
  description?: string
  // Request package insurance
  request_insurance?: boolean
  // Request thermal label size pdf
  request_thermal_lable?: boolean
}
```

### #/components/schemas/Address

```ts
// Courier pick up address
{
  street: string
  number: string
  // Address borough/suburb
  town: string
  zip_code: string
  // Address references
  references?: string
  // Cross street
  cross_streets?: string
}
```

### #/components/schemas/contact1

```ts
// Contact person
{
  phone: string
  // Contact name
  name?: string
}
```

### #/components/schemas/Model22

```ts
{
  // Required box size
  size: {
    // Package width in cm
    width?: number
    // Package height in cm
    height?: number
    // Package length in cm
    length?: number
  }
  // Custom package identifier
  shipment_id: string
  // Additional package information
  package_info: {
    // Package estimated value in MXN
    value?: number
    // Package weight in kg
    weight: number
    // Package description, useful for logistic services, use a comma to split multiple descriptions
    description?: string
    // Request package insurance
    request_insurance?: boolean
    // Request thermal label size pdf
    request_thermal_lable?: boolean
  }
  // Required pick up date
  pickup_date: string
  // Courier pick up address
  origin: {
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
  // Contact person
  contact: {
    phone: string
    // Contact name
    name?: string
  }
}
```

### #/components/schemas/Model23

```ts
{
  status_code?: enum[200]
  // Custom package identifier
  shipment_id?: string
  // Tracking guide identifier
  tracking_id?: string
  // Carrier used
  carrier: string
  // Shipping cost in MXN
  fee: string
  // Carrier estimated delivery date
  estimated_delivery_date?: string //default: 2020-09-08T00:28:35.817Z
  // Tracking guide in PDF format
  label: string
  // Resource identifier
  reservation_id?: string
}
```

### #/components/schemas/ReservationRequest

```ts
{
  // Size for which to find an available box
  size: enum[s, m, l, xl]
  // Custom delivery token.
  delivery_token?: string
  delivery_confirmation_enabled?: boolean
  // Your identifier for the reservation.
  shipment_id?: string
  // Additional package information
  package_info: {
    // Package estimated value in MXN
    value?: number
    // Package weight in kg
    weight?: number
    // Package description, useful for logistic services, use a comma to split multiple descriptions
    description?: string
  }
  // Custom reservation information (up to 30 keys). Any key starting with `lok` will be ignored
  additional: {
    string?: string
  }
  // Contact person for any issue regarding the reservation. It should contain either name or email.
  contact_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Contact details of the final user
  recipient_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Whether to send the pick up token to recipient's email.
  recipient_notifications_enabled?: boolean
}
```

### #/components/schemas/Error10

```ts
// Insufficient scope in requested location
{
  // HTTP status code
  status_code?: enum[403]
  // Error type
  type?: enum[INSUFFICIENT_SCOPE, FORBIDDEN_LOCATION, GRANT_TOKEN]
  // Human readable message
  message: string
}
```

### #/components/schemas/Error11

```ts
// Shipment id already exists within the company
{
  // HTTP status code
  status_code?: enum[409]
  // Error type
  type?: enum[UNIQUE_SHIPMENT]
  // Human readable message
  message: string
}
```

### #/components/schemas/Error12

```ts
// Unable to assign a box with required size
{
  // HTTP status code
  status_code?: enum[422]
  // Error type
  type?: enum[BOX_INCOMPATIBLE, NO_CAPACITY]
  // Human readable message
  message: string
}
```

### #/components/schemas/Model24

```ts
{
  // Courier pick up address
  origin: {
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
  // Contact person
  contact: {
    phone: string
    // Contact name
    name?: string
  }
}
```

### #/components/schemas/tokens1

```ts
// Reservation tokens and shipping order id with encoded images
{
  delivery_token?: string
  delivery_token_qr?: string
  reopen_1_token?: string
  reopen_1_token_qr?: string
  reopen_2_token?: string
  reopen_2_token_qr?: string
  pickup_token?: string
  pickup_token_qr?: string
  cancel_token?: string
  cancel_token_qr?: string
}
```

### #/components/schemas/Model25

```ts
{
  status_code?: enum[201]
  // Reservation with shipping
  reservation: {
    // Reservation identifier
    id?: string
    // Custom package identifier
    shipment_id?: string
    // Whether the reservation is confirmed on the locker
    is_confirmed?: boolean
    // Whether the reservation is in local mode
    is_local?: boolean
    // Location identifier
    location?: string
    // Package information provided
    package_info: {
      // Package width in cm
      width?: number
      // Package height in cm
      height?: number
      // Package length in cm
      length?: number
      // Package estimated value in MXN
      value?: number
      // Package weight in kg
      weight?: number
      // Package description, useful for logistic services, use a comma to split multiple descriptions
      description?: string
    }
    // Delivery date limit in ISO 8601 format
    delivery_due_date?: string
    // Pick up date limit in ISO 8601 format
    pickup_due_date?: string
    // #/components/schemas/EventHistory
    event_history: {
      status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
      // Update date in ISO 8601 format
      date?: string
    }[]
    // Additional reservation data
    additional: {
      string?: string
    }
    // Shipping information
    shipping: {
      // Carrier tracking id
      tracking_id?: string
      pickup_point: {
        // Courier pick up address
        address: string
        // Courier pick up notes
        notes?: string
      }
      destination_point: {
        // Courier destination address
        address: string
        // Courier destination notes
        notes?: string
      }
      // Carrier used
      carrier: string
      // Vehicle type quoted
      vehicle_type?: string
      // Shipping cost in MXN
      fee: string
      // Carrier recollection date
      required_pickup_date?: string //default: 2020-09-08T00:26:35.817Z
      // Carrier estimated delivery date
      estimated_delivery_date?: string //default: 2020-09-08T00:28:35.817Z
    }
    // Reservation creation date in ISO 8601 format
    created_at?: string
    // Reservation last update date in ISO 8601 format
    updated_at?: string
    // Assigned box information
    locker: {
      // Assigned box size
      size?: string
      // Assigned box identifier
      uuid?: string
      // Whether the box belongs to public network
      is_public?: boolean
      // #/components/schemas/boxes1
      boxes: {
        // Box size
        size?: string
        // Box identifier
        uuid?: string
      }[]
    }
    // Contact person for any issue regarding the reservation.
    contact_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Contact details of the final user
    recipient_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Whether recipient's notifications are enabled.
    recipient_notifications_enabled?: boolean
  }
  // Reservation tokens and shipping order id with encoded images
  tokens: {
    delivery_token?: string
    delivery_token_qr?: string
    reopen_1_token?: string
    reopen_1_token_qr?: string
    reopen_2_token?: string
    reopen_2_token_qr?: string
    pickup_token?: string
    pickup_token_qr?: string
    cancel_token?: string
    cancel_token_qr?: string
  }
}
```

### #/components/schemas/Error13

```ts
// Reservation cannot longer be updated
{
  // HTTP status code
  status_code?: enum[410]
  // Error type
  type?: enum[INVALID_RESERVATION]
  // Human readable message
  message: string
}
```

### #/components/schemas/Error14

```ts
// Reservation has missing information
{
  // HTTP status code
  status_code?: enum[422]
  // Error type
  type?: enum[INVALID_RESERVATION]
  // Human readable message
  message: string
}
```

### #/components/schemas/Error15

```ts
// Failed to create order with carrier
{
  // HTTP status code
  status_code?: enum[424]
  // Error type
  type?: enum[LOGISTIC_ERROR]
  // Human readable message
  message: string
}
```

### #/components/schemas/quoting

```ts
// Quoting information
{
  // Courier pick up address
  origin: string
  // Locker address
  destination: string
  // Carrier used
  carrier: string
  // Vehicle type quoted
  vehicle_type?: string
  // Shipping cost in MXN
  fee: string
  // Carrier recollection date
  required_pickup_date?: string //default: 2020-09-08T00:26:35.817Z
  // Carrier estimated delivery date
  estimated_delivery_date?: string //default: 2020-09-08T00:28:35.817Z
}
```

### #/components/schemas/Model26

```ts
{
  status_code?: enum[200]
  // Quoting information
  quoting: {
    // Courier pick up address
    origin: string
    // Locker address
    destination: string
    // Carrier used
    carrier: string
    // Vehicle type quoted
    vehicle_type?: string
    // Shipping cost in MXN
    fee: string
    // Carrier recollection date
    required_pickup_date?: string //default: 2020-09-08T00:26:35.817Z
    // Carrier estimated delivery date
    estimated_delivery_date?: string //default: 2020-09-08T00:28:35.817Z
  }
}
```

### #/components/schemas/Error16

```ts
{
  // HTTP status code
  status_code?: enum[410]
  // Error type
  type?: string
  // Human readable message
  message: string
}
```

### #/components/schemas/Model27

```ts
{
  operator_email?: string
  administrator_email?: string
}
```

### #/components/schemas/Model28

```ts
{
  url: string
  // #/components/schemas/events
  events?: enum[reserved, delivered, reopened, reopened2, picked_up, cancelled, undelivered, unpicked][]
}
```

### #/components/schemas/PreReservationUpdate

```ts
// Reservation fields to update. At least one field must be sent.
{
  // Your identifier for the reservation.
  shipment_id?: string
  // Additional package information
  package_info: {
    // Package estimated value in MXN
    value?: number
    // Package weight in kg
    weight?: number
    // Package description, useful for logistic services, use a comma to split multiple descriptions
    description?: string
  }
  // Custom reservation information (up to 30 keys). Any key starting with `lok` will be ignored
  additional: {
    string?: string
  }
  // Contact person for any issue regarding the reservation. It should contain either name or email.
  contact_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Contact details of the final user
  recipient_info: {
    name?: string
    email?: string
    phone?: string
  }
  // Whether to send the pick up token to recipient's email.
  recipient_notifications_enabled?: boolean
  // Size for which to find an available box
  size?: enum[s, m, l, xl]
  // Expected delivery date in ISO 8601 format. Defaults to 24 hours from request
  delivery_date?: string
  delivery_confirmation_enabled?: boolean
  // Custom delivery token.
  delivery_token?: string
}
```

### #/components/schemas/Error17

```ts
{
  // HTTP status code
  status_code?: enum[410]
  // Error type
  type?: enum[INVALID_RESERVATION]
  // Human readable message
  message: string
}
```

### #/components/schemas/locations3

```ts
string[]
```

### #/components/schemas/directory1

```ts
{
  is_active?: boolean //default: true
  // #/components/schemas/locations3
  locations?: string[]
  // Directory name
  name: string
}
```

### #/components/schemas/DirectoryUpdateBody

```ts
{
  directory: {
    is_active?: boolean //default: true
    // #/components/schemas/locations3
    locations?: string[]
    // Directory name
    name: string
  }
}
```

### #/components/schemas/Model29

```ts
{
  enabled: boolean
}
```

### #/components/schemas/Model30

```ts
{
  // #/components/schemas/contacts
  contacts: {
    // Contact name
    name: string
    // Contact name
    last_name: string
    email?: string
    gender: enum[male, female, unspecified]
    age: integer
    // #/components/schemas/phones
    // Contact number
    phones?: string[]
    address: {
      delivery_info?: string
      city?: string
      state?: string
      country?: string //default: Mexico
      street: string
      number: string
      // Address borough/suburb
      town: string
      zip_code: string
      // Address references
      references?: string
      // Cross street
      cross_streets?: string
    }
    // Directory identifier
    directory?: string
  }[]
}
```

### #/components/schemas/Model31

```ts
{
  // Contact name
  name: string
  // Contact name
  last_name: string
  email?: string
  gender: enum[male, female, unspecified]
  age: integer
  // #/components/schemas/phones
  // Contact number
  phones?: string[]
  address: {
    delivery_info?: string
    city?: string
    state?: string
    country?: string //default: Mexico
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
  // Directory identifier
  directory?: string
  created_at: string
  updated_at?: string
}
```

### #/components/schemas/contacts1

```ts
{
  // Contact name
  name: string
  // Contact name
  last_name: string
  email?: string
  gender: enum[male, female, unspecified]
  age: integer
  // #/components/schemas/phones
  // Contact number
  phones?: string[]
  address: {
    delivery_info?: string
    city?: string
    state?: string
    country?: string //default: Mexico
    street: string
    number: string
    // Address borough/suburb
    town: string
    zip_code: string
    // Address references
    references?: string
    // Cross street
    cross_streets?: string
  }
  // Directory identifier
  directory?: string
  created_at: string
  updated_at?: string
}[]
```

### #/components/schemas/Model32

```ts
{
  // #/components/schemas/contacts1
  contacts: {
    // Contact name
    name: string
    // Contact name
    last_name: string
    email?: string
    gender: enum[male, female, unspecified]
    age: integer
    // #/components/schemas/phones
    // Contact number
    phones?: string[]
    address: {
      delivery_info?: string
      city?: string
      state?: string
      country?: string //default: Mexico
      street: string
      number: string
      // Address borough/suburb
      town: string
      zip_code: string
      // Address references
      references?: string
      // Cross street
      cross_streets?: string
    }
    // Directory identifier
    directory?: string
    created_at: string
    updated_at?: string
  }[]
}
```

### #/components/schemas/ReservationResponse

```ts
{
  reservation: {
    // Reservation identifier
    id?: string
    // Custom package identifier
    shipment_id?: string
    // Whether the reservation is confirmed on the locker
    is_confirmed?: boolean
    // Whether the reservation is in local mode
    is_local?: boolean
    // Location identifier
    location?: string
    // Package information provided
    package_info: {
      // Package width in cm
      width?: number
      // Package height in cm
      height?: number
      // Package length in cm
      length?: number
      // Package estimated value in MXN
      value?: number
      // Package weight in kg
      weight?: number
      // Package description, useful for logistic services, use a comma to split multiple descriptions
      description?: string
    }
    // Delivery date limit in ISO 8601 format
    delivery_due_date?: string
    // Pick up date limit in ISO 8601 format
    pickup_due_date?: string
    // #/components/schemas/EventHistory
    event_history: {
      status?: enum[pending, reserved, delivered, reopened, reopened2, picked_up, cancelled]
      // Update date in ISO 8601 format
      date?: string
    }[]
    // Additional reservation data
    additional: {
      string?: string
    }
    shipping: {
    }
    // Reservation creation date in ISO 8601 format
    created_at?: string
    // Reservation last update date in ISO 8601 format
    updated_at?: string
    // Assigned box information
    locker: {
      // Assigned box size
      size?: string
      // Assigned box identifier
      uuid?: string
      // Whether the box belongs to public network
      is_public?: boolean
      // #/components/schemas/boxes1
      boxes: {
        // Box size
        size?: string
        // Box identifier
        uuid?: string
      }[]
    }
    // Contact person for any issue regarding the reservation.
    contact_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Contact details of the final user
    recipient_info: {
      name?: string
      email?: string
      phone?: string
    }
    // Whether recipient's notifications are enabled.
    recipient_notifications_enabled?: boolean
  }
  status_code?: number
}
```

### #/components/schemas/Error18

```ts
// The reservation is in an ongoing state
{
  // HTTP status code
  status_code?: enum[409]
  // Error type
  type?: enum[INVALID_RESERVATION]
  // Human readable message
  message: string
}
```
