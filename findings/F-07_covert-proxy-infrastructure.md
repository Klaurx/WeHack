# F-07: Covert Proxy Infrastructure with Root Credentials in Repository

## What was observed

A directory named `server-template/v2ray/` exists in the 试玩测试 and 豆豆赚 workspaces, committed to [REDACTED_GITEE_REPO_2]. It contains two files: a V2Ray VMess proxy server configuration and a documentation file.

The V2Ray configuration defines an inbound VMess listener on port 55555 and a freedom outbound, deployed via Docker:

```
docker run -d -v /etc/v2ray:/etc/v2ray -p 55555:55555 v2ray/official v2ray -config=/etc/v2ray/config.json
```

The documentation file accompanying the configuration contains:

- The IP address of the server running this proxy: [REDACTED_PROXY_IP]
- The root username and password for that server: root / [REDACTED_PROXY_ROOT_PASSWORD]
- A proxy chain mapping: [REDACTED_PROXY_IP]:50001 proxies to [REDACTED_DOWNSTREAM_IP]:50013
- The VMess client UUID used for authentication: [REDACTED_VMESS_UUID]

The V2Ray server appears to be a separate cloud instance unrelated to the main application infrastructure, provisioned specifically to run this proxy.

## Why this matters

This is not a misconfigured application setting. This is operational proxy infrastructure with its root credentials stored in a source repository. Anyone with read access to the repository has had root access to this server.

V2Ray with the VMess protocol is commonly used to bypass network filtering or to create covert tunnels. Its presence in an application repository, combined with the proxy chain configuration pointing to a second downstream server, indicates this infrastructure exists outside of any standard deployment or monitoring process.

The root password for [REDACTED_PROXY_IP] must be treated as fully compromised.

## What needs to change

- Rotate the root password for [REDACTED_PROXY_IP] immediately.
- Rotate the VMess UUID.
- Audit the purpose of this proxy infrastructure and determine whether it is authorized.
- Audit what traffic has passed through this proxy using whatever logging exists on [REDACTED_PROXY_IP].
- Remove all files from this directory from the repository and rewrite git history.
- Evaluate whether [REDACTED_DOWNSTREAM_IP] is part of the production infrastructure and audit it accordingly.
