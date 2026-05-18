# F-15: Full Workspace Readable via CI Interface

## What was observed

The Jenkins workspace for all three jobs is browsable via the Jenkins web interface without authentication. Every file in every workspace — including source code, properties files, and committed supporting files — is viewable and downloadable through the browser.

This is how the contents of this entire report were obtained. No exploitation of application vulnerabilities was required to read any file documented here. The workspace browser is a standard Jenkins feature that becomes a complete source disclosure mechanism when authentication is absent or bypassed.

The following files were directly readable via the workspace browser during this engagement:

- `fastcpl-common/src/main/resources/dev/*.properties`
- `fastcpl-common/src/main/resources/prod/*.properties`
- `fastshiwan-common/src/main/resources/dev/*.properties`
- `fastshiwan-common/src/main/resources/prod/*.properties`
- `fastcpl-web-backstage/src/main/resources/security/applicationContext-shiro.xml`
- `fastcpl-web-backstage/src/main/resources/applicationContext.xml`
- `server-template/nginx.conf`
- `server-template/checklist`
- `server-template/v2ray/config.json`
- `server-template/v2ray/doc`
- `doc` (credential notebook)
- `xx`, `1`, `waimaifanxian` (frontend config files)
- All compiled `.class` files in `target/`
- All build logs

Build console output for all three jobs was also readable, exposing deployment targets, git repository URLs, credential IDs, and remote server addresses.

## Why this matters

The workspace reader does not require the ability to run builds, modify jobs, or interact with the Jenkins API. It is a passive read operation. Any party who can reach the Jenkins web interface can retrieve the full source of every application, all production credentials, all infrastructure documentation, and the complete deployment history.

## What needs to change

- Require authentication for all Jenkins interface access including workspace browsing and build log viewing.
- Apply the principle of least privilege to Jenkins roles: operators who need to trigger builds should not automatically have access to workspace file contents.
- Consider whether workspace files need to be browsable at all after builds complete, and configure workspace cleanup accordingly.
