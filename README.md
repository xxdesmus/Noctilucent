
![Noctilucent](docs/Noctilucent.jpeg)

## Description

This is the code developed and presented as part of the DEF CON 28 (Safe Mode) talk "Domain Fronting is Dead, Long Live Domain Fronting: Using TLS 1.3 to evade censors, bypass network defenses, and blend in with the noise."

Domain fronting, the technique of circumventing internet censorship and monitoring by obfuscating the domain of an HTTPS connection was killed by major cloud providers in April of 2018. However, with the arrival of TLS 1.3, new technologies enable a new kind of domain fronting. This time, network monitoring and internet censorship tools are able to be fooled on multiple levels. This talk will give an overview of what domain fronting is, how it used to work, how TLS 1.3 enables a new form of domain fronting, and what it looks like to network monitoring. You can circumvent censorship and monitoring today without modifying your tools using an open source TCP and UDP transport tool (Cloak) that will be released alongside this talk.

Talk: [Youtube](https://youtu.be/TDg092qe50g)

Slides are available in the `docs` folder.

Compiled test client, test server, and Cloak client binaries are available under "Releases."

## Demos

Noctilucent test client bypassing Palo Alto 10.0 TLS decryption: [Youtube](https://youtu.be/TDg092qe50g?t=1002)

Noctilucent Cloak client: [Youtube](https://youtu.be/TDg092qe50g?t=1192)

Noctilucent Cloak client with CobaltStrike: [Youtube](https://youtu.be/TDg092qe50g?t=1322)

Noctilucent built into DeimosC2 Agent: [Youtube](https://youtu.be/TDg092qe50g?t=1417)

## Layout

```
Noctilucent
├── Cloak # The Cloak fork
│   ├── build # Compiled Cloak binaries 
│   ├── cmd # Cloak client and server source code
│   ├── example_config # Configs for Cloak and Shadowsocks
│   └── internal # Code for Cloak client and server
├── DeimosC2 # Modified HTTPS agent source code
├── _dev
│   └── GOROOT # Modified Go source tree (tls is placed in here)
├── client # Test client source code
│   └── build # Test client binaries
├── docs # Slides and other docs
│   ├── example-traffic.pcapng # 2 requests made with the Noctilucent test client
│   └── screenshots
├── findfronts # Helper to find domains that can be used with Noctilucent
├── server # Test server (HTTP and websockets)
├── tls # Noctilucent tls library (copied to _dev/GOROOT)
└── websocket # Noctilucent websocket library
```

## Build from source

### Setup - Ubuntu 18.04
Install latest go from [golang.org](https://golang.org/dl/)

Note: Developed with Go 1.14

```
sudo apt install make git gcc
make client # this will take a minute, we are replacing tls in the standard library and recompiling it
make cloak
```
### Setup - macOS 10.15

Install latest go from [golang.org](https://golang.org/dl/)

Note: Developed with Go 1.14

```
xcode-select --install
make client # this will take a minute, we are replacing tls in the standard library and recompiling it
make cloak
```
Client binaries are available in `client/build` and Cloak client binaries are available in `Cloak/build`.

The modifications made to Cloak can be seen in `Cloak/git_diff.patch`

## Testing

### Test Client

The `server/server.go` contains a sample HTTP and websocket server to test against.
You can build the server and run it on your own VPS with `go build` inside the server directory.
Setup a Cloudflare account and point a domain at your VPS. Ensure that SSL/TLS is set to "Flexible"
as the server is not using TLS (or setup a reverse-proxy to handle TLS). Run the server binary (as root, it binds to port 80) on the VPS and 
replace `defcon28.hackthis.computer` in the examples with your domain. You can replace `-serverName` 
with anything (really, even non-domain strings) and `-TLSHost`
with any site that is hosted behind Cloudflare (lots, try medium.com).
Cloudflare has a helpful site for finding frontable domains [here](https://www.cloudflare.com/case-studies/), or you can choose any from `findfronts/frontable100k.txt`.

#### TLS 1.3

![](docs/screenshots/plain-https.png)

#### TLS 1.3 with ESNI (ECH)

![](docs/screenshots/esni-https.png)

#### TLS 1.3 with ESNI (ECH) and Hiding

![](docs/screenshots/fonted-no-sni.png)

#### TLS 1.3 with ESNI (ECH), Hiding, and decoy SNI

![](docs/screenshots/fonted-with-sni.png)

#### TLS 1.3 with ESNI (ECH), Hiding, and decoy SNI - Websocket

![](docs/screenshots/websocket.png)

### Cloak Client

1. Setup a standard Cloak + Shadowsocks server using [this script](https://github.com/HirbodBehnam/Shadowsocks-Cloak-Installer/blob/master/Cloak2-Installer.sh).
2. Download a [shadowsocks-rust binary](https://github.com/shadowsocks/shadowsocks-rust/releases) for your platform.
3. Use the `noctilucent-cloak-client` and `sslocal` to create a local SOCKS proxy that is hidden behind a Cloudflare hosted domain. Example configs are available in `Cloak/example_config` and should be edited to match the values given by the Cloak + Shadowsocks setup script.


## Thanks

This project is based on cloudflare's [tls-tris](https://github.com/cloudflare/tls-tris) and inspired by [DigiNinja's](https://digi.ninja/blog/cloudflare_example.php) rough openssl PoC work. It also includes a modified version of ahhh's DNS over HTTPS code, [godns](https://github.com/ahhh/godns) and of course [Cloak](https://github.com/cbeuw/Cloak).