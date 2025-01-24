Profile availability API
=========================

RESTIA Lite offers temporary closing/opening your custom API profile

- Client must provide webhook url to recieve profile status updates
- HTTP Method of webhook must be PATCH
- We only accept "https" webhook urls
- RESTIA will provide also API Key. API Key will be used for authentication of request


Restaurant ID
---------------------
It is global identifier used across all public APIs. 
Identifier will be provided by RESTIA.
- is used in [__Generic order import API__](GenericImportAPI.md#restaurant)
- is used in [__Menu item availability API__](GenericMenuItemAvailabilityAPI.md#body)
- is used in [__Profile availability API__](GenericProfileAvailabilityAPI.md#body)

Headers
---------------------
- All requests to client's API always contains Authorization header with API key provided by RESTIA
```
Authorization: ApiKeyProvidedByRestia
```

Close profile request
---------------------
- Request contains body encoded as JSON string
- On success expect response HTTP 200


### Body
Field|Type|Required|Description|Example value|
|---            |---                |---|---|---|
|id             |string             | Y | [Restaurant ID](#restaurant-id), provided by RESTIA | 69314b087fdd7f45d9c8c3faaa5084ce  |
|status         |string             | Y | New status of profile | CLOSED |
|until          |string             | Y | The time until which the profile is closed. <br>Is encoded as ISO8601 string | 2020-11-30T10:10:00.000Z |

### Body json example
```
{
    "id": "69314b087fdd7f45d9c8c3faaa5084ce", 
    "status: "CLOSED", 
    "until": "2020-11-30T10:10:00.000Z"
}
```

### Request Example
```
curl -i https://your-api.example.com/profile-availability-webhook \
    -X PATCH \
    -H 'Authorization: ApiKeyProvidedByRestia' \
    -H 'Content-Type: application/json' \
    -d '{"id":"69314b087fdd7f45d9c8c3faaa5084ce","status":"CLOSED","until":"2020-11-30T10:10:00.000Z"}'
```

Open profile request
---------------------
- Request contains body encoded as JSON string
- On success expect response HTTP 200

### Body
Field|Type|Required|Description|Example value|
|---            |---                |---|---|---|
|id             |string             | Y | [Restaurant ID](#restaurant-id), provided by RESTIA | 69314b087fdd7f45d9c8c3faaa5084ce  |
|status         |string             | Y | New status of profile | OPEN |

### Body json example
```
{
    "id": "69314b087fdd7f45d9c8c3faaa5084ce", 
    "status: "OPEN"
}
```

### Request Example
```
curl -i https://your-api.example.com/profile-availability-webhook \
    -X PATCH \
    -H 'Authorization: ApiKeyProvidedByRestia' \
    -H 'Content-Type: application/json' \
    -d '{"id":"69314b087fdd7f45d9c8c3faaa5084ce","status":"OPEN"}'
```
