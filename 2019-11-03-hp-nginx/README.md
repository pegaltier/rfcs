# HP Nginx

## The purpose

HoloPort is running multiple services that need to be available to end user. Traffic is directed to the HoloPort by holo-router over the ZeroTier network. HoloPort needs to have a 443 inbound port open on the ZeroTier network interface. On that port (443) HP-nginx is listening.

## TLS

Nginx terminates https traffic. TLS Certificate installation is handled by nixOS and Let's Encrypt SSL service.

## Routes

Nginx needs to handle following connections:
| Route | Authorization | Type | Destination | Purpose |
| ----- | ------------- | ---- | ----------- | ------- |
| `/`   | [`rate-limit`](#rate-limit) | http | HP Admin static files folder | Serve static files of HP Admin |
| `/holfuel/` | [`rate-limit`](#rate-limit) | http | Holofuel static files folder | Serve static files of HoloFuel |
| `/api/v1/` | [`X-Holo-Admin`](#X-Holo-Admin) | http | port xxx | Redirect to HP Admin Server |
| `/api/v1/ws/` | [`X-Holo-Admin`](#X-Holo-Admin) | ws | port xxx | Redirect to conductor running HoloFuel and HoloHosting app |

Return `404 Not Found` otherwise.

## Authorization

Depending on the service there are various ways for authorizing traffic.

### rate-limit

Static assets server will have to have rate limit on number of requests per one IP.
```
limit_req_zone $binary_remote_addr zone=zone1:1m rate=2r/s;

location / {
    limit_req zone=zone1 burst=30;
}
```

### X-Holo-Admin
HP Admin related calls have to be secured with the Signature-based authorization. It needs to be calculated in HP-nginx because Holochain Conductor, which receives websocket traffic, does not have capability for authorization check. 

Authorization schema is `X-Holo-Admin-Signature` HTTP header, followed by Base64-encoded Ed25519 signature. Payload signed is ???

Example: `X-Holo-Admin-Signature: EGeYSAmjxp1kNBzXAR2kv7m3BNxyREZnVwSfh3FX7Ew`


