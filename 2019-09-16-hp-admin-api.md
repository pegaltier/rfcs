# HP Admin API Document Draft

## Features not covered 

### Ports

Since we are tunneling to the machine, it is recommended to deprecate the
requirement for changing ports. The user should not have any need to change
ports, and updating from hp admin will also require updating DNS settings each
time ports are changed. Recommended to remove this feature unless we serve
these applications on local network.

### Factory reset

It is impossible to create the feature that is implied in the wireframe,
because it would require hardware that supports accessing the boatloader in a
secure manner. Our hardware does not support this currently.

Instead, we recommend offering the user instructions to download a
rescue/restore image, and follow instructions to reset their machine by copying
this file to the USB and then insert into the holoport, and restart the
machine.

## Authorization

Authorization schema is `X-Holo-Admin-Signature` HTTP header, followed by
Base64-encoded Ed25519 signature.

Example: `X-Holo-Admin-Signature: EGeYSAmjxp1kNBzXAR2kv7m3BNxyREZnVwSfh3FX7Ew`

## Endpoints

### `GET /v1/config`

#### `200 OK`

Returns `holo-config.json` data with `seed` field filtered out.

```json
{
    "admin": {
        "email": "sam.rose@holo.host",
        "public_key": "Tw7179WYi/zSRLRSb6DWgZf4dhw5+b0ACdlvAw3WYH8"
    },
    "ssh_access": true
}
```

#### `401 Unauthorized`

### `POST /v1/config`

Updates `holo-config.json`.

```json
{
    "admin": {
        "public_key": "z4NA8s70Wyaa2kckSQ3S3V3eIi8yLPFFdad9L0CY3iw"
    },
    "ssh_access": false
}
```

#### `200 OK`
#### `400 Bad Request`
#### `401 Unauthorized`

### `GET /v1/status`

#### `200 OK`

Prints immutable HoloPort status data.

```json
{
    "holo_nixpkgs": {
      "url": "https://hydra.holo.host/channel/custom/holo-nixpkgs/develop/holo-nixpkgs",
      "rev": "b13891c28d78f1e916fdefb5edc1d386e4f533c8",
    },
    "zerotier": {
        // `zerotier-cli -j info` output
    }
}
