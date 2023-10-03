# ⛩ Reverse Proxies

Running a reverse proxy in front of `sqliterg` is almost mandatory if you want to expose it on the internet. More than that, there are a number of reverse proxies that allow you to protect a http connection with https, using a free certificate provided for example by [Let's Encrypt](https://letsencrypt.org) or similar.

Read more [here](../security.md#use-a-reverse-proxy-if-going-on-the-internet).

We'll show here how to integrate with two popular solutions, [`caddy`](https://caddyserver.com) and [`NGINX`](https://www.nginx.com).

### Caddy

To access `sqliterg` from `https://sqlite.mydomain.com`:

1. Expose ports 80 and 443 of the server to which the DNS points;
2. Run `sqliterg`. Leave the port as 12321;
3. Launch `caddy`:

{% code lineNumbers="true" %}
```bash
caddy reverse-proxy --from sqlite.mydomain.com --to localhost:12321
```
{% endcode %}

{% hint style="danger" %}
You'll need to launch `caddy` with root/admin privileges, as it must access privileged ports.
{% endhint %}

### NGINX

NGINX is quite complex to configure, and its configuration is beyond the scope of this document. Usually, we make use of [LinuxServer's `Swag`](https://docs.linuxserver.io/general/swag) Docker image, paired with `sqliterg`'s own docker image. The relevant config is in `nginx/site-confs/default`:

{% code lineNumbers="true" %}
```nginx
server {
        listen 443 ssl http2;
        server_name sqlite.mydomain.com;
        include /config/nginx/proxy-confs/*.subfolder.conf;
        include /config/nginx/ssl.conf;
        location / {
                proxy_pass http://localhost:12321/;
        }
}
```
{% endcode %}
