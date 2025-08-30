# Protofish

## Credits
Written by MincoMK\
Dedicated to the one who is an author of the name *Protofish*.

## Introduction
Protofish is an intermediate transport layer that abstracts transportation of streams and messages between various zako2 components. Protofish itself does not specify the whole process of data flow, but the guideline to implement Protofish over any upstream protocol.

## Features
Protofish supports various simple essential features. Take a brief look at them.

### Arbitary Messaging
Protofish lets you safely send your own binary data through the connection. For example, you can send a data request via Protofish messaging channel.

### Lossy/Loseless Streaming
Protofish supports both lossy and lossless binary streaming. Once you send a message that notifies open stream, the new independant logical stream is initiated and made it able to send the binary data through it. For example, you can leverage lossy binary stream to send a live audio stream.

### Context Tracking
Protofish make it possible to track request and corresponding multiple responses easily.

# Basic Concept

## Messages and Payloads
Messages and payloads are closely related concepts. It's similar to a REST API. It's used for passing events, simple data, and communicating.

### Payload
Payload is a concept similar to a schema at a REST endpoint. It has a structured data format. It's basically a kind of command that makes peer do something. For example, `StreamOpen`, `SocketClose`.

### Message
Message is an envelope for the payload. It contains one of the possible payloads. It's the final serialization format, which is directly sent to the message channel in Protofish.

> **NOTE** It might confusing to distuingish concept of messages and payloads. You can simply understand the payload as a product, and the message as some kind of cardboard box that encloses the product before shipping.

## Context System
Basically, the message channel is *flat*. Client can try to send multiple messages simultaneously, and it's not possible to identify what message is a response to the other message by itself. Once I send a request A to the server, probably I'd want to track and get corresponding response to A. But as I mentioned, since the channel is *flat*, it doesn't guarantee you that the very next message is the response to the request A. This is exactly the reason why context is needed. As soon as I send a request A, a new context is formed. Also all corresponding responses are belonged to the context. Hence we're able to track the corresponding response.\
Methodologically, we mark every messages with a unique context ID. It makes it possible to track the context.

## Stream Management
- Mention about stream creation

## Benchmarking

# Specification

- TODO organize
``
Primary stream MUST be used for messaging. Messages MUST be serialized with Google Protocol Buffers, and streamed via [framing rules](#framing). Schema of Message is specified in proto definitions (at `/protos`). Message MUST contain a context ID, and one of the following payloads.

Following list is a list of the payloads.
- **Hello** The packet that initializes a handshake
- **OK** Represents the request in the context has been succeeded
- **Error** Represents the request in the context has been failed
- **StreamOpen** The packet that opens a new stream
- **StreamClose** The packet that destroys the existing stream
- **ArbitaryData** Packet to pass any downstream binary message
``

## Upstream Protocol
Protofish entirly relies on the upstream protocol. Therefore, strict prerequisites are applied to the upstream protocol. Following list is the prerequisites.
- **Stability** It MUST guarantee full binary integrity for all reliable streams.
- **Non-blocking** The stream SHOULD NOT block nor interfere other streams.
- **Multiplexing** It MUST follow these conditions.
    - It MUST provide a reliable stream, and also an interface to create more isolated streams.
    - It MUST have a mechanism to notify the other side that a new stream is created.
    - It MUST have a way to distuingish a stream by ID, regardless of the reliability of the stream.

## Version Management
Protofish's version system follows [Semantic Versioning](https://semver.org). Also, it follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) rule.

## The Standard
Protofish is designed to support adaption into wide range of upstream protocol. However, the recommended implementation setup is QUIC-based upstream protocol, which is specially specified as [QUICFish](/protofish/quicfish.md).

### Server Behavior
TODO: All packets by small heading, specify behavior\
TODO: specify client or server or both for message list

### Client Behavior

## Overall Communication Diagram
### Handshake

## Framing
TODO: little endian
TODO: length, opcode, context id

## Terminology (TODO: separate, highlight noted word)
- **Protofish** trivial
- **Upstream Protocol** The protocol in which Protofish is implemented
- **Server** The place where the upstream protocol is hosted
- **Client** The place where the upstream protocol connects to the server
- **Reliable Stream** A network stream that guarantees no byte loss and perfect integrity
- **Unreliable Stream** A network stream that does not check the packet integrity
- **Stream Channel** A single isolated logical binary stream
- **Primary Stream** The first stream channel provided on open
- **Context** A single session that continues with flowing context pattern
- **Message Channel** A stream channel used to send payloads
- **Payload** A unit of data structure passed through the message channel