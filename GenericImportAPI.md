Generic Order Import API
=========================

RESTIA Lite offers webhook to import new orders from custom service as JSON object.

```
POST https://apilite.restia.cz/api/import/generic
```
with JSON payload defined bellow. \[[Examples](./payload/)\]


Restaurant ID
---------------------
It is global identifier used across all public APIs. 
Identifier will be provided by RESTIA.
- is used in [__Generic order import API__](GenericImportAPI.md#restaurant)
- is used in [__Menu item availability API__](GenericMenuItemAvailabilityAPI.md#body)
- is used in [__Profile availability API__](GenericProfileAvailabilityAPI.md#body)

Headers
---------------------
- All requests should contains Authorization header with API key provided by RESTIA
```
Authorization: ApiKeyProvidedByRestia
```


### Request Example
```
curl -i https://apilite.restia.cz/api/import/generic \
    -X POST \
    -H 'Authorization: ApiKeyProvidedByRestia' \
    -H 'Content-Type: application/json' \
    -d '{"orderNumber":"customOrderNumber","shortCode":"asdf","token":"xxxxxxxxxxxxxxxxxxx","destination":{"street":"Ulice 11","city":"Praha","zip":"11000","longitude":15,"latitude":50},"customer":{"name":"John Doe","phone":"123456789","email":"example@example.com"},"createdAt":"2021-09-01T10:36:24.544+02:00","deliveryAt":"2021-09-01T10:46:24.544+02:00","paymentType":"online","price":{"deliveryPrice":2000,"packingPrice":1000},"items":[{"id":"itemId","name":"Wings in the marinade - mix it","posId":"912_500g","count":1,"price":19000,"note":"Not spicy please","extras":[{"name":"Extra sauce","posId":"111_extras","count":1,"price":1000}]}],"restaurant":{"id":"genericRestaurantId","name":"DGC - Default Generic Restaurant"},"status":"new","note":"Please hurry, I'm hungry"}'
```


Import Order fields
-------------------

### Order

- The object bellow is expected directly in request body
- Combination of **orderNumber, shortCode** must be unique for one restaurant, otherwise order data will be ignored and API return id of already existing order

Fields of an order object:

|Field|Type|Required|Description|
|---            |---                |---|---|
|orderNumber    |string             | Y | Order identifier, e.g. ID  |
|shortCode      |string             | Y | Short code for easier identification in restaurant, should be better human readable than orderNumber  |
|token          |string             | N | Order token, optional |
|destination    |[Destination](#destination) object | N | Object with order destination data, object is required only if order is not for delivery|
|customer       |[Customer](#customer) object    | Y | Object with customer data |
|createdAt      |datetime           | Y | Datetime when order was created. <br>Is encoded as ISO8601 string |
|deliveryAt     |datetime           | Y | Expected order delivery time to the customer, if order is for pickup, this time is used as expected pickup time. <br>Is encoded as ISO8601 string|
|deliveryOnTime |boolean            | N | If true, order is not immediate and will be prepared later by 'deliveryAt' option |
|isPickup       |boolean            | N | If true, order is marked as takeaway order and deliveryAt time will be used as time ready for pickup|
|isInRestaurant |boolean            | N | If true, and __isPickup is true__ also, order is marked as in-restaurant order and deliveryAt time will be used as time ready for pickup. For delivery orders is ignored|
|paymentType    |string             | Y | Type of payment:<br> &bull; **cash** - order will be paid in cash to the courier or in restaurant if order is for pickup. <br> &bull; **card** - order will be paid with card to the courier or in restaurant if is for pickup. <br> &bull; **online** - order was already paid online.<br>**Custom type** is supported also, but in Lite API will be resolved as already paid (**online**), to the POS will be transfered with given custom value |
|price          |[Price](#price) object       | Y | Object contains summary prices |
|items          |array with [Order items](#order-item) | Y | Array contains order items|
|restaurant     |[Restaurant](#restaurant) object  | Y | Object contains restaurant informations|
|status         |string             | Y | Order status, allowed value is **new**|
|note           |string             | N | Customer note for order, can be empty|
|statusUpdateWebhookUrl|string      | N | Url of webhook where API will send [POST request](#status-update-request-data) with status updates|


### Destination

**NOTE**: If order is for delivery, destination is required and cannot be empty! If order is for pickup, destination is optional.

Fields of a destination object:

|Field|Type|Required|Description|
|---            |---                |---|---|
|street         |string             | N | Street name with house number |
|city           |string             | N | City name |
|zip            |string             | N | Postal code |
|note           |string             | N | Customer note for destination |
|longitude      |float              | N | GPS longitude coordinate, example: 14.97833 |
|latitude       |float              | N | GPS latitude coordinate, example: 50.738707 |

### Customer

Fields of a Customer object:

|Field|Type|Required|Description|
|---            |---                |---|---|
|name           |string             | Y | Customer full name|
|phone          |string             | Y | Customer phone |
|email          |string             | N | Customer email |

### Price

- All taxes are included
- Values are integers representing original price multiplied by 100. <br>**EXAMPLE**: 10€ would be 1000 
- itemsPrice will be computed from items on server after import

Fields of a Price object:

|Field|Type|Required|Description|
|---            |---                |---|---|
|deliveryPrice  |integer            | Y | Delivery price |
|packingPrice   |integer            | Y | Sum of all packing prices for whole order |
|tipPrice       |integer            | N | Tips for staff |
|surchargeToMin |integer            | N | Surcharge to order minimal price |
|discountPrice  |integer            | N | Sum of all discounts. Positive integer, which will be subtracted from total order price |

### Order Item

- Price is, same as in [Price](#price) object, integer representing original price multiplied by 100. <br>**EXAMPLE**: 10€ would be 1000 

Fields of a Order item object:

|Field|Type|Required|Description|
|---            |---                |---|---|
|id             | string \| number  | N | Custom provided ID |
|posId          | string            | Y | ID of product in POS system, cannot be empty |
|price          | integer           | Y | Unit price of item |
|count          | integer           | Y | Count of item |
|name           | string            | Y | Name/label of item |
|note           | string            | N | Customer note for item (eg. not spicy, etc.) |
|extras         | array of [Extra Order Items](#extra-order-item) | N | Array of extras, e.g. toppings etc. |

### Extra Order Item

- Same object as [Order Item](#order-item), but without extras field
- Price is, same as in [Price](#price) object, integer representing original price multiplied by 100. <br>**EXAMPLE**: 10€ would be 1000 

|Field|Type|Required|Description|
|---            |---                |---|---|
|id             | string \| number  | N | Custom provided ID |
|posId          | string            | Y | ID of product in POS system, cannot be empty |
|price          | integer           | Y | Unit price of item |
|count          | integer           | Y | Count of item |
|name           | string            | Y | Name/label of item |
|note           | string            | N | Customer note for item (eg. not spicy, etc.) |


### Restaurant

Fields of a Restaurant object:

|Field|Type|Required|Description|
|---            |---                |---|---|
|id             | string  | Y | [Restaurant ID](#restaurant-id) provided be RESTIA |
|name           | string  | N | Restaurant custom name, only for better human readable identification |



Cancel imported order API
=========================

Cancel request
----------------------------

```
POST https://apilite.restia.cz/api/import/generic
```
with JSON payload defined bellow. \[[Example](./payload/generic-order-cancel-request.json)\]

### Order cancel request data

- The object bellow is expected directly in request body
- Combination of **orderNumber, shortCode** must be same as it has imported order

Fields of an order object:

|Field|Type|Required|Description|
|---            |---                                        |---|---|
|orderNumber    |string                                     | Y | Order identifier, e.g. ID  |
|shortCode      |string                                     | Y | Short code for easier identification in restaurant, should be better human readable than orderNumber  |
|restaurant     |[Restaurant object](#restaurant)           | Y | Object contains restaurant informations |
|status         |string                                     | Y | Order status, allowed value is **canceled**|


Webhook order status update 
===========================

```
POST https://localhost/exampleStatusUpdateWebhookUrl
```
with JSON payload defined bellow. \[[Example](./payload/generic-webhook-status-update.json)\]

### Status update request data

Fields of the request payload:

|Field|Type|Required|Description|
|---            |---                                        |---|---|
|id             |string                                     | Y | ID in Restia API  |
|orderNumber    |string                                     | Y | Order identifier from [Order object](#order)  , e.g. ID  |
|restaurantId   |string                                     | Y | Object contains restaurant id used in [Restaurant object](#restaurant)  |
|status         |string                                     | Y | Order status, values \[**ready_pickup**, **dispatched**, **arrived_customer**, **delivered**\]|


Example:
----------------------------


```
curl -i https://localhost/exampleStatusUpdateWebhookUrl \
    -X POST \
    -H 'Authorization: ApiKeyProvidedByRestia' \
    -H 'Content-Type: application/json' \
    -d '{"id":"46e9e171-66a7-40f2-a819-57bce6d14abb","orderNumber":"20150101","status":"delivered","restaurantId":"genericRestaurantId"}'
```

###  Special case *canceled* status:

Fields of the request payload:

|Field|Type|Required|Description|
|---            |---                                        |---|---|
|id             |string                                     | Y | ID in Restia API  |
|orderNumber    |string                                     | Y | Order identifier from [Order object](#order)  , e.g. ID  |
|reason         |string                                     | Y | String with reason  |
|restaurantId   |string                                     | Y | Object contains restaurant id used in [Restaurant object](#restaurant)  |
|status         |string                                     | Y | Order status, values \[**canceled**\]|




Example:
----------------------------


```
curl -i https://localhost/exampleStatusUpdateWebhookUrl \
    -X POST \
    -H 'Authorization: ApiKeyProvidedByRestia' \
    -H 'Content-Type: application/json' \
    -d '{"id":"46e9e171-66a7-40f2-a819-57bce6d14abb","orderNumber":"20150101","status":"delivered","restaurantId":"genericRestaurantId","reason":"UNKNOWN"}'
```
