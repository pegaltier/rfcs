# HP Admin API Document Draft

## Features not covered 

### Ports

Since we are tunneling to the machine, it is recommended to deprecate the requirement for changing ports. The user should not have any need to change ports, and updating from hp admin will also require updating DNS settings each time ports are changed. Recommended to remove this feature unless we serve these applications on local network.

### Factory reset

It is impossible to create the feature that is implied in the wireframe, because it would require hardware that supports accessing the boatloader in a secure manner. Our hardware does not support this currently.

Instead, we recommend offering the user instructions to download a rescue/restore image, and follow instructions to reset their machine by copying this file to the USB and then insert into the holoport, and restart the machine.

## Authorization

Authorization schema is `X-Holo-Admin-Signature` HTTP header, followed by Base64-encoded Ed25519 signature.

Example: `X-Holo-Admin-Signature: EGeYSAmjxp1kNBzXAR2kv7m3BNxyREZnVwSfh3FX7Ew`

## Endpoints

### `GET /v1/config`

#### `200 OK`

Layout of this endpoint matches `holo-config.json`, with `seed` field filtered out.

```json
{
    "admins": {
        "email": "sam.rose@holo.host",
        "public_key": "Tw7179WYi/zSRLRSb6DWgZf4dhw5+b0ACdlvAw3WYH8",
    },
    "ssh_access": true
}
```

#### `401 Unauthorized`

### `POST /v1/config`

Updates `holo-config.json`.

```json
{
    "public_key": "Tw7179WYi/zSRLRSb6DWgZf4dhw5+b0ACdlvAw3WYH8",
    "ssh_access": false
}
```

#### `200 OK`
#### `400 Bad Request`
#### `401 Unauthorized`

### `GET /v1/status`

#### `200 OK`

Prints immutable data with HoloPort status data.

```json
{
    "holo_nixpkgs": {
      "url": "https://hydra.holo.host/channel/custom/holo-nixpkgs/develop/holo-nixpkgs",
      "rev": "b13891c28d78f1e916fdefb5edc1d386e4f533c8",
    },
    "holoport_network_id": {
        // `zerotier-cli -j info` output
    }
    "avatar_url": "https://holo.host/wp-content/uploads/Arthur-Brock.jpg", 
    "email": "person@holo.host",
    "name": "Holo Naut",
    "public_key": "Tw7179WYi/zSRLRSb6DWgZf4dhw5+b0ACdlvAw3WYH8",
    "ssh_access": true,
    "holoport_url": "<url>",
    "holoport_name": "<name>", 	
}
