# Fly Dev Tunnel

Developers commonly use apps like [ngrok](https://ngrok.com), [localtunnel](https://localtunnel.github.io/www/), and [cloudflared](https://github.com/cloudflare/cloudflared) to expose a local web service at a publicly-accessible domain name. This allows them to develop using HTTPS or share a preview of a site with a client without messing with firewalls and port forwarding.

With [Fly](https://fly.io), [Wireguard](https://www.wireguard.com), and a reverse proxy, you can achieve something similar while using a custom domain and mapping subdomains to different ports. Best of all, it's free as long as you stay within Fly's [generous limits](https://fly.io/docs/about/pricing/).

## 1. Install flyctl, the Fly command-line utility

Follow the instructions for your OS from the [Fly docs](https://fly.io/docs/flyctl/installing/). After installing, run `flyctl auth login` to sign up or sign in.

## 2. Connect to Fly via Wireguard

[Install Wireguard](https://www.wireguard.com/install/) for your OS. Next, run `flyctl wireguard create` to create a Wireguard config. Use the following settings:

```
Region: dev
DNS name for peer: <any name you want for your local machine>
Filename: fly.conf
```

Note: The default config sets Fly as your DNS resolver. If you don't need Fly's internal DNS features, edit `fly.conf` and comment out the DNS line with a pound sign (`#`).

Finally, setup the tunnel in Wireguard:
- On Mac or Windows, open the Wireguard app and click "Import Tunnel(s) from File". Once imported, click "Activate" to connect.
- On Linux, [use the command line](https://fly.io/docs/reference/wireguard/).

## 3. Create the reverse proxy app

Run `mkdir tunnel && cd tunnel` to create an empty app folder. Run `flyctl init` to create a `fly.toml` config file. Use the following settings

```
App name: <any name you want for you app>
Select builder: Image
Select image: lukelambert/fly-dev-tunnel
Internal port: 8080
```

## 4. Configure the reverse proxy

The reverse proxy is configured using two environment variables:

- `SUBDOMAINS`: A comma-separated list in the format `subdomain:local_port`. An underscore (`_`) matches the default (catch all) domain.
- `UPSTREAM`: The IPv6 address of your local machine on the Wireguard network. To find this value, open the `fly.conf` file created in step 2 and copy the address starting with `fd` and ending just before the forward slash.

Edit `fly.toml` and add these lines at the bottom, replacing the values with your own:

```
[env]
  SUBDOMAINS = "_:8000"
  UPSTREAM = "fdaa:0:ffff:ffff:ffff:0:0:1"
```

## 5. Deploy the reverse proxy

Run `flyctl deploy`. Once the app is deployed, you should have a fully functional tunnel from `https://your-app-name.fly.dev` to port `8000` on your local machine.


## 6. (Optional) Connect custom domains and subdomains

Visit the [Apps dashboard](https://fly.io/apps/) and select your app. Under the Certificates section, follow the instructions to add custom domains. You can also add a wildcard subdomain, but this incurs a monthly fee. To map subdomains to ports, update your `fly.toml` and re-run `flyctl deploy`. Example:

```
[env]
  SUBDOMAINS = "_:8000,app1:9001,app2:9002"
  UPSTREAM = "fdaa:0:ffff:ffff:ffff:0:0:1"
```

## Notes

All traffic will be proxied over IPv6, so your web service should bind to an IPv6 address. To take down the tunnel and prevent traffic from reaching your machine, simply deactivate the Wireguard tunnel.