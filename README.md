# WePay Chat Platform
WePay's internal CI was breached, here we are, documenting it because we aren't like the fucking losers that did it.  

Three active build jobs were examined: CPL, 试玩测试, and 豆豆赚. Each builds a Java/Spring web application and deploys to a production Tomcat instance. 试玩测试 and 豆豆赚 share an identical git revision and codebase deployed to different targets. CPL is a separate codebase sharing the same credential infrastructure as the other two.

All sensitive values are redacted. The goal is for a developer to understand exactly what was exposed.

---

## Structure

```
findings/         <- Individual technical findings
credentials/      <- Full inventory of every exposed credential category
infrastructure/   <- Reconstructed network and service topology  
surface/          <- Application endpoint and integration surface map
```

---

## Affected Jobs

| Job | Latest Build | Deployment Target | Build Profile |
|-----|-------------|-------------------|---------------|
| CPL | #1846 | [REDACTED_PROD_IP_1] via root SSH | prod |
| 试玩测试 | #9845 | localhost | dev |
| 豆豆赚 | #6651 | [REDACTED_INTERNAL_IP_1] via root SSH | prod |

---

## Finding Index

```
F-01  Root SSH deployment to production
F-02  Unauthenticated remote build trigger
F-03  Jenkins master is a production application server
F-04  All secrets committed to source repository
F-05  Plaintext third-party credential notebook in workspace
F-06  Shiro filter chain misconfiguration
F-07  Covert proxy infrastructure with root credentials in repo
F-08  Unmanaged local JAR dependencies
F-09  MD5 used for payment API signatures
F-10  AJP connector exposed without secret
F-11  Nginx running as root with database ports proxied externally
F-12  Error pages disabled in production
F-13  SSH public keys committed to source repository
F-14  Alipay RSA certificates committed to source repository
F-15  Workspace fully readable via CI interface
```

---

The single most urgent action is credential rotation across every category in `credentials/CREDENTIAL-INVENTORY.md`. Hardening the infrastructure while the credentials remain valid accomplishes nothing.
Update;
When we log into a a specific account, we have the option to choose between;
```
Hunan Yinbing Technology Co., Ltd. (湖南引冰科技有限公司) 
Anhui Yunhuida Technology Co., Ltd. (安徽云汇达科技有限公司)
```
