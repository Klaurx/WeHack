# Credential Inventory

All values are redacted. This document exists so a developer can confirm every category has been rotated without needing to re-read every finding.
You thought you were going to get credentials? Aww, grow up buddy. 
---

## Database

| Category | Detail | Source | Status |
|----------|--------|--------|--------|
| MySQL prod | Host [REDACTED_INTERNAL_IP_2]:3306, user root, password [REDACTED_DB_PASSWORD] | prod.properties (all three jobs) | Rotate |
| MySQL dev | Host [REDACTED_INTERNAL_IP_3]:3306, user root, password [REDACTED_DB_PASSWORD] | dev.properties | Rotate |
| SQL Server legacy | Host [REDACTED_SQLSERVER_HOST]:[REDACTED_PORT], user [REDACTED], password [REDACTED] | applicationContext.xml (commented out, CPL) | Rotate |

Note: dev and prod MySQL share the same password.

---

## Cache

| Category | Detail | Source | Status |
|----------|--------|--------|--------|
| Redis prod | Alibaba Cloud managed instance [REDACTED_REDIS_HOST]:6379, password [REDACTED_REDIS_PASSWORD_PROD], db 2/4/8 | prod.properties | Rotate |
| Redis dev | Host [REDACTED_INTERNAL_IP_4]:6379, password [REDACTED_REDIS_PASSWORD_DEV] | dev.properties | Rotate |

---

## Alibaba Cloud

| Category | Detail | Source | Status |
|----------|--------|--------|--------|
| OSS Access ID | [REDACTED_OSS_ACCESS_ID] | prod.properties (all three jobs) | Rotate |
| OSS Access Key | [REDACTED_OSS_ACCESS_KEY] | prod.properties (all three jobs) | Rotate |
| OSS Bucket | [REDACTED_OSS_BUCKET] on oss-cn-hangzhou.aliyuncs.com | prod.properties | Rotate bucket policy |
| Aliyun App Push | App ID [REDACTED_ALIYUN_PUSH_APPID] | prod.properties | Rotate |

Same OSS credentials used across all three applications.

---

## WeChat

| Category | Detail | Source | Status |
|----------|--------|--------|--------|
| Web AppID | [REDACTED_WX_WEB_APPID] | prod.properties | Rotate secret |
| Web AppSecret | [REDACTED_WX_WEB_APPSEC] | prod.properties | Rotate |
| App AppID | [REDACTED_WX_APP_APPID] | prod.properties | Rotate secret |
| Pay merchant 1 | Merchant number [REDACTED_WX_MERCHANT_1], secret [REDACTED_WX_HONGBAO_SEC_1] | prod.properties | Rotate |
| Pay merchant 2 | Merchant number [REDACTED_WX_MERCHANT_2], secret [REDACTED_WX_HONGBAO_SEC_2] | prod.properties | Rotate |
| Pay certificate 1 | apiclient_cert.p12 at /opt/wxhb/ | prod.properties | Audit and replace |
| Pay certificate 2 | apiclient_cert.p12 at /opt/wxhb1/ | prod.properties | Audit and replace |
| Open Platform account | Email [REDACTED_WX_OPEN_EMAIL], password [REDACTED_WX_OPEN_PASSWORD] | doc file | Rotate |

---

## Alipay

| Category | Detail | Source | Status |
|----------|--------|--------|--------|
| Enterprise account | QQ email [REDACTED_ALIPAY_QQ_EMAIL], password [REDACTED_ALIPAY_PASSWORD] | doc file | Rotate |
| App ID | [REDACTED_ALIPAY_APPID] | doc file, certificates | Audit |
| RSA2 certificates | Three cert files in server-template/江西/ | git repository | Audit for private key |

---

## Game Channel SDK Credentials

All of the following had appKey, appSecret, mediaId, token, or callback key values committed to prod.properties and/or the doc file.

| Channel | Platforms | Source |
|---------|-----------|--------|
| ABX (AiBianXian) | Android + iOS appKey/appSecret | prod.properties + doc |
| DuoYou | Android + iOS mediaId/key | prod.properties + doc |
| JXW (JuXiangWan) | Android + iOS mid/token | prod.properties + doc |
| TaoJin | Android + iOS mtId/mtKey | prod.properties + doc |
| XianWan | appId/appSecret | prod.properties + doc |
| XQ (HzXiQu) | Android + iOS appId/appSecret/key | prod.properties + doc |
| XW (PcEggs/XiangWan) | pid/key | prod.properties + doc |
| XYX | Android + iOS mtId/mtKey + callback keys | prod.properties + doc |
| QW | pid/appSecret | prod.properties |
| DDZ | Login credentials | doc |

All channel credentials in prod.properties must be rotated via the respective channel's developer portal.

---

## Third-Party Platform Accounts (from doc file)

These are operational accounts, not SDK integrations. All require direct login rotation on the respective platform.

| Platform | Account Identifier | Notes |
|----------|-------------------|-------|
| ChangLian Pay | Phone [REDACTED] | Payment processor |
| XinZhangHu | Account [REDACTED], login + transaction passwords | Payment platform |
| MoguXingQiu channel | Company account [REDACTED] | Game distribution |
| YiPinJianZhi | Username doudouzhuan | Part-time job platform |
| DianZhuan | Username [REDACTED] | Task platform |
| JuLiuBao | Email [REDACTED] | Ad platform |
| ZhuanZhuanHuYu | Username [REDACTED] | Channel platform |
| DuoYou panel | Email [REDACTED] | Ad/game panel |
| Tencent Coral | Email [REDACTED], password [REDACTED] | Tencent points wall |
| Meituan Union | Phone [REDACTED] | Delivery cashback |
| Mobile Safety Alliance | Account [REDACTED] | OAID certification |
| Mob SMS/Push | Phone [REDACTED], password [REDACTED] | SMS service |
| Fulu platform | Phone [REDACTED], password [REDACTED] | Gift card/top-up |
| Email account | [REDACTED_EMAIL] | Login + payment passwords |
| ShowDoc | Account [REDACTED] | API documentation platform |
| Taobao open platform | appKey/appSecret | CPL prod.properties |

---

## Infrastructure Credentials

| Category | Detail | Source | Status |
|----------|--------|--------|--------|
| V2Ray proxy server root | Host [REDACTED_PROXY_IP], user root, password [REDACTED_PROXY_ROOT_PASSWORD] | server-template/v2ray/doc | Rotate immediately |
| VMess UUID | [REDACTED_VMESS_UUID] | server-template/v2ray/config.json | Rotate |
| Developer SSH key | Public key for lowkey@lowkey.local | server-template/checklist | Audit authorized_keys |
| Jenkins SSH key | Public key for root@gamebackstage | server-template/checklist | Audit authorized_keys |
| Jenkins credential ID | 6ef2eb70-7266-490f-9fa0-2fc529a5eb82 | Build logs (all three jobs) | Rotate stored credential |

---

## Git Repository

| Category | Detail |
|----------|--------|
| Gitee repo 1 (CPL) | [REDACTED_GITEE_REPO_1], user lowkey3636 |
| Gitee repo 2 (试玩测试 + 豆豆赚) | [REDACTED_GITEE_REPO_2], user lowkey3636 |

Audit Gitee access logs for both repositories. Determine who has cloned or had read access. All credentials in this inventory must be rotated before repository access is cleaned up, because anyone who has already cloned the repository already has the credentials.
