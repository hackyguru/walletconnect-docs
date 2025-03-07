# Data Structures

In this document we define data structures and definitions used in the v2.0-rc protocol

## Relay

Relay is defined by the transport protocol used for the two clients to publish and subscribe messages between each other.

```jsonc
{
  "protocol": string,
  "data": string, // optional
}
```

## Metadata

Metadata is a set of parameters used to identify each participant in a session and/or pairing which are provided by the consumer for the client to broadcast to its peer

```jsonc
{
  "name": string,
  "description": string,
  "url": string,
  "icons": [string],
  "redirect": { // Optional
    "native": string, // Optional
    "universal": string, // Optional
  }
}
```

## Pairing

Pairing is a topic encrypted by a symmetric key shared through a URI between two clients with the sole purpose of communicating session proposals

```jsonc
{
  "topic": string,
  "relay": {
    "protocol": string,
    "data": string
  },
  "peerMetadata": Metadata,
  "expiry": Int64,
  "active": boolean
}
```

## Session

Session is a topic encrypted by a symmetric key derived using a key agreement established after an approved proposal and it has a controller pariticipant that can update its accounts, methods, events and expiry

```jsonc
{
  "topic": string,
  "relay": {
    "protocol": string,
    "data": string
  },
  "self": {
    "publicKey": string,
    "metadata": Metadata
  },

  "peer": {
    "publicKey": string,
    "metadata": Metadata
  },

  "expiry": Int64, // timestamp (seconds)
  "acknowledged": boolean,
  "controller": string,
  "namespaces": {
    "<namespace_name>" : {
      "accounts": [string],
      "methods": [string],
      "events": [string],
      "extension": [ // optional
        {
          "accounts": [string],
          "methods": [string],
          "events": [string],
        }
      ]
    }
  },
  "requiredNamespaces": {
    "<namespace_name>" : {
      "chains": [string],
      "methods": [string],
      "events": [string],
      "extension": [ // optional
        {
          "chains": [string],
          "methods": [string],
          "events": [string],
        }
      ]
    }
  },
}
```

## Proposal

Proposal is sent by the proposer client to be approved or rejected by the responder who is assumed to be the controller of the resulting session and will respond with its public key to derive the future topic and symKey.

```jsonc
{
  "id": number, // json-rpc request id
  "relays": [
    {
      "protocol": string,
      "data": string
    }
  ],
  "proposer": {
    "publicKey": string,
    "metadata": Metadata
  },
  "requiredNamespaces": {
    "<namespace_name>" : {
      "chains": [string],
      "methods": [string],
      "events": [string],
      "extension": [ // optional
        {
          "chains": [string],
          "methods": [string],
          "events": [string],
        }
      ]
    }
  },
  "pairingTopic": string
}
```

## Response

Response is sent by the responder client and can either be an approval or rejection to the proposal made and if approved will be followed by a settlement request on the new topic derived from the hashed symmetric key from the key agreement

```jsonc
// Approval
{
  "relay": {
    "protocol": string,
    "data": string // Optional
  },
  "responderPublicKey": string
}

// Rejection

{
  "error": {
    "code": Int64,
    "message": string
  }
}
```

## Settlement

Settelement is sent by the responder after approval and it's broadcasted right after the response on the new topic derived from the hashed symmetric key from the key agreement

```jsonc
{
  "relay": {
    "protocol": string,
    "data": string // Optional
  },
  "controller": {
    "publicKey": string,
    "metadata": Metadata
  },
  "namespaces": {
    "<namespace_name>" : {
      "accounts": [string],
      "methods": [string],
      "events": [string],
      "extension": [ // optional
        {
          "accounts": [string],
          "methods": [string],
          "events": [string],
        }
      ]
    }
  },
  "expiry": Int64, // seconds
}
```
