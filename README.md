# Encrypted DNS Server

An easy to install, zero maintenance proxy to run an encrypted DNS server.

Written in Rust.

## Protocols

The proxy aims at supporting the following protocols:

- [DNSCrypt v2](https://github.com/DNSCrypt/dnscrypt-protocol/blob/master/DNSCRYPT-V2-PROTOCOL.txt)
- [Anonymized DNSCrypt](https://github.com/DNSCrypt/dnscrypt-protocol/blob/master/ANONYMIZED-DNSCRYPT.txt) (WIP)
- DNS-over-HTTP (DoH)

All of these can be served simultaneously, on the same port (usually port 443). The proxy automatically detects what protocol is being used by each client.

## Installation

The proxy uses recent features of the Rust compiler, and currently requires rust-nightly.

Rust can installed with:

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Once rust is installed, the proxy can be compiled and installed with the following command, to be run in the source directory:

```sh
cargo install --path .
```

## Setup

The proxy requires a recursive DNS resolver, such as Knot, PowerDNS or Unbound.

That resolver can run locally and only respond to `127.0.0.1`. External resolvers such as Quad9 or Cloudflare DNS can also be used, but this may be less reliable due to rate limits.

In order to support DoH in addition to DNSCrypt, a DoH proxy must be running as well. [rust-doh](https://github.com/jedisct1/rust-doh) is the recommended DoH proxy server. DoH support is optional, as it is currently way more complicated to setup than DNSCrypt due to certificate management.

Review the `encrypted-dns.toml` configuration file. This is where all the parameters can be configured, including the IP addresses to listen to. You should probably at least change the `provider_name` setting.

Start the proxy. It will automatically create a new provider key pair if there isn't any.

The DNS stamps are printed. They can be used directly with `dnscrypt-proxy`.

There is nothing else to do. Certificates are automatically generated and rotated.

## Migrating from dnscrypt-wrapper

If you are currently running an encrypted DNS server using `dnscrypt-wrapper`, moving to the new proxy is simple:

- Double check that the provider name in `encrypted-dns.toml` matches the one you previously configured. If you forgot it, it can be recovered [from its DNS stamp](https://dnscrypt.info/stamps/).
- Run `dnscrypt-dns --import-from-dnscrypt-wrapper secret.key`, with `secret.key` being the file with the `dnscrypt-wrapper` provider secret key.

Done. Your server is now running the new proxy.

## State file

The proxy creates and updates a file named `encrypted-dns.state` by default. That file contains the provider secret key, as well as certificates and encryption keys.

Do not delete the file, unless you want to change parameters (such as the provider name), and keep it secret, or the keys will be lost.

Putting it in a directory that is only readable by the super-user is not a bad idea.
