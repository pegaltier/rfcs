# HP Admin HTTP API

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
    "holoportos": {
        "network": "live",
        "sshAccess": true
    },
    "name": "My HoloPort"
}
```

#### `401 Unauthorized`

### `POST /v1/config`

Updates `holo-config.json`.

```json
{
    "admin": {
        "name": "Holo Naut",
        "public_key": "z4NA8s70Wyaa2kckSQ3S3V3eIi8yLPFFdad9L0CY3iw"
    },
    "holoportos": {
        "sshAccess": false
    }
}
```

#### `200 OK`

```json
{
    "admin": {
        "email": "sam.rose@holo.host",
        "name": "Holo Naut",
        "public_key": "z4NA8s70Wyaa2kckSQ3S3V3eIi8yLPFFdad9L0CY3iw"
    },
    "holoportos": {
        "network": "live",
        "sshAccess": false
    },
    "name": "My HoloPort"
}
```

#### `400 Bad Request`
#### `401 Unauthorized`

### `GET /v1/status`

#### `200 OK`

Prints immutable HoloPort status data. `zerotier` field is verbatim `zerotier-cli -j info` output.

```json
{
    "holo_nixpkgs": {
      "url": "https://hydra.holo.host/channel/custom/holo-nixpkgs/develop/holo-nixpkgs",
      "rev": "b13891c28d78f1e916fdefb5edc1d386e4f533c8",
    },
    "zerotier": {
        "address": "2f07044b7a",  
        "clock": 1571075895334, 
        "config": { 
            "physical": null, 
            "settings": { 
               "allowTcpFallbackRelay": true, 
                "portMappingEnabled": true, 
                "primaryPort": 9993, 
                "softwareUpdate": "disable", 
                "softwareUpdateChannel": "release" 
            }
        }, 
        "online": true, 
        "planetWorldId": 149604618, 
        "planetWorldTimestamp": 1567801551272, 
        "publicIdentity": "2f07044b7a:0:505688f5c97313e5c7e34547e49a6ac46a05746b2e3faad724103b8ed34a4b108e15d08051db09eedd53ed089b19a5bfae9b1afdb7a9c65ad6f8aa9d98e4f2f2", 
        "tcpFallbackActive": false, 
        "version": "1.2.12", 
        "versionBuild": 0, 
        "versionMajor": 1, 
        "versionMinor": 2, 
        "versionRev": 12
    }
}
