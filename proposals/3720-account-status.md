# MSC3720: Account status endpoint

Matrix clients sometimes need a way to display additional information about a
user. For example, when interacting with a Matrix user, it might be useful for
clients to show whether this user's account has been deactivated, or even exists
at all.

Currently clients are forced to resort to hacks to try to derive this
information. Some, for example, check whether a user has a profile (display
name, avatar) set when trying to send an invite, which means that if a user
doesn't have any profile information (which is valid under the Matrix
specification), the inviter might be warned that the user might not exist.

## Proposal

Two new endpoints are added to the specification, one to the client-server API
and one to the server-server API.

### `GET /_matrix/client/v1/account_status`

This endpoint requires authentication via an access token.

This endpoint takes a `user_id` query parameter indicating which user account(s)
to look up the status of. This parameter may appear multiple times in the
request if the client wishes to look up the statuses of multiple users at once.

If no error arises, the endpoint responds with a body using the following
format:

```json
{
    "account_statuses": {
        "@user1:example.com": {
            "exists": true,
            "deactivated": false
        },
        "@user2:example.com": {
            "exists": false
        },
    },
    "failures": [
        "@user3:example.com",
        "@user4:otherexample.com"
    ]
}
```

The `account_statuses` object in the response lists all statuses that could
successfully be retrieved. Each key in this object maps to one of the `user_id`
parameter(s) in the request. For each account:

* `exists` is a boolean that indicates whether an account exists with this user
  ID.
* `deactivated` is a boolean that indicates whether the account has been
  deactivated. Omitted if `exists` is `false`.

The `failures` object in the response lists all user IDs for which no status
could be retrieved for any reason (e.g. federation issues, missing federation
endpoint, missing user in the remote server's response, etc).

The combination of the lists of user IDs from `user_statuses` and `failures`
must match the full list of user IDs provided in the request via `user_id`
parameters.

If one or more account(s) is not local to the homeserver this request is
performed on, the homeserver must attempt to retrieve account status using the
federation endpoint described below.

Below is how this endpoint behaves in case of errors:

* If no `user_id` parameter is provided, the endpoint responds with a 400 status
  code and a `M_MISSING_PARAM` error code.
* If one or more of the `user_id` parameter(s) provided cannot be parsed as a
  valid Matrix user ID, the endpoint responds with a 400 status code and a
  `M_INVALID_PARAM` error code.

### `GET /_matrix/federation/v1/query/account_status`

This endpoint behaves in an identical way to the client-side endpoint described
above, with the additional following error case:

* If one or more of the `user_id` parameter(s) does not match a local account,
  the endpoint responds with a 400 status code and a `M_INVALID_PARAM` error
  code.

### `m.account_status` capability

Some server administrators might not want to disclose too much information about
their users. To support this use case, homeservers must expose a
`m.account_status` capability to tell clients whether they support retrieving
account status via the client-side endpoint described above.

## Security considerations

Should a server administrator not want to disclose information about their users
through the federation endpoint described above, they should use a reverse proxy
or similar tool to prevent access to the endpoint. On top of this, homeserver
implementations may implement measures to only respond with an empty JSON object
`{}` in this case.

## Unstable prefixes

Until this proposal is stabilised in a new version of the Matrix specification,
implementations should use the following paths for the endpoints described in
this document:

| Stable path                                   | Unstable path                                                          |
|-----------------------------------------------|------------------------------------------------------------------------|
| `/_matrix/client/v1/account_status`           | `/_matrix/client/unstable/org.matrix.msc3720/account_status`           |
| `/_matrix/federation/v1/query/account_status` | `/_matrix/federation/unstable/org.matrix.msc3720/query/account_status` |

Additionally, implementations should use the unstable identifier
`org.matrix.msc3720.account_status` instead of `m.account_status` for the
client-side capability.