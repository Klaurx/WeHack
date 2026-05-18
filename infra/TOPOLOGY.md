# Infrastructure Topology

Reconstructed from build logs, properties files, nginx configuration, and workspace contents. All IP addresses and hostnames are redacted.

---

## Servers Identified

| Identifier | Role | Notes |
|------------|------|-------|
| [REDACTED_PROD_IP_1] | CPL production app server + Nginx reverse proxy | CPL deploys here via root SSH. Nginx proxies HTTP on port 9000, WebSocket on 9100, download on 21000. Also proxies MySQL on 13306 and Redis on 6379 externally via stream block. |
| [REDACTED_JENKINS_HOST] | Jenkins master + 试玩测试 production app server | Hosts Jenkins CI. Also runs the 试玩测试 Tomcat instance locally at /opt/shiwan/tomcat. |
| [REDACTED_INTERNAL_IP_1] | 豆豆赚 production app server | Internal network address. 豆豆赚 deploys here via root SSH. Pinpoint APM agent running. |
| [REDACTED_INTERNAL_IP_2] | MySQL database | Internal network. Hosts databases: cpl, prod (used by both 试玩测试 and 豆豆赚). Root credentials known. |
| [REDACTED_INTERNAL_IP_3] | Redis | Internal network. Used by 试玩测试 prod and 豆豆赚 prod (db 2). |
| [REDACTED_REDIS_HOST] | Alibaba Cloud managed Redis | Used by CPL prod (db 8) and dev environments. |
| [REDACTED_PROXY_IP] | V2Ray proxy server | Separate cloud instance. Runs V2Ray VMess on port 55555 via Docker. Proxies port 50001 to [REDACTED_DOWNSTREAM_IP]:50013. Root credentials exposed in repository. |
| [REDACTED_DOWNSTREAM_IP] | Unknown downstream proxy target | Receives proxied traffic from [REDACTED_PROXY_IP]:50001 on port 50013. Role unknown. |

---

## Internal Network Ranges

Two internal network ranges are visible from the properties files and nginx configuration:

- `172.16.x.x` - application servers and database hosts
- `172.31.x.x` - referenced in nginx stream proxy for MySQL

---

## Domain Names

The following domains are referenced across configuration files and frontend assets. All are redacted to [REDACTED_DOMAIN_n] format.

| Identifier | Usage |
|------------|-------|
| [REDACTED_DOMAIN_1] | Main application API endpoint, referenced in frontend JS |
| [REDACTED_DOMAIN_2] | WebSocket endpoint |
| [REDACTED_DOMAIN_3] | Hot update / OTA update endpoint |
| [REDACTED_DOMAIN_4] | Download endpoint |
| [REDACTED_DOMAIN_5] | Lanniao78 variants of the above (nginx server_name entries) |
| [REDACTED_DOMAIN_6] | Backstage safe URL, referenced in CPL prod properties |
| [REDACTED_DOMAIN_7] | Backstage safe URL, referenced in 试玩测试 and 豆豆赚 prod properties |
| [REDACTED_DOMAIN_8] | Chatbot avatar static asset host, referenced in frontend JS |
| [REDACTED_DOMAIN_9] | Alibaba Cloud OSS prefix for user-facing assets |

---

## CDN

The checklist documents Kunlun CDN (kunlunca.com / kunluncan.com) as the CDN provider in front of the main application domains. Four CDN-mapped hostnames are present. The CDN configuration itself was not examined.

---

## Service Ports Summary

| Port | Protocol | Service | Exposed |
|------|----------|---------|---------|
| 80 | HTTP | Nginx (multiple vhosts) | External |
| 9000 | HTTP | Main application Tomcat | Via Nginx proxy |
| 9100 | WebSocket | Live agent WebSocket server | Via Nginx proxy |
| 13306 | TCP | MySQL stream proxy | External via Nginx stream |
| 6379 | TCP | Redis stream proxy | External via Nginx stream |
| 21000 | HTTP | game-backstage Tomcat HTTP | Via Nginx proxy |
| 21005 | TCP | Tomcat shutdown port | Local |
| 21009 | AJP | Tomcat AJP connector | Potentially local |
| 21443 | HTTPS | Tomcat redirect target | Configured but SSL not confirmed active |
| 55555 | TCP | V2Ray VMess proxy | External (separate host) |
| 15002 | HTTP | Rasa chatbot webhook (localhost) | Internal only |
| 19090 | WebSocket | Live agent WS handler (localhost) | Internal only |
| 10114 | HTTP | Chatbot static asset server | External (referenced in frontend) |

---

## Application Relationships

```
CPL codebase (fastcpl)
  └── Builds fastcpl-web-backstage.war
  └── Deploys to [REDACTED_PROD_IP_1]:/opt/cpl/tomcat/webapps/ROOT
  └── Connects to MySQL at [REDACTED_PROD_IP_1]:13306 (proxied)
  └── Connects to Alibaba Cloud Redis [REDACTED_REDIS_HOST]:6379

试玩测试 codebase (fastshiwan, dev profile)
  └── Builds fastshiwan-web-backstage.war
  └── Deploys to localhost:/opt/shiwan/tomcat/webapps/ROOT
  └── Connects to MySQL at [REDACTED_INTERNAL_IP_2]:3306
  └── Connects to Redis at [REDACTED_INTERNAL_IP_3]:6379

豆豆赚 codebase (fastshiwan, prod profile, identical revision to 试玩测试)
  └── Builds fastshiwan-web-backstage.war
  └── Deploys to [REDACTED_INTERNAL_IP_1]:/opt/shiwan/tomcat/webapps/ROOT
  └── Connects to MySQL at [REDACTED_INTERNAL_IP_2]:3306 (same DB host as 试玩测试)
  └── Connects to Redis at [REDACTED_INTERNAL_IP_3]:6379 (same Redis, different db index)
```

Note that 试玩测试 and 豆豆赚 share the same MySQL host and Redis host. A compromise of either application's database credentials gives access to both applications' data.
