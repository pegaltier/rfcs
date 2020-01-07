
# Holo Envoy

## Data Formats

### `Signature`
base64 encoded ed25519 signature

Example `lWstadoXkI1oMuwMeiGJbd+D7U7NblsGBU/iZTRUWUuO06eLWYAoNDYzeZKUFTDOIsMB2aXodsOr176odPrrAQ==`

### `Hash`
Multihash encoded sha2-256 digest

Example `QmUgZ8e6xE1h9fH89CNqAXFQkkKyRh2Ag6jgTNC8wcoNYS`

### `AgentID`
HCID encoded public key

Example `HcSCjUNP6TtxqfdmgeIm3gqhVn7UhvidaAVjyDvNn6km5o3qkJqk9P8nkC9j78i`

## Data Structs

### `ConfirmationPayload`
```javascript
{
    "response_hash"        : string,
    "client_metrics"       : object,
}
```

### `ServiceRequest`
```javascript
{
    "anonymous"            : boolean,
    "agent_id"             : string,
    "payload"              : ServicePayload,
    "service_signature"    : string
}
```

### `ServicePayload`
```javascript
{
    "timestamp"            : string,
    "host_id"              : string,
    "call_spec": {
        "instance_id"      : string,
        "zome"             : string,
        "function"         : string,
        "args"             : array<any>,
        "hha_hash"         : string,
        "dna_alias"        : string
    }
}
```


## RPC WebSocket API

