# Quinn-jls
[中文](./README.md)|[English](./README_en.md)

This is a fork of [quinn](https://github.com/quinn-rs/quinn) which replaces tls layer with [JLS](https://github.com/JimmyHuang454/JLS)
aimed at high performance,low latency anti-whitelist proxy.

## Features
* Multiplex powered by quic
* User-space bbr congestion controller
* zero rtt support powered by quic and tls1.3
* SNI camouflage (anti sni whitelist)
* certificate and domain name free
* UDP forward for a failed JLS authentication.

## About Zero RTT
Even zero rtt is enabled, it's not always available for each connnection. It has strict 
constraints. Any constrait broken will lead to a 1 rtt connection
* To setup a zero rtt connection, both server and client must have session ticket (zero rtt key) in memory. Namely a fresh started client will always use one rtt connection
* The session ticket can't be expired (ususlly serveral hours)


### Security Risk of Zero RTT
Zero Rtt is provided by quic and tls1.3. It's well known that tls1.3 zero rtt suffers from the risk of replay attack. However, in my opinion,this security issue is unfairly overstated, especially for a proxy server.

According to [RFC 8446 Section 8](https://datatracker.ietf.org/doc/html/rfc8446#page-98). There are two kinds of replay attacks.
* The first attack is to simply replay zero rtt client hello. This can't be prevented by limit zero rtt key
to be used at most once.
* The second kind of attack involves two servers using the same domain name. For example, suppose a client tries to connect server A with zero rtt data, middle man can caputure this message and block it. Due to retry mechanism,
client will try to send a 1 rtt client hello, and middle man will redirect this message to server B and use captured 0 rtt client hello to connect server A. Then, a successful replay attack is performed. However, for a single server proxy, this won't happen.

For most users, they only use one server as proxy. In such a case, zero rtt is a safe choice


