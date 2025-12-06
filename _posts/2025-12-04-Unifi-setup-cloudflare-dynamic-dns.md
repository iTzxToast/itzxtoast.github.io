---
title: Unifi setup Cloudflare dynamic DNS
date: 2025-12-04 20:00 +0100
categories: [Unifi, Network]
tags: [Unifi, Cloudflare, Network, DNS]
---

With UniFi Network 9.1.96, Cloudflare became a native Dynamic DNS provider for WAN connections.
Before this I used a small script on a LXC container to update my DNS records.
To simplify the setup I switched to the integrated Cloudflare DDNS feature.

---

## Set Up Cloudflare DDNS in UniFi

From the UniFi Portal:

1. Open Settings and go to Internet
2. Select your WAN interface
3. Scroll to Dynamic DNS and create a new entry
4. Select Cloudflare
5. Enter the FQDN of your A record
6. Enter your DNS zone
7. Paste your [Cloudflare API token](https://developers.cloudflare.com/fundamentals/api/get-started/create-token/)

---

## Force a DDNS Update from the UniFi CLI

If you want to verify the configuration without waiting for the next IP update you can do this manually.

SSH into your UniFi gateway and use your UniFi password.
```
root@<gateway-ip>
```

Identify the active inadyn config file:
```
ps aux | grep inadyn
```

Example Output:
```
root /usr/sbin/inadyn -n -s -C -f /run/ddns-ppp0-inadyn.conf -P /run/ddns-ppp0-inadyn.pid -b ppp0
```

Use the config path from the output, for example:<br>
`/run/ddns-ppp0-inadyn.conf`

<br>
Run a forced update:
```
/usr/sbin/inadyn -n -1 --force -f CONFIG PATH
```

Example output:
```
inadyn: In-a-dyn version 2.12.0
Guessing DDNS plugin 'default@cloudflare.com
' from 'cloudflare:1'
Update forced for alias home.cyens.com, new IP#
Updating IPv4 cache for blog.cyens.com
```

---

## Conclusion

The Cloudflare integration in UniFi Network 9.1.96 replaces custom scripts and workarounds over DDNS tools like DNS-O-Matic.
Configuration is simple and updates run reliably.