# F-14: Alipay RSA Certificates Committed to Source Repository

## What was observed

The `server-template/江西/` directory in the 试玩测试 workspace contains three Alipay certificate files:

```
alipayCertPublicKey_RSA2.crt       (1.30 KB)
alipayRootCert.crt                 (5.01 KB)
appCertPublicKey_[REDACTED_APPID].crt  (1.64 KB)
```

These are the certificate files required for Alipay RSA2 signature verification. The app certificate filename contains the Alipay app ID: `[REDACTED_APPID]`.

The files inspected are public certificate files, not private keys. However their presence in the repository alongside the rest of the credential material in this codebase warrants explicit documentation.

## Why this matters

The public certificates themselves do not grant access to the Alipay account. However their presence in the repository confirms the Alipay app ID in use, which combined with the Alipay account credentials in F-05 gives a complete picture of the Alipay integration.

The more significant concern is what is not present here: the corresponding private key file. Alipay RSA2 integration requires an application private key for signing requests. That private key must exist somewhere. Given the pattern observed throughout this codebase of committing all credential material to the repository, the private key may exist in a location not yet examined, in git history, or on a production server at a path referenced in the properties files.

## What needs to change

- Search the full git history of [REDACTED_GITEE_REPO_2] for any file containing `-----BEGIN RSA PRIVATE KEY-----` or `-----BEGIN PRIVATE KEY-----`.
- Search production servers for any `.pem`, `.key`, or `.p12` files in the application directories.
- If the private key has been committed to the repository at any point in its history, treat it as compromised and regenerate the Alipay RSA2 key pair through the Alipay open platform.
- Remove all certificate files from the repository. They are runtime dependencies that belong in a secure configuration location on the server, not in source control.
