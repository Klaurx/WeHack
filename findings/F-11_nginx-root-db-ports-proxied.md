# F-11: Nginx Running as Root with Database Ports Proxied Externally

## What was observed

The Nginx configuration committed in `server-template/nginx.conf` contains two separate issues of concern.

The first line of the configuration:

```
user root;
```

Nginx worker processes run as root.

The second issue is in the stream block at the bottom of the configuration, which proxies TCP connections directly to internal database hosts:

```
stream {
    server {
        listen 13306;
        proxy_pass [REDACTED_INTERNAL_MYSQL_IP]:3306;
    }
    server {
        listen 6379;
        proxy_pass [REDACTED_INTERNAL_REDIS_IP]:6379;
    }
}
```

This means the Nginx host is accepting TCP connections on port 13306 and 6379 and forwarding them directly to the internal MySQL and Redis instances. This is confirmed by the CPL prod properties file which connects to MySQL at [REDACTED_PROD_IP_1]:13306 rather than directly to the database host.

## Why this matters

Nginx running as root means a vulnerability in Nginx, in any of the proxied applications, or in the Nginx configuration itself that allows process escape gives an attacker root on that host immediately. There is no privilege boundary between the web server and the operating system.

The database port proxying is a more immediate concern. It means MySQL and Redis are not just accessible on the internal network — they are accessible from anywhere that can reach the Nginx host on ports 13306 and 6379. Combined with the database credentials exposed in F-04, this means the production database and cache are directly reachable and authenticatable from outside the internal network using credentials that are now known.

The Redis instance in particular has no TLS in this configuration. Authentication is a single password over plaintext TCP.

## What needs to change

- Remove the `user root;` directive and run Nginx as a dedicated low-privilege user such as `nginx` or `www-data`.
- Remove the stream proxy blocks for MySQL and Redis entirely. Application servers should connect to databases via internal network addresses directly, not via a public-facing proxy.
- If the database stream proxy exists to allow developer access, replace it with SSH tunneling with strong key-based authentication instead.
- Ensure the MySQL and Redis whitelist rules only permit connections from known application server IPs, not from the Nginx host's public interface.
- Enable TLS for Redis connections.
