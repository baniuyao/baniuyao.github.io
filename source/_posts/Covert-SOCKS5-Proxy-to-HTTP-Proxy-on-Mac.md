title: Convert SOCKS5 Proxy to HTTP Proxy on Mac
date: 2015-10-20 17:30:12
tags: 
- Linux
- Mac

---


We usually use ShadowsocksX on Mac to build a SOCKS5 tunnel. But as a developer, I need HTTP Proxy in many places such as npm or some IDE tools. I need a tool to build a HTTP Proxy and pack data stream to SOCKS5 tunnel.

I found a solution of using a tool called `polipo`. We can simply install this tool by `homebrew`.

``` shell
brew instasll polipo
```

After installed `polipo`, let's make it work by a simple command.

```shell
polipo socksParentProxy=localhost:1080
```

`localhost:1080` is your local SOCKS5 address. In default, HTTP proxy created by `polipo` is running on `localhost:8123`.