*Connecting to HPOS*
```javascript
const RPCWebSocket = require('rpc-websockets').Client;

const example_host_domain = "5m5srup6m3b2iilrsqmxu6ydp8p8hr0rdbh4wamupk3s4sxqr5.holohost.net";
const client = new WebSocket( `wss://${example_host_domain}` );
```


### Handling Errors

There are 2 error scenarios

- `HoloError` - *Holo infrastructure failed*
- `Error` - *Any errors not caused by Holo*

Hosts do not record service logs when a `HoloError` occurs, because the hApp providers are not
responsible for the failure.

---
### `holo/agent/signup`
Request Envoy to set up new hApp instances for this Agent.  We should have a way to verify that this
Agent is actually unregisterd for the given hApp.

**Arguments**
- `<hha_hash> : Hash`
- `<agent_id> : AgentID`

**Example**
```javascript
let resp = await client.call("holo/agent/signup", [
    "QmUgZ8e6xE1h9fH89CNqAXFQkkKyRh2Ag6jgTNC8wcoNYS",
    "HcSCjUNP6TtxqfdmgeIm3gqhVn7UhvidaAVjyDvNn6km5o3qkJqk9P8nkC9j78i"
]);
```

#### Possible responses
Ok
- Success
  ```javascript
  true
  ```
  
Err
- Generic failure
  ```javascript
  {
      "name": "HoloError",
      "message": "Failed to create a new hosted agent",
  }
  ```
- Service logger failed
  ```javascript
  {
      "error": {
          "name": "HoloError",
          "message": "servicelogger.log_service failed: ${service_log.Err}",
      }
  }
  ```


---
### `holo/agent/identify`
A simple maintenance endpoint for RPC WebSockets.  It creates a unique event for sending messages
directly to the Agent.  Returns the event name so the client can subscribe to it.

**Arguments**
- `<agent_id> : AgentID`

**Example**
```javascript
let resp = await client.call("holo/agent/identify", [
    "HcSCjUNP6TtxqfdmgeIm3gqhVn7UhvidaAVjyDvNn6km5o3qkJqk9P8nkC9j78i"
]);
```

#### Possible responses
Ok
- Success
  ```javascript
  "HcSCjUNP6TtxqfdmgeIm3gqhVn7UhvidaAVjyDvNn6km5o3qkJqk9P8nkC9j78i/wormhole/request"
  ```
  
Err
- Agent unknown
  ```javascript
  {
      "name": "HoloError",
      "message": "Agent 'HcSCjUNP6TtxqfdmgeIm3gqhVn7UhvidaAVjyDvNn6km5o3qkJqk9P8nkC9j78i' is unknown to this Host",
  }
  ```
- Call conductor threw error
  ```javascript
  {
      "name": "HoloError",
      "message": String(err),
  }
  ```


---
### `holo/call`
Call a zome function.

**Arguments**
- `<request> : ServiceRequest`

**Example**
```javascript
let resp = await client.call("holo/call", {
    "anonymous": false,
    "agent_id": "HcSCjUNP6TtxqfdmgeIm3gqhVn7UhvidaAVjyDvNn6km5o3qkJqk9P8nkC9j78i",
    "payload": {
	"timestamp": 1578416083200,
	"host_id": "localhost",
	"call_spec": {
	    "instance_id": "QmUgZ8e6xE1h9fH89CNqAXFQkkKyRh2Ag6jgTNC8wcoNYS::HcSCjUNP6TtxqfdmgeIm3gqhVn7UhvidaAVjyDvNn6km5o3qkJqk9P8nkC9j78i-dna_alias",
	    "zome": "zome_name",
	    "function": "func_name",
	    "args": {},
	    "hha_hash": "QmUgZ8e6xE1h9fH89CNqAXFQkkKyRh2Ag6jgTNC8wcoNYS",
	    "dna_alias": "dna_alias"
	}
    },
    "service_signature": "q/IGm5vmsAD6Eh/rPnsfbxxYXWb1h20xhEZ2ASAJjQcK+LxqRO4FPJWV4BxJB/ydRzCmpaQvrMBopbCbYyz0Aw=="
});
```

#### Possible responses
Ok
- Success
  ```javascript
  {
      "response_id": "Qmaa779KVD5z1LkzSccCMjJmYtd11DXA4gtTgsNFo8365G",
      "result": ...zome call result,
  }
  ```
  
Err
- Service logger request failed
  ```javascript
  {
      "error": {
          "name": "HoloError",
          "message": "servicelogger.log_request failed: ${req_log.Err}",
      }
  }
  ```
- Failed signing
  ```javascript
  {
      "error": {
          "name": "HoloError",
          "message": "We were unable to contact Chaperone for the Agent signing service.  Please check ...",
      },
      "response_id": "QmSvPd3sHK7iWgZuW47fyLy4CaZQe2DwxvRhrJ39VpBVMK",
      "result": {},
  }
  ```
- Not signed-in
  ```javascript
  {
      "error": {
          "name": "HoloError",
          "message": "Agent is not signed-in",
      },
      "response_id": "QmeyYcMz1vFuSKciNAJXKgniv89Wt1JqHGWko9bpR6rWsk",
      "result": {},
  }
  ```
- Service logger response failed
  ```javascript
  {
      "error": {
          "name": "HoloError",
          "message": "servicelogger.log_response failed: ${res_log.Err}",
      }
  }
  ```


---
### `<agent_id>/wormhole/request`
Each signed-in Agent has a unique endpoint so that the server can send signing requests.

**Arguments**
- `<id>    : number`
- `<entry> : object`

**Example**
```javascript
client.on("holo/wormhole/response", async ( id, data ) => {
    // sign data and call 'holo/wormhole/response'
});
```


---
### `holo/wormhole/response`
The client sends signatures for signed entries to this endpoint.

**Arguments**
- `<id>        : number`
- `<signature> : Signature`

**Example**
```javascript
let resp = await client.call("holo/wormhole/response", [
    5421, // Wormhole request ID
    "TotZt8AGH3IxOJTYToT2sGePdy2J2rzGgF2XP4sTC3ruOQp3ac2W0XikXj+CrGkso/JuNEWxKO0Q/K7G1ThVAg=="
]);
```

#### Possible responses
Ok
- Success
  ```javascript
  true
  ```
  
Err
- ID does not exist
  ```javascript
  {
      "name": "HoloError",
      "message": "ID 5421 does not match any pending entries",
  }
  ```


---
### `holo/service/confirm`
The client calls this to confirm that it received a response.

> **NOTE:** We will have to limit the number of responses that can be unconfirmed before a Host
> stops serving that Agent.  Otherwise, a modified Chaperone would be able to get services without
> paying.

**Arguments**
- `<response_id>  : Hash`
- `<confirmation> : ConfirmationPayload`
- `<signature>    : Signature`

**Example**
```javascript
let resp = await client.call("holo/service/confirm", [
    "Qmaa779KVD5z1LkzSccCMjJmYtd11DXA4gtTgsNFo8365G",
    {
        "response_hash": "QmV7T6o7dkUbBrVpHAyY3eE6xtd3VjdbM6PAgs9j5MhkC6",
        "client_metrics": {},
    },
    "lWstadoXkI1oMuwMeiGJbd+D7U7NblsGBU/iZTRUWUuO06eLWYAoNDYzeZKUFTDOIsMB2aXodsOr176odPrrAQ=="
]);
```

#### Possible responses
Ok
- Success
  ```javascript
  true
  ```
  
Err
- Service logger confirm failed
  ```javascript
  {
      "error": {
          "name": "HoloError",
          "message": "servicelogger.log_service failed: ${service_log.Err}",
      }
  }
  ```
