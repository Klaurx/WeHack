# F-05: Plaintext Third-Party Credential Notebook in Workspace

## What was observed

A file named `doc` (no extension) exists in the root of both the 试玩测试 and 豆豆赚 workspaces. It is committed to the git repository at [REDACTED_GITEE_REPO_2] and therefore present in every clone of that repository.

The file is a manually maintained plaintext document containing login credentials for every third-party platform the platform operator uses. It is not a config file consumed by the application. It is a personal credential notebook that someone committed to a shared repository.

The following platform categories had credentials exposed in this file. All values are redacted here:

**Payment processors**
- ChangLian Pay (河南) - account, phone, password
- XinZhangHu comprehensive services platform - merchant name, login account, login password, transaction password

**Game channel backends**
- ABX / AiBianXian channel backend - username, password, URL
- XW / PcEggs SDK backend - pid, key, account, password, URL
- XQ / HzXiQu channel - username, password, URL
- XianWan channel - username, password, URL
- DuoYou channel - account, password, URL
- JXW channel - account, password, URL, Android mid/token, iOS mid/token
- DDZ channel backend - login name, password, URL
- TaoJin - account, password, URL
- MoguXingQiu - account, password, URL
- YiPinJianZhi - account, password, URL
- DianZhuan - account, password, URL
- JuLiuBao - account, password, URL
- QingTing (via QingtingAPI integration)
- ZhuanZhuanHuYu channel - username, password

**Alipay enterprise account**
- QQ email used as Alipay account: [REDACTED_ALIPAY_QQ_EMAIL]
- Password: [REDACTED_ALIPAY_PASSWORD]
- Alipay app ID: [REDACTED_ALIPAY_APPID]

**WeChat Open Platform**
- Account email: [REDACTED_WX_ACCOUNT_EMAIL]
- Password: [REDACTED_WX_OPEN_PASSWORD]

**Tencent Coral (积分墙)**
- Account: [REDACTED_TENCENT_CORAL_EMAIL]
- Password: [REDACTED_TENCENT_CORAL_PASSWORD]

**Meituan Union**
- Account phone: [REDACTED_MEITUAN_PHONE]
- Login via verification code (no static password, but account identity exposed)

**Mobile Safety Alliance**
- Account: [REDACTED_MSA_ACCOUNT]
- Password: [REDACTED_MSA_PASSWORD]
- Associated email and phone

**SMS / Push service (Mob platform)**
- Phone: [REDACTED_MOB_PHONE]
- Password: [REDACTED_MOB_PASSWORD]

**Fulu platform**
- Account phone: [REDACTED_FULU_PHONE]
- Password: [REDACTED_FULU_PASSWORD]

**Additional account**
- Email: [REDACTED_EMAIL_2] with login and payment passwords

**Android APK signing certificate fingerprints**
The file contains the MD5, SHA-1, and SHA-256 fingerprints of the production Android APK signing certificate. While fingerprints alone are not the private key, their presence confirms the signing identity and could be used to verify a fake APK claiming to be the same app.

**Production deploy command**
The file contains a hardcoded rsync command targeting [REDACTED_PROD_IP_1], confirming that server's role and the deployment user.

## Why this matters

This file represents the largest single credential exposure in the entire engagement. It is not a configuration issue or a framework misuse. Someone deliberately wrote down every password they use and committed it to a shared repository. The git history of [REDACTED_GITEE_REPO_2] now permanently contains this data unless a full history rewrite is performed.

Every account listed above must be treated as fully compromised. If any of these accounts share passwords with other services not listed here, those services are also compromised.

## What needs to change

- Immediately revoke and rotate every credential listed above across all platforms.
- Remove the file from the repository and perform a complete git history rewrite.
- Never store credentials in any file that is committed to version control, regardless of whether the file has a recognizable extension or name.
- Use a password manager for operational credentials. Most of them have team sharing features that do not involve committing a text file to git. The year is 2026.
