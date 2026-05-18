# F-02: Unauthenticated Remote Build Trigger

## What was observed

The 试玩测试 build #9845 was initiated by a remote host rather than a user action:

```
Started by remote host [REDACTED_EXTERNAL_IP_1]
```

This means the Jenkins remote build trigger API was used to start the build from an external IP address. No authentication token is visible in the build log, indicating the remote trigger endpoint accepts requests without a valid token or that the token is embedded in a webhook URL that is not rotated.

CPL build #1846 was started by user `lowkey` through the UI. 豆豆赚 #6651 was also started by user `lowkey`. Only 试玩测试 shows the remote trigger pattern.

## Why this matters

If the remote trigger endpoint requires no authentication, any party that knows the Jenkins URL and job name can trigger a build of that job at will. Combined with F-01, this means an external party can initiate a deployment to production by sending a single HTTP request. They do not need credentials, code access, or any prior foothold on the system.

Even if a token is required but embedded in a webhook URL, that URL is effectively a credential and should be treated as compromised given the broader exposure documented in this report.

## What needs to change

- Audit all remote trigger configurations and require authentication tokens for every job.
- Rotate any existing webhook URLs or trigger tokens.
- Restrict the Jenkins remote trigger API to known IP ranges at the network level.
- Review build history for 试玩测试 to identify all builds initiated via remote trigger and determine whether any were unauthorized.
