# The NoCAN REST API

This **draft** describes the NOCAN REST API as implemented when invoking the
command `nocanc webui`.

## GET /api/v1/channels

Expected response code: 200 Success

### Example

#### Query

```
GET /api/v1/channels
```

#### Response

```json
{
  "channels": [
    {
      "id": 1,
      "name": "pir",
      "status": "updated",
      "value": "0"
    },
    {
      "id": 2,
      "name": "bme280/humidity",
      "status": "updated",
      "value": "48.62"
    },
    {
      "id": 3,
      "name": "bme280/pressure",
      "status": "updated",
      "value": "96932.36"
    },
    {
      "id": 4,
      "name": "bme280/altitude",
      "status": "updated",
      "value": "372.31"
    },
    {
      "id": 0,
      "name": "bme280/temperature",
      "status": "updated",
      "value": "23.55"
    }
  ]
}
```

## GET /api/v1/channels/:id

### Description

Expected response code: 200 Success

### Example

#### Query

```
GET /api/v1/channels/1
```

#### Response

```json
{
  "id": 1,
  "name": "pir",
  "status": "updated",
  "value": "0"
}
```

## GET /api/v1/nodes

### Description

Expected response code: 200 Success

### Example

```
GET /api/v1/nodes
```

```json
{
  "nodes": [
    {
      "id": 3,
      "state": "connected",
      "udid": "2b:4c:d2:e6:2f:6c:ae:44",
      "last_seen": "2019-06-03T15:09:25.210476859Z"
    },
    {
      "id": 4,
      "state": "connected",
      "udid": "39:0c:65:d3:9d:c0:40:d3",
      "last_seen": "2019-06-03T15:01:23.983354961Z"
    }
  ]
}
```

## GET /api/v1/nodes/:id

### Description

Expected status code: 200 Success

### Example

#### Query

```
GET /api/v1/nodes/3
```

#### Response

```json
{
  "id": 3,
  "state": "connected",
  "udid": "2b:4c:d2:e6:2f:6c:ae:44",
  "last_seen": "2019-06-03T15:10:25.285819308Z"
}
```

## PUT /api/v1/nodes/:id/reboot

### Description

Expected response code: 204 No Content

### Example

#### Query

```
PUT /api/v1/nodes/3
```

#### response

Empty.

## GET /api/v1/power_status

### Description

Expected response code: 200 Success

### Example

#### Query

```
GET /api/v1/power_status
```

#### Response

```json
{
  "status": "powered",
  "voltage": 11.940439,
  "current_sense": 20,
  "reference_voltage": 3.3672585
}
```
