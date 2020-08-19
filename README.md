socks5
======

Package socks5 implements a "SOCKS Protocol Version 5" server.

This server supports a subset of RFC 1928:

* auth methods: "NO AUTHENTICATION REQUIRED", "USERNAME/PASSWORD"
* commands: "CONNECT"
* address types: "IP V4 address", "DOMAINNAME", "IP V6 address"
(but tested "DOMAINNAME" only)

INSTALL
-------

```sh
go get -u github.com/PayaMousavi/socks5
```
Additional Feature
======
verSocks5 value changed to 0x06 instead of 0x05 for bypass Iran's firewall(client-side modification needed)

USAGE
-----

```go
package main

import (
	"github.com/PayaMousavi/socks5"
	"log"
)

func main() {
	srv := socks5.New()
	srv.AuthUsernamePasswordCallback = func(c *socks5.Conn, username, password []byte) error {
		user := string(username)
		if user != "guest" {
			return socks5.ErrAuthenticationFailed
		}

		log.Printf("Welcome %v!", user)
		c.Data = user
		return nil
	}
	srv.HandleConnectFunc(func(c *socks5.Conn, host string) (newHost string, err error) {
		if host == "example.com:80" {
			return host, socks5.ErrConnectionNotAllowedByRuleset
		}
		if user, ok := c.Data.(string); ok {
			log.Printf("%v connecting to %v", user, host)
		}
		return host, nil
	})
	srv.HandleCloseFunc(func(c *socks5.Conn) {
		if user, ok := c.Data.(string); ok {
			log.Printf("Goodbye %v!", user)
		}
	})

	srv.ListenAndServe(":12345")
}
```
