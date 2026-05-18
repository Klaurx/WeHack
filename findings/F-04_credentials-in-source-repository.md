# F-04: All Secrets Committed to Source Repository

## What was observed

Production and development credentials for every service this platform depends on are stored directly in the source repository under `fastcpl-common/src/main/resources/` and `fastshiwan-common/src/main/resources/`, in plaintext `.properties` files organized by Maven build profile (`dev/` and `prod/`).

The following credential categories were found committed in these files across all three jobs:

**Database**
- MySQL connection URL, username, and password for both dev and prod environments
- Both environments use the same password ([REDACTED_DB_PASSWORD])
- Prod database is accessible at [REDACTED_INTERNAL_IP_2]:3306

**Redis**
- Host, port, password, and database index for both environments
- Dev Redis password ([REDACTED_REDIS_PASSWORD_DEV]) differs from prod ([REDACTED_REDIS_PASSWORD_PROD])
- Prod Redis is an Alibaba Cloud managed instance at [REDACTED_REDIS_HOST]

**Alibaba Cloud OSS**
- Access ID: [REDACTED_OSS_ACCESS_ID]
- Access Key: [REDACTED_OSS_ACCESS_KEY]
- Bucket name: [REDACTED_OSS_BUCKET]
- Endpoint: oss-cn-hangzhou.aliyuncs.com
- Same credentials used across all three applications

**WeChat**
- Web AppID: [REDACTED_WX_WEB_APPID]
- Web AppSecret: [REDACTED_WX_WEB_APPSEC]
- App AppID: [REDACTED_WX_APP_APPID]
- Two WeChat Pay merchant numbers with their secrets: [REDACTED_WX_HONGBAO_SEC_1], [REDACTED_WX_HONGBAO_SEC_2]
- WeChat Pay certificate files referenced at `/opt/wxhb/apiclient_cert.p12` and `/opt/wxhb1/apiclient_cert.p12`

**Third-party game channel SDKs**
The following channel integrations each had their appKey, appSecret, mediaId, or token committed:
- ABX (Android + iOS)
- DuoYou (Android + iOS)
- JXW (Android + iOS)
- TaoJin (Android + iOS)
- XianWan
- XQ (Android + iOS)
- XW
- XYX (Android + iOS with callback keys)
- QW
- DDZ

**Aliyun Push**
- App push ID: [REDACTED_ALIYUN_PUSH_APPID]

**Additional in CPL prod only**
- Taobao open platform app credentials
- Additional channel-specific keys not present in 试玩测试/豆豆赚

**Previously active credentials found in commented-out code**
Inside `applicationContext.xml` in CPL, a commented-out Druid datasource bean contains a SQL Server connection string with credentials:
- Host: [REDACTED_SQLSERVER_HOST]:[REDACTED_SQLSERVER_PORT]
- Database: [REDACTED_SQLSERVER_DB]
- Username: [REDACTED_SQLSERVER_USER]
- Password: [REDACTED_SQLSERVER_PASSWORD]

Commented-out code in a git repository is not deleted. It remains in git history indefinitely.

## Why this matters

Every person who has ever cloned, forked, or been given read access to these repositories has had full access to every production system this platform touches. The Gitee repositories are accessible to anyone the repository owner has granted access to, and potentially more depending on repository visibility settings.

Credentials in git history cannot be removed by deleting the file. They require a full history rewrite, which is destructive and must be coordinated carefully.

The practice of using the same credentials across dev and prod removes the last remaining safety boundary between environments.

## What needs to change

- Rotate every credential listed above immediately, before any other remediation.
- Remove all credential files from the repository and rewrite git history to purge them from all branches and tags.
- Adopt a secrets management approach such as environment variables injected at runtime, a secrets vault, or encrypted secret storage that is never committed to source control.
- Use separate credentials for dev and prod environments.
- Audit Gitee repository access logs to determine who has cloned or accessed these repositories.
