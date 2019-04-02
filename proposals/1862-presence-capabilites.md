# Presence flag for capabilities API

## Proposal

Some homeservers choose to disable the presence feature, so that users cannot send or receive presence.
This should be specificed in the new capabilities API. Servers can also choose to restrict some users
while allowing others, and the capabilities response should reflect this (since the endpoint is authed).

This proposal makes it optional to support presence in the spec, while this has been the reality in the 
Matrix network for some time now.

## Rationale

While it would be good if every server enabled presence today, this is not the case. Some servers have opted to
disable presence to reduce the strain on their servers (notably matrix.org and other larger community servers).
This proposal allows these servers to advertise to clients connected over C2S that presence is not enabled.

## Solution

`GET /_matrix/client/r0/capabilities`

```javascript
{
  "capabilities": {
    "m.presence": {
      "send_enabled": false,
      "receive_enabled": false,
    },
    // ...
}
```

An omission of `m.presence` would default both sending and receiving to true. An omission of either child flag
would also default that flag to true.

When `send_enabled` is `false`, homeservers should respond to requests to  `PUT /_matrix/client/r0/presence/{userId}/status` with a status of `200`,
but with a body of  `{"send_enabled": false}`. This is to ensure that this spec is backwards compatible with
existing clients who may explode on a non-200 response,  but will still report if it actually sent the presence.
It's unfortunate that this isn't symmetrical with the `GET` request below.

When `receive_enabled`  is `false`, homeservers should respond to requests to 
`GET /_matrix/client/r0/presence/{userId}/status` with `M_FORBIDDEN`.

The `/sync` format does not change based on whether this is enabled. `presence.events` MUST be empty when
`receive_enabled` is `false`. The query parameter `set_presence` is ignored and no presence information
is changed. Clients SHOULD ensure that they have checked the capailities API before assuming that the parameter
will work.

## Tradeoffs

This proposal makes no attempt to address disabled presence over S2S, so remote homeservers can still send EDUs
containing presence to a presence-disabled server and the expectation is that the transactions are accepted,
but any presence events contained within are not processed.

## Potential issues

None that I can think of.

## Security considerations

None that I can think of.