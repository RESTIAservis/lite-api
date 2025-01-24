Menu item availability API
=========================

RESTIA Lite offers display/hide menu  item in your custom API

- Client must provide webhook url to recieve menu updates
- HTTP Method of webhook must be PATCH
- We only accept "https" webhook urls
- RESTIA will provide API Key. This API Key will be used for authentication of request


Restaurant ID
---------------------
It is global identifier used across all public APIs. 
Identifier will be provided be provided by RESTIA.
- is used in [__Generic order import API__](GenericImportAPI.md#restaurant)
- is used in [__Menu item availability API__](GenericMenuItemAvailabilityAPI.md#body)
- is used in [__Profile availability API__](GenericProfileAvailabilityAPI.md.md#body)

Headers
---------------------
- All requests to client's API always contains Authorization header with API key provided by RESTIA
```
Authorization: ApiKeyProvidedByRestia
```

Show/Hide menu item request
---------------------
- Request contains body encoded as JSON string
- The JSON contains array of objects
- On success expect response HTTP 200
- Availability is determined by _isAvailable_ property, which can be __TRUE__ or __FALSE__

### Body
Field|Type|Required|Description|Example value|
|---            |---                        |---|---|---|
|id             |string                     | Y | [Restaurant ID](#restaurant-id), provided by RESTIA | 69314b087fdd7f45d9c8c3faaa5084ce  |
|items          |array of [Items](#item)    | Y | Array of items |  |


### Item
Field|Type|Required|Description|Example value|
|---            |---                |---|---|---|
|itemId         |string             | Y | Item ID, provided by RESTIA   | menu-item-1234  |
|posId          |string             | N | POS ID of item - may be null  | POS-CODE-11 |
|isAvailable    |boolean            | Y | IF is item available          | TRUE |


Examples
---------------------

### SHOW
```
{ 
    "id": "69314b087fdd7f45d9c8c3faaa5084ce",
    "items": [
        {
            "itemId": "menu-item-1234",
            "posId": "POS-CODE-11"
            "isAvailable: true
        }
    ]
}
```

#### Request Example
```
curl -i https://your-api.example.com/menu-item-availability-webhook \
    -X PATCH \
    -H 'Authorization: ApiKeyProvidedByRestia' \
    -H 'Content-Type: application/json' \
    -d '{"id":"69314b087fdd7f45d9c8c3faaa5084ce","items":[{"itemId":"menu-item-1234","posId":"POS-CODE-11","isAvailable":true}]}'
```

### HIDE
```
{ 
    "id": "69314b087fdd7f45d9c8c3faaa5084ce",
    "items": [
        {
            "itemId": "menu-item-1234",
            "posId": "POS-CODE-11"
            "isAvailable: false
        }
    ]
}
```

#### Request Example
```
curl -i https://your-api.example.com/menu-item-availability-webhook \
    -X PATCH \
    -H 'Authorization: ApiKeyProvidedByRestia' \
    -H 'Content-Type: application/json' \
    -d '{"id":"69314b087fdd7f45d9c8c3faaa5084ce","items":[{"itemId":"menu-item-1234","posId":"POS-CODE-11","isAvailable":false}]}'
```

