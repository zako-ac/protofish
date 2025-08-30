# Protofish

## Credits
Written by MincoMK\
Dedicated to the one who is an author of the name *Protofish*.

## Why Protofish?
Before the development of Protofish, traditional HTTP/WS method has been used to handle audio requests from TapHub. Let me briefly explain the various problems we encountered while using it, and how Protofish solved it.\
Traditional method leveraged a HTTP stream to send an encoded audio stream. It was the simplest and easiest method to transfer streamed audio. However, the easy method shortly unveiled numerous problems. These are some of them.

### TCP Head-of-Line Blocking


- TODO: Mention old HTTP/WS method and its problems:
    - HTTP blocking, unnecessary integrity checks -> solved by UDP
    - HTTP ACK! Dirty, low library support, needs low level access
    - Reversed req/res pattern: how to do it without port forwarding! -> solved by a unified connection
    - and finally.. this is unified in one!

## Introduction
Protofish is an intermediate transport layer that abstracts transportation of streams and messages between various zako2 components. Protofish itself does not specify the whole process of data flow, but the guideline to implement Protofish over any upstream protocol.

## Terminology
- **Protofish** trivial
- **Upstream Protocol** The protocol in which Protofish is implemented
- **Server** The place where the upstream protocol is hosted
- **Client** The place where the upstream protocol connects to the server
- **Reliable Stream** A network stream that guarantees no byte loss and perfect integrity
- **Unreliable Stream** A network stream that does not check the packet integrity
- **Stream Channel** A single isolated logical binary stream
- **Primary Stream** The first stream channel provided on open
- **Context** A single session that continues with flowing context pattern

## Version Management
Protofish's version system follows [Semantic Versioning](https://semver.org). Also, it follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) rule.

## Upstream Protocol
Protofish entirly relies on the upstream protocol. Therefore, strict prerequisites are applied to the upstream protocol. Following list is the prerequisites.
- **Stability** It MUST guarantee full binary integrity for all reliable streams.
- **Non-blocking** The stream SHOULD NOT block nor interfere other streams.
- **Multiplexing** It MUST follow these conditions.
    - It MUST provide a reliable stream, and also an interface to create more isolated streams.
    - It MUST have a mechanism to notify the other side that a new stream is created.
    - It MUST have a way to distuingish a stream by ID, regardless of the reliability of the stream.

## The Standard
Protofish is designed to support adaption into wide range of upstream protocol. However, the recommended implementation setup is QUIC-based upstream protocol, which is specially specified as [QUICFish](/protofish/quicfish.md).

## Message
Primary stream MUST be used for messaging. Messages MUST be serialized with Google Protocol Buffers, and streamed via [framing rules](#framing). Schema of Message is specified in [protofish.proto](/protofish/proto/protofish.proto). Message contains a context ID, and one of the following payloads.
- **Hello** The packet that initializes a handshake
- **OK** Represents the request in the context has been succeeded
- **Error** Represents the request in the context has been failed
- **AudioStreamOpen** The packet that opens a new unreliable audio stream
- **AudioStreamClose** The packet that destroys the existing audio stream
- **ArbitaryData** Packet to pass any downstream binary message

### Server Behavior
TODO: All packets by small heading, specify behavior\
TODO: specify client or server or both for message list

### Client Behavior

## Overall Communication Diagram
### Handshake

### Flowing Context Pattern
Flowing context pattern is a standard messaging pattern in Protofish. Context is a logical boundary that marks the specific communication flow. For example, all messages in [4-Way Audio Flow](#4-way-audio-request) share the same context. The pattern is achieved by leaving **context ID** to every messages in relation with the context.

## Framing
TODO: little endian
TODO: length, opcode, context id

## Something Specific TODO
### 4-Way Audio Request