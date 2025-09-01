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

## Zako Standard
Zako2 infrastructure leverages QUICfish as its standard transport medium.