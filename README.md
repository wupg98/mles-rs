# mles-rs [![Build Status](https://travis-ci.org/jq-rs/mles-rs.svg?branch=master)](https://travis-ci.org/jq-rs/mles-rs) [![Build Status](https://ci.appveyor.com/api/projects/status/github/jq-rs/mles-rs?svg=true)](https://ci.appveyor.com/api/projects/status/github/jq-rs/mles-rs?svg=true)
**Mles-utils** [![crates.io status](https://img.shields.io/crates/v/mles-utils.svg)](https://crates.io/crates/mles-utils)
**Mles** [![crates.io status](https://img.shields.io/crates/v/mles.svg)](https://crates.io/crates/mles)
**Mles-client** [![crates.io status](https://img.shields.io/crates/v/mles-client.svg)](https://crates.io/crates/mles-client)

Mles is a client-server data distribution protocol targeted to serve as a lightweight and reliable distributed publish/subscribe database service. It consists of _Mles-utils_, _Mles-client_ and _Mles_ server implementations.

##Mles protocol overview

Mles clients connect to server using (uid, channel, message) value triple on TCP session where the triple is Concise Binary Object Representation (CBOR) [1] encapsulated. An Mles server then distributes the value triple between all clients on the same channel. An Mles server may save the history of received data, which can be then distributed to new clients when they connect to the Mles server. Every channel uses its own context and is independent of other channels; thus a TCP connection per channel is always used.

An Mles server may contact to an Mles peer server. The Mles peer server sees this connection just as another client connection. This allows Mles servers to share and distribute value triple data in an organized and powerful, but yet simple, manner between other Mles servers.

Every connection between Mles server and client is authenticated using 64-bit SipHash [2]. This allows Mles server to verify that connecting Mles client is really connecting from the endpoint which it claims to be connecting. Optionally an shared secret key can be used as part of SipHash calculation between Mles server and Mles client which allows only those Mles clients to connect which know the shared key. After Mles server has authenticated the connection and moved the connected Mles client to its channel context, SipHash key is ignored by the Mles server. After context change, SipHash key may be used to Mles clients within the channel context.

Mles clients and servers are independent of IP version and do not use IP broadcast or multicast. An Mles server may be configured to use IP anycast.

##Mles protocol details

The protocol header for Mles is as follows:
```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | ASCII 'M'(77) |            Encapsulated data length           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +                          SipHash                              +
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         CBOR encapsulated data (uid, channel, message)        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

CBOR encapsulated data is arranged as follows:
```
uid:     Major type 3, UTF-8 String
channel: Major type 3, UTF-8 String
message: Major type 2, bytestring
```
With Rust the above looks as follows:
```
#[derive(Serialize, Deserialize, Debug)]
pub struct Msg {
    pub uid:     String,
    pub channel: String,
    pub message: Vec<u8>,
}
```
##References:

 1. Concise Binary Object Representation (CBOR), RFC 7049
 2. SipHash: a fast short-input PRF, https://131002.net/siphash/, referenced 4.2.2017
 

