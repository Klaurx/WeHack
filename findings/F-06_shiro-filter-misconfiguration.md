# F-06: Shiro Filter Chain Misconfiguration

## What was observed

The Shiro security filter chain is defined in `fastcpl-web-backstage/src/main/resources/security/applicationContext-shiro.xml`. The following paths are configured as `anon`, meaning they require no authentication to access:

```
/waimai/api/**
/m/**
/openclaw/**
/chat/**
/cpl/**
/cps/m/**
/activity/litegame/**
/activity/**/**
/article/**
/file/**
/qudao/**
/static/**
/openinstall/**
/pay/**
/api/**
/jzapi/**
/alipay/**
/login
/thirdpay/**
/channel/**
/css/**
/images/**
/mp3/**
/js/**
/logout
/account/**
/merchant/**/**
```

Three of these rules contain a syntax error. The following paths use `==anon` (double equals) instead of `=anon`:

```
/pay/**==anon
/api/**==anon
/jzapi/**==anon
```

In Shiro's filter chain definition parser, `==anon` is not a valid filter definition. Depending on the Shiro version, this either causes the rule to be silently ignored (meaning those paths fall through to `/**=authc` and are protected), or causes a parse error that results in the filter chain failing to initialize correctly.

The correct behavior in this specific version should be verified. If the rules are being ignored, `/pay/**`, `/api/**`, and `/jzapi/**` are protected. If the filter chain initialization is affected, the behavior of all rules becomes unpredictable.

Beyond the syntax issue, the following intentionally unauthenticated paths represent a significant attack surface:

- `/merchant/**/**` - merchant management endpoints, fully unauthenticated
- `/channel/**` - all channel notification callbacks, no authentication
- `/api/**` - the entire API layer for the mobile application
- `/file/**` - file access endpoints
- `/cpl/**` - CPL-specific endpoints
- `/thirdpay/**` - third-party payment callbacks

## Why this matters

Merchant management being fully unauthenticated means anyone who can reach the server can interact with merchant data without logging in. Channel notification endpoints being unauthenticated is somewhat expected for webhook receivers, but they must have their own signature verification logic. If that verification uses the compromised keys from F-04, it is ineffective.

The `/api/**` path being intentionally unauthenticated means the mobile API has its own session management layer separate from Shiro, implemented in `ApiLoginFilter`. That custom implementation needs to be audited independently.

## What needs to change

- Fix the three `==anon` syntax errors and verify correct parse behavior after fixing.
- Audit every path marked `anon` and confirm that each one has an appropriate alternative security control (signature verification, token validation, rate limiting).
- Specifically audit the merchant management endpoints to determine whether they should be publicly accessible or whether this is an oversight.
- Review the custom `ApiLoginFilter` implementation for session handling vulnerabilities.
