# HP Dispatcher

## The purpose

HoloPort is running multiple services that need to be available to end user. Traffic is directed to the HoloPort by holo-router over the ZeroTier network. HoloPort needs to have a 443 inbound port open on the ZeroTier network interface. On that port (443) HP-dispatcher (eg. implemented via nginx) is listening.

## TLS

HP-dispatcher terminates https traffic. TLS Certificate installation is handled by nixOS and Let's Encrypt SSL service.

## Routes

HP-dispatcher needs to handle following connections:

| Route         | Authorization                   | Type | Destination                  | Purpose                                      |
| -----         | -------------                   | ---- | -----------                  | -------                                      |
| `/`           | [`rate-limit`](#rate-limit)     | http | 302 redirect to              | Redirect to `/admin/`                        |
| `/admin/`     | [`rate-limit`](#rate-limit)     | http | HP Admin static files folder | Serve static files of HP Admin               |
| `/holofuel/`  | [`rate-limit`](#rate-limit)     | http | HoloFuel static files folder | Serve static files of HoloFuel               |
| `/api/v1/`    | [`X-Holo-Admin`](#X-Holo-Admin) | http | port 23646                   | Proxy to HP Admin Server                     |
| `/api/v1/ws/` | [`X-Holo-Admin`](#X-Holo-Admin) | ws   | port 42233                   | Proxy to Conductor Admin Instances           |
| `/hosting/`   | none*                           | ws   | port 4656                    | Proxy to Envoy service handling Holo-hosting |

Return `404 Not Found` otherwise.

## Authentication / Authorization

Depending on the service there are various ways for authorizing traffic.

### rate-limit

Static assets server will have to have rate limit on number of requests per one IP.
```
limit_req_zone $binary_remote_addr zone=zone1:1m rate=2r/s;

location / {
    limit_req zone=zone1 burst=30;
}
```
This is a measure against bandwidth depletion due to unwanted serving of static files from HoloPort, as the are meant to be served to HoloPort Admin alone.

### X-Holo-Admin
HP Admin related calls have to be secured with the Signature-based authorization. It needs to be handled by HP-dispatcher because Holochain Conductor, which receives websocket traffic, does not have capability for authorization check. 

Authorization schema is `X-Holo-Admin-Signature` HTTP header, followed by Base64-encoded Ed25519 signature of the following payload:
```
{
  "method": ${verb}, // eg. "GET"
  "request": ${endpoint}, // eg "/api/v1/config"
  "body": ${body} // eg. "{\"name\": \"My HoloPort Name\"}"
}
```

Example: `X-Holo-Admin-Signature: EGeYSAmjxp1kNBzXAR2kv7m3BNxyREZnVwSfh3FX7Ew`

### none for hosting

Authorization for hosting related traffic is still an open question. Can any user open a websocket connection with Conductor? How can Conductor handle concurrency in requests?

## Transport Layer Reservations

The `name` in `<ip>:<port> "<name>"` corrosponds to the port number in that it matches the alpha/numeric mapping
of English telephone keypads (eg. https://phonespell.org).

### `0.0.0.0:443` Reverse Proxy

**Supported protocols**
- HTTPS
- WebSocket Secure

### `localhost:4656 "holo"` Envoy

**Supported protocols**
- HTTP
- WebSocket

### `localhost:23646 "admin"` Admin Server

**Supported protocols**
- HTTP

### `localhost:2884 "auth"` Authentication Service

**Supported protocols**
- HTTP

### `localhost:422[11,22,33,44] "hcc##"` Conductor

**Supported protocols**
- WebSocket

#### `localhost:42211` Admin
The only Conductor interface with Administrative privileges

#### `localhost:42222` Internal
Used for Service Logger

#### `localhost:42233` Admin Instances
Access to the Admin's installed hApp instances

#### `localhost:42244` Hosted Instances
Access to the hosting enabled hApp instances
