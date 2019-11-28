# HP Dispatcher

## The purpose

HoloPort is running multiple services that need to be available to end user. Traffic is directed to the HoloPort by Holo Router over the ZeroTier network. HoloPort needs to have a 443 inbound port open on the ZeroTier network interface. On that port (443) HP Dispatcher (eg. implemented via nginx) is listening.

## TLS

HP Dispatcher terminates https traffic. TLS Certificate installation is handled by nixOS and Let's Encrypt service.

## Routes

HP Dispatcher needs to handle following connections:

| Route         | Authorization                   | Type | Destination                  | Purpose                                      |
| -----         | -------------                   | ---- | -----------                  | -------                                      |
| `/`           | [`rate-limit`](#rate-limit)     | http | 302 redirect to              | Redirect to `/admin/`                        |
| `/admin/`     | [`rate-limit`](#rate-limit)     | http | HP Admin static files folder | Serve static files of HP Admin               |
| `/holofuel/`  | [`rate-limit`](#rate-limit)     | http | HoloFuel static files folder | Serve static files of HoloFuel               |
| `/api/v1/`    | [`X-Hpos-Admin-Signature`](#X-Hpos-Admin-Signature) | http | Unix domain socket | Proxy to HP Admin Server                     |
| `/api/v1/ws/` | [`X-Hpos-Admin-Signature`](#X-Hpos-Admin-Signature) | ws   | port 42233                   | Proxy to Conductor Admin Instances           |
| `/auth/`   | none - internal | http | port 2884                    | Hpos-Admin-Signature verification service |
| `/v1/hosting/`   | none*                           | ws   | port 4656                    | Proxy to Envoy service handling Holo-hosting |

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

### X-Hpos-Admin-Signature
HP Admin related calls have to be secured with the Signature-based authorization. It needs to be handled by HP Dispatcher because Holochain Conductor, which receives websocket traffic, does not have capability for authorization check.

This can be achieved using auth_request module of nginx:
```
  ...
    auth_request /auth;
  ...

  location /auth {
    internal;
    proxy_pass http://127.0.0.1:2884;
    proxy_set_header X-Original-URI $request_uri;
  }
```
where `/auth` is the service that verifies authorization.

Authorization schema is `X-Hpos-Admin-Signature` HTTP header, followed by Base64-encoded Ed25519 signature of the following payload:
```
{
  "method": ${verb}, // eg. "GET"
  "request": ${URI with arguments}, // eg. "/api/v1/config?a=b"
  "body": ${body} // eg. "{\"name\": \"My HoloPort Name\"}"
}
```

Example: `X-Hpos-Admin-Signature: EGeYSAmjxp1kNBzXAR2kv7m3BNxyREZnVwSfh3FX7Ew`

### none for hosting

Authorization for hosting related traffic is still an open question. Can any user open a websocket connection with Conductor? How can Conductor handle concurrency in requests?

## Transport Layer Reservations

The `name` in `<ip>:<port> "<name>"` corresponds to the port number in that it matches the alpha/numeric mapping
of English telephone keypads (eg. https://phonespell.org).

### `0.0.0.0:443` Reverse Proxy

**Supported protocols**
- HTTPS
- WebSocket Secure

### `localhost:4656 "holo"` Envoy

**Supported protocols**
- HTTP
- WebSocket

### `localhost:9676 "worm"` Wormhole

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
