Project Conncept
================

In my spare time, I'm working on a layer 4 app for Caddy that I call *Project Conncept*. It routes raw TCP/UDP streams using composable handlers, much like Caddy's HTTP server.

With it, you can listen on sockets/ports and express logic such as:

- "Echo all input back to the client."
- "Proxy all the raw bytes to 10.0.3.14:1592."
- "If connection is TLS, terminate TLS then proxy all bytes to :5000."
- "Terminate TLS; then if it is HTTP, proxy to localhost:80; otherwise echo."
- "If connection is TLS, proxy to :443 without terminating; if HTTP, proxy to :80; if SSH, proxy to :22."
- "If the HTTP Host is `example.com` or the TLS ServerName is `example.com`, then proxy to 192.168.0.4."
- "Block connections from these IP ranges: ..."
- "Throttle data flow to simulate slow connections."

...and much more.

Currently, only the JSON config is supported, but I am planning on a Layer4 Caddyfile, which might look something like this:

```plain
:443 {
	tls example.com  # terminates TLS
	proto {
		ssh  proxy :22
		http proxy :80
	}
}
```

^ This config would let you use both SSH or HTTP in a TLS tunnel on the standard HTTPS port. (This is just a contrived example; I am still designing it, so it will probably change.)

Like the [HTTP app](https://caddyserver.com/docs/modules/http#servers/routes), layer 4 routes consist of matchers and handlers; if a route's matchers match the properties of the connection or first few bytes of the stream, its handlers will be invoked, one after another. This enables a great degree of flexibility when composing routes.

You can match on more than just client IPs and protocols. For example, [all existing HTTP matchers](https://caddyserver.com/docs/json/apps/http/servers/routes/match/) can be used in layer 4 routes with the same degree of efficiency as the HTTP server. Similarly, you can also use [any existing TLS connection matchers](https://caddyserver.com/docs/json/apps/http/servers/tls_connection_policies/match/).

As with all other Caddy configurations, you can change it while it is running using [Caddy's on-line config API](https://caddyserver.com/docs/api).

✨And of course, all certificates used for TLS termination are fully-managed in regular Caddy fashion.


## Try it today

I already have it working for HTTP, TLS, and SSH routes: it can terminate TLS, echo, tee, and proxy _any_ streams.

**💚 Become a sponsor to access it today: https://github.com/sponsors/mholt**

**🏆 Once I reach 50 sponsors, I will release the Apache-licensed source code to the public for everyone to use and participate in. We're already halfway to the goal!**

I want everyone to be able to use this -- it just takes a lot of spare time, hence the sponsorship goal. This project can provide a lot of value to both individuals and businesses, and it take a lot of time to get it right. Thank you for your support and understanding!

**Sponsors can access the repository here: https://github.com/mholt/caddy-l4** - if you need access, feel free to [ping me on Twitter](https://twitter.com/mholt6) (or you can open an issue on this repo).


## Config examples

All these configs are already fully-functional.

This is a simple echo server:

```json
{
	"apps": {
		"layer4": {
			"servers": {
				"example": {
					"listen": [":5000"],
					"routes": [
						{
							"handle": [
								{"handler": "echo"}
							]
						}
					]
				}
			}
		}
	}
}
```

A simple echo server with TLS termination that uses a self-signed cert for localhost (which is short-lived but stays renewed):

```json
{
	"apps": {
		"layer4": {
			"servers": {
				"example": {
					"listen": [":5000"],
					"routes": [
						{
							"handle": [
								{"handler": "tls"},
								{"handler": "echo"}
							]
						}
					]
				}
			}
		},
		"tls": {
			"certificates": {
				"automate": ["localhost"]
			},
			"automation": {
				"policies": [
					{
						"issuer": {"module": "internal"}
					}
				]
			}
		}
	}
}
```


A multiplexer that proxies HTTP to one backend, and TLS to another (without terminating TLS):

```json
{
	"apps": {
		"layer4": {
			"servers": {
				"example": {
					"listen": ["127.0.0.1:5000"],
					"routes": [
						{
							"match": [
								{
									"http": []
								}
							],
							"handle": [
								{
									"handler": "proxy",
									"upstreams": [
										{"dial": [":80"]}
									]
								}
							]
						},
						{
							"match": [
								{
									"tls": {}
								}
							],
							"handle": [
								{
									"handler": "proxy",
									"upstreams": [
										{"dial": [":443"]}
									]
								}
							]
						}
					]
				}
			}
		}
	}
}
```

Same as previous, but filter by HTTP Host header and/or TLS ClientHello ServerName:

```json
{
	"apps": {
		"layer4": {
			"servers": {
				"example": {
					"listen": ["127.0.0.1:5000"],
					"routes": [
						{
							"match": [
								{
									"http": [
										{"host": ["example.com"]}
									]
								}
							],
							"handle": [
								{
									"handler": "proxy",
									"upstreams": [
										{"dial": [":80"]}
									]
								}
							]
						},
						{
							"match": [
								{
									"tls": {
										"sni": ["example.net"]
									}
								}
							],
							"handle": [
								{
									"handler": "proxy",
									"upstreams": [
										{"dial": [":443"]}
									]
								}
							]
						}
					]
				}
			}
		}
	}
}
```
