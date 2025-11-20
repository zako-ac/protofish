# QUICfish
QUICfish (f is a lowercase letter) is an implementation of Protofish, with upstream protocol set to QUIC.

### Notes
This document is written in an assumption that you already read and understood the [Protofish specification](protofish.md).

## Upstream Protocol Satisfaction
Let's check whether QUIC satisfies [preconditions of an upstream protocol](protofish.md#upstream-protocol).

- **Stability** Yes. QUIC supports full integrity checking and retransmission mechanism.
- **Non-blocking** Yes. QUIC leverages UDP frames to achieve this.
- **Multiplexing** Yes. QUIC also leverages UDP frames to achieve this.

Yes it does.

## Primary Messaging Channel (PMC)
Client SHOULD open a reliable bidirectional stream immediately after the connection. It becomes a PMC.

## Streaming
Due to the requirements of Protofish, it is mandatory to add an extra logic that allows Protofish to identify a raw QUIC stream to bridge QUIC and Protofish. QUICfish has different rules for each of reliable and unreliable streams.

### Reliable Stream
QUICfish leverages a normal bidirectional QUIC stream for reliable streaming.
#### Stream ID Resolution
The ID of the reliable stream is resolved by the first **unsigned 64-bit integer in little-endian** transmitted through the stream.
```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Stream ID (LE)|                                               |
+-+-+-+-+-+-+-+-+                                               +
|                             ...Data                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### Unreliable Stream
Pure QUIC does not support unreliable streaming, intrinsically. Hence, the QUIC UTP SHOULD support datagram ([RFC 9221](https://datatracker.ietf.org/doc/html/rfc9221)) extension to support the underlying Protofish feature.
#### Stream ID Resolution
Like in the [Reliable Stream](#reliable-stream), QUICfish add **unsigned 64-bit integer in little-endian** ahead of the data. However, as QUIC datagram supports limited amount of bytes, the data is chunked.Then, each chunks are tagged with the stream ID before them.

Also, note that the actual size of the chunk is 8 bytes smaller than the QUIC chunk size.
```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Stream ID (LE)|                                               |
+-+-+-+-+-+-+-+-+                                               +
|                            ...Chunk                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

## Zako Standard
Zako2 infrastructure leverages QUICfish as its standard transport medium.
