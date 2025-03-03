# Auto SSL with Snaps

We now include the option to enable Caddy in your snap. Caddy makes use of [Let's Encrypt](https://letsencrypt.org) to automatically provide you SSL protection for your communications.

Starting from release 0.73 you can easily configure everything related to Caddy using snap hooks to ensure your DNS configuration is set up correctly before starting Caddy and Let's Encrypt support.

## Configure everything using snap

{% hint style="info" %}
For 4.x latest AMD64 snaps or 3.x latest ARM64 snaps
{% endhint %}

Set the `SiteUrl` to your domain, make sure it is prefixed with `https`.

```
sudo snap set rocketchat-server siteurl=https://<your domain>
```

Now start Caddy and restart the snap services.

```
sudo systemctl enable --now snap.rocketchat-server.rocketchat-caddy 
sudo snap restart rocketchat-server
```

{% hint style="info" %}
For older snaps
{% endhint %}

If you want to enable SSL and Let's Encrypt certificates you should:

1. Input a URL starting with HTTPS
2. Own the domain name you would like to use
3. Have the correct DNS record set up to resolve your domain name to your public IP (remember DNS records could take some time to propagate).

These next commands will check that configuration is set up correctly before starting the services:

```bash
sudo snap set rocketchat-server caddy-url=https://<your-domain-name>
sudo snap set rocketchat-server caddy=enable
sudo snap set rocketchat-server https=enable
sudo snap run rocketchat-server.initcaddy
```

If no errors were found, it is safe to restart Rocket.Chat and Caddy:

```bash
sudo systemctl restart snap.rocketchat-server.rocketchat-server.service
sudo systemctl restart snap.rocketchat-server.rocketchat-caddy.service
```

In case you don't want to configure SSL for your site, or want to remove SSL configuration:

```bash
sudo snap set rocketchat-server https=disable
sudo snap set rocketchat-server caddy-url=http://<your-domain-name>
sudo snap set rocketchat-server caddy=enable
sudo snap run rocketchat-server.initcaddy
```

If no errors were found, it is safe to restart Rocket.Chat and Caddy:

```bash
sudo systemctl restart snap.rocketchat-server.rocketchat-server.service
sudo systemctl restart snap.rocketchat-server.rocketchat-caddy.service
```

### Check Caddy's logs

```bash
sudo snap logs -f rocketchat-server.rocketchat-caddy
```

## Configure Caddy yourself or use another HTTP Proxy

{% hint style="info" %}
For 4.x latest AMD64 snaps or 3.x latest ARM64 snaps
{% endhint %}

{% hint style="info" %}
Both caddy v2 and caddy v1 (EOL) is delivered in the snap, v2 is prioritized over v1.
{% endhint %}

If you want to configure Caddy yourself, place the Caddyfile in `/var/snap/rocketchat-server/current/` directory and restart rocketchat-server.

```
sudo snap restart rocketchat-server
```

If you want to use some other reverse proxy, just disable Caddy by running.

```
sudo systemctl disable snap.rocketchat-server.rocketchat-caddy
```

{% hint style="info" %}
For older versions
{% endhint %}

In case you plan to use another https proxy or you prefer other options in Caddy configuration, you can disable caddy:

```bash
sudo snap set rocketchat-server caddy=disable
```

Then, edit the Caddyfile found at `/var/snap/rocketchat-server/current/Caddyfile` and write your configuration.

Replace `_caddy-url_` and `_port_` with your site information. For instance, let's say I have example-domain.com pointing at my server.

First, be sure that your DNS has finished resolving **before** attempting to enable SSL. If your DNS is not working yet, you could be instantly [throttled by Let's Encrypt](https://caddyserver.com/docs/automatic-https#testing) for _up to a week_. To test your DNS you can use http:

```bash
http://example-domain.com
proxy / localhost:3000 {
  websocket
  transparent
}
```

and restart Caddy:

```bash
sudo systemctl restart snap.rocketchat-server.rocketchat-caddy
```

You can check that the Caddy service started correctly by running:

```bash
sudo systemctl status snap.rocketchat-server.rocketchat-caddy
```

Once that is tested and resolved, to get secured communications, you can remove the `http://`:

```bash
example-domain.com
proxy / localhost:3000 {
  websocket
  transparent
}
```

**Please note: using an IP address will not work for automatically enabling SSL. You must use a valid hostname (**[**here's why**](https://caddyserver.com/docs/automatic-https)**).**

Now you can restart the Caddy service by running:

```bash
sudo systemctl restart snap.rocketchat-server.rocketchat-caddy
```

You can check that the Caddy service started correctly by running:

```bash
sudo systemctl status snap.rocketchat-server.rocketchat-caddy
```

If everything went well, the site will be accessible at `https://example-domain.com`.

## Testing with an Untrusted Self-Signed Certificate

Simply add the `tls self_signed` directive to your Caddyfile like so:

```bash
https://example-domain.com
tls self_signed
proxy / localhost:3000 {
  websocket
  transparent
}
```

Remember to restart the Caddy service:

```bash
sudo systemctl restart snap.rocketchat-server.rocketchat-caddy
```

This will enable SSL with an untrusted, self-signed certificate for testing purposes.

For details on the Caddy TLS directive, visit [https://caddyserver.com/docs/tls](https://caddyserver.com/docs/caddyfile/directives/tls)

## Redirecting HTTP to HTTPS

Redirecting is handled automatically by caddy by omitting the http / https in front.

```bash
example-domain.com {
  proxy / localhost:3000 {
    websocket
    transparent
  }
}
```

Remember to restart the Caddy service:

```bash
sudo systemctl restart snap.rocketchat-server.rocketchat-caddy
```

## Disabling SSL or Listening on Custom Ports

This configuration will listen **without SSL** on the default port 80:

```bash
http://example-domain.com {
  proxy / localhost:3000 {
    websocket
    transparent
  }
}
```

This configuration will listen **without SSL** on port 8080:

```bash
http://example-domain.com:8080 {
  proxy / localhost:3000 {
    websocket
    transparent
  }
}
```

This configuration will listen **with SSL** on port 8080:

```bash
https://example-domain.com:8080 {
  proxy / localhost:3000 {
    websocket
    transparent
  }
}
```

Note that, without SSL, you can use an IP address:

```bash
http://192.168.1.1:8080 {
  proxy / localhost:3000 {
    websocket
    transparent
  }
}
```

Remember to restart the Caddy service:

```bash
sudo systemctl restart snap.rocketchat-server.rocketchat-caddy
```

## Opening ports when running Rocket.Chat Server from behind router

For Caddy to be able to work from behind a router, the following ports need to be opened between the internet and the server. This is usually achieved through router software or web-interface.

* HTTP: port **80**
* HTTPS: port **443**
