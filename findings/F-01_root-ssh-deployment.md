# F-01: Root SSH Deployment to Production

## What was observed

All three jobs deploy compiled artifacts directly to production servers using `rsync` and `ssh` as the `root` user. The exact pattern used in CPL:

```
sudo rsync -a --delete -e ssh fastcpl-web-backstage/target/fastcpl-web-backstage/ root@[REDACTED_PROD_IP_1]:/opt/cpl/tomcat/webapps/ROOT
sudo ssh root@[REDACTED_PROD_IP_1] 'sudo /opt/cpl/tomcat/restart.sh'
```

豆豆赚 deploys to an internal network address:

```
sudo rsync -a --delete -e ssh fastshiwan-web-backstage/target/fastshiwan-web-backstage/ root@[REDACTED_INTERNAL_IP_1]:/opt/shiwan/tomcat/webapps/ROOT
sudo ssh root@[REDACTED_INTERNAL_IP_1] 'sudo /opt/shiwan/tomcat/restart.sh'
```

试玩测试 deploys locally using rsync without SSH, meaning the Jenkins master itself hosts a production Tomcat instance (see F-03).

## Why this matters

The Jenkins service account has passwordless root SSH access to every production server in this deployment. Any code execution on the Jenkins host, or any ability to trigger or modify a build, translates directly to root access on every downstream server. There is no privilege boundary between the build system and production.

The `--delete` flag on rsync means a malicious build could wipe the entire webapps directory on the remote host as part of a normal-looking deployment.

The restart scripts run as root and kill Java processes by path matching, which is coarse enough to affect unintended processes if path collisions exist.

## What needs to change

- Create a dedicated low-privilege deployment user on each target server with write access only to the specific webapps directory.
- Remove root SSH access from the Jenkins host entirely.
- Replace direct SSH deployment with a proper artifact handoff mechanism.
- Rotate the SSH keys used for deployment (see F-13 for the key exposure).
