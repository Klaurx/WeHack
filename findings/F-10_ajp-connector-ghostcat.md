# F-10: AJP Connector Exposed Without Secret (Ghostcat)

## What was observed

The Tomcat configuration committed in `server-template/game-backstage/conf/server.xml` has an AJP 1.3 connector enabled on a non-default port with no `secret` or `requiredSecret` attribute:

```xml
<Connector port="21009" protocol="AJP/1.3" redirectPort="21443"/>
```

The server also exposes a shutdown port with the default command string:

```xml
<Server port="21005" shutdown="SHUTDOWN">
```

This configuration is part of the `server-template` directory committed to the repository and described in the checklist as the template deployed to production hosts.

## Why this matters

An AJP connector without a required secret is vulnerable to CVE-2020-1938, known as Ghostcat. This vulnerability allows an unauthenticated attacker who can reach the AJP port to read arbitrary files from the web application, including configuration files and source code packaged in the WAR. In some configurations it also allows file inclusion leading to remote code execution.

The AJP port is port 21009. Whether this port is exposed externally depends on the firewall configuration of each host, which is not directly observable. However, given that the Nginx configuration proxies port 21000 (the HTTP connector) externally, and given the general posture of this infrastructure, treating the AJP port as potentially reachable is the conservative and correct assumption.

The shutdown port using the default `SHUTDOWN` command means any process on the local machine can send a TCP connection to port 21005 with the string `SHUTDOWN` and cleanly stop the Tomcat instance.

## What needs to change

- Disable the AJP connector entirely if it is not being used by a load balancer or Apache httpd in front of Tomcat. Most deployments observed here use Nginx as a reverse proxy, which does not use AJP.
- If AJP must remain enabled, add `secret="[strong-random-value]"` and `requiredSecret="true"` to the connector definition.
- Change the shutdown port command from the default `SHUTDOWN` to a long random string, or set `port="-1"` to disable the shutdown port entirely.
- Verify that port 21009 is not reachable from outside the host's loopback interface.
