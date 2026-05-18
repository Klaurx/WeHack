# F-03: Jenkins Master is a Production Application Server

## What was observed

The 试玩测试 job deploys its built artifact to the local filesystem of the Jenkins host:

```
sudo rsync -avu --delete fastshiwan-web-backstage/target/fastshiwan-web-backstage/ /opt/shiwan/tomcat/webapps/ROOT
```

No remote host is involved. The Jenkins master runs a Tomcat instance at `/opt/shiwan/tomcat` serving the 试玩测试 application directly to end users. The build log confirms the Tomcat restart happens on the same machine immediately after rsync completes.

## Why this matters

The Jenkins build system and a production web application share the same operating system, filesystem, process table, and network interfaces. There is no isolation between them whatsoever.

A compromise of the build environment is a direct compromise of the production application, and vice versa. A vulnerability in the production application provides a path to the build system. A malicious build step can modify the application it just deployed, read its runtime secrets from memory or disk, or interfere with other Tomcat instances on the same host.

The Pinpoint APM agent observed starting during the 豆豆赚 restart suggests application performance monitoring is also running on these hosts, adding another process with broad system access to the same environment.

## What needs to change

- Separate the build system and production application servers onto distinct hosts with no shared access.
- The Jenkins host should have no inbound application traffic and no outbound access to production data stores.
- Production Tomcat instances should run under their own service accounts with no relationship to the Jenkins service account.
