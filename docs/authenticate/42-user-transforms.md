---
sidebar_position: 42
---

# User Transforms

A user transform allows performing the following once a user has passed
authentication:

* add/remove user roles
* add link to UI portal page
* require multi-factor authentication (MFA/2FA)
* require accepting term and conditions
* block/deny access (by not issuing a token)
* injecting custom, arbitrary claims into a token


## Add Roles

The following transform matches `sub` field and grants `authp/viewer` role:

```
  transform user {
    exact match sub github.com/greenpau
    action add role authp/viewer
  }
```

The following transform adds role `verified` to Facebook-authenticated user
with id of `123456789`:

```
  transform user {
    exact match sub 123456789
    exact match origin facebook
    action add role verified
  }
```

The following transform adds the role `contoso_users` to users with an email
address from the contoso.com domain:

```
  transform user {
    suffix match email @contoso.com
    add role contoso_users
  }
```

## Add UI Links

The following transform, in addition to the above adds a link to a user's
portal page:

```
  transform user {
    exact match sub github.com/greenpau
    action add role authp/viewer
    ui link "Caddy Version" /version icon "las la-code-branch"
  }
```

## Force Multi-Factor Authentication

The following transform requires to pass multi-factor authentication when the
authenticated user's email is `webadmin@localdomain.local`:

```
  transform user {
    match email webadmin@localdomain.local
    require mfa
  }
```

## Deny Access

The following transform blocks a user with email `anonymous@badactor.com`
from getting authenticated:

```
  transform user {
    match email anonymous@badactor.com
    block
  }
```

## Inject Custom Claims

The syntax:

```
action add <claim_name> <claim_value> as <string|list>
action add nested [arg1 ... argN] with [arg1 ... argN] as <string|list>
```

Here, if a user is authentication in `local` realm and has email address
of `webadmin@localdomain.local`, then a number of custom claims will be
added to the token issued by the portal.

```
  transform user {
    match realm local
    match email webadmin@localdomain.local
    action add foo bar as string
    action add nested "acl" "paths" "/*/users/**" as map
    action add nested "acl" "paths" "/*/conversations/**" as map
    action add nested "acl" "paths" "/*/sessions/**" as map
    action add nested "acl" "paths" "/*/devices/**" as map
    action add nested "metadata" "language" with "english" as string
    action add nested "metadata" "interests" with "movies" "gaming" as list
  }
```

The outcome follows:

```json
{
  "acl": {
    "paths": {
      "/*/conversations/**": {},
      "/*/devices/**": {},
      "/*/sessions/**": {},
      "/*/users/**": {}
    }
  },
  "metadata": {
    "interests": [
      "movies",
      "gaming"
    ],
    "language": "english"
  },
  "foo": "bar"
}
```

Alternatively, one can add `name` and `picture` (avatar) claims:

```
  transform user {
    match realm local
    match email webadmin@localdomain.local
    action add name "Paul Greenberg"
    action add picture "https://avatars.githubusercontent.com/u/3826416?v=4" as string
  }
```
