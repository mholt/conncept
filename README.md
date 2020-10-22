Project Conncept
================

In my spare time, I'm working on a layer 4 app for Caddy that I call *Project Conncept*. It routes raw TCP/UDP streams using composable handlers, much like how Caddy's HTTP server is configured.

When I started working on this, it was available only to sponsors, but now it is available to the public:

### Click here for the code: https://github.com/mholt/caddy-l4

Like the [HTTP app](https://caddyserver.com/docs/modules/http#servers/routes), layer 4 routes consist of matchers and handlers; if a route's matchers match the properties of the connection or first few bytes of the stream, its handlers will be invoked, one after another. This enables a great degree of flexibility when composing routes.

You can match on more than just client IPs and protocols. For example, [all existing HTTP matchers](https://caddyserver.com/docs/json/apps/http/servers/routes/match/) can be used in layer 4 routes with the same degree of efficiency as the HTTP server. Similarly, you can also use [any existing TLS connection matchers](https://caddyserver.com/docs/json/apps/http/servers/tls_connection_policies/match/).

As with all other Caddy configurations, you can change it while it is running using [Caddy's on-line config API](https://caddyserver.com/docs/api).

✨And of course, all certificates used for TLS termination are fully-managed in regular Caddy fashion.

**💚 Become a sponsor (or advocate for your company to sponsor) to continue development: https://github.com/sponsors/mholt -- thank you!**
