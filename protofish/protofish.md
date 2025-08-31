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

# Basic Operational Concept
In this chapter, we're going to deal with the basic operational concept of Protofish.

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
Protofish can easily manage multiple parallel binary streams. Streams are identified by their ID. `StreamOpen` and `StreamClose` messages notify the peer the open and close of the stream, respectively. Since they are messages, they also have the ability to track contexts. Therefore, a binary stream can be a response to a request.

## Benchmarking
Based on the experience gained with HTTP/WS, we've learned that benchmark monitoring is a crucial part of achieving reduced latency. So Protofish now has the ability to automatically perform various benchmark tests to assess the performance.

# Specification

## Specifications of Requirements
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC2119.

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

### Primary Stream
The very first of the upstream protocol MUST be a reliable stream, which is called **primary stream**. Protofish entirely relies on the primary stream to handle all protocol messages.

## The Standard
Protofish is designed to support adaptation to a wide range of upstream protocols. However, the recommended implementation setup is QUIC-based upstream protocol, which is specially specified as [QUICFish](/protofish/quicfish.md).

## Summary of Operation
Protofish follows simple operational rules.

### Protofish Finite State Machine (FSM)
This section describes the Protofish operation in terms of a Finite State Machine (FSM).

## Primary Messaging Channel (PMC)
Primary stream MUST be used for messaging. Messages MUST follow the [framing rules](#framing) to frame the binary content. Only payloads SHOULD transferred through PMC.

## Payload
All [payload](#payload)s are defined at [payloads.proto](protos/payload/payloads.proto).

Following list is the payloads and their explanations.
- **Hello** The packet that initializes a handshake
- **OK** Represents the request in the context has been succeeded
- **Error** Represents the request in the context has been failed
- **StreamOpen** The packet that opens a new stream
- **StreamClose** The packet that destroys the existing stream
- **ArbitaryData** Packet to pass any downstream binary message

## Framing
TODO: little endian
TODO: length, content