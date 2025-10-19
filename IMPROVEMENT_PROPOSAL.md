# Protofish Improvement Proposal

This document outlines a proposal for improving the Protofish protocol. The proposed changes aim to enhance stream management, error handling, message structure, security, and data streaming capabilities.

## 1. Enhanced Stream Management

### 1.1. `StreamData` Message

**Problem:** The current protocol uses a generic `ArbitaryData` message for data transfer, which lacks a direct association with a specific stream. This makes it cumbersome to route data to the correct stream, especially in a multiplexed scenario.

**Proposal:** Introduce a new `StreamData` message to explicitly carry data for a particular stream.

```proto
// In payload/v1/payloads.proto

message StreamData {
  uint64 stream_id = 1;
  bytes data = 2;
}
```

And replace `ArbitaryData` with `StreamData` in the `Payload` oneof in `message.proto`:

```proto
// In payload/v1/message.proto

message Payload {
  oneof payload {
    // ... other messages
    StreamData stream_data = 6; // Replaces ArbitaryData
    // ... other messages
  }
}
```

### 1.2. `StreamClose` with Reason

**Problem:** The current `StreamClose` message doesn't provide any information about why a stream is being closed. This makes debugging and handling stream closures more difficult.

**Proposal:** Add a `reason` field to the `StreamClose` message.

```proto
// In payload/v1/payloads.proto

message StreamClose {
  uint64 stream_id = 1;
  optional string reason = 2;
}
```

## 2. Richer Error Handling

**Problem:** The current `Error` message is too simplistic and the `ErrorType` enum is not comprehensive.

**Proposal:** Expand the `ErrorType` enum and add more context to the `Error` message.

```proto
// In common/v1/schema.proto

enum ErrorType {
  ERROR_TYPE_UNSPECIFIED = 0;
  ERROR_TYPE_TIMEOUT = 1;
  ERROR_TYPE_INVALID_REQUEST = 2;
  ERROR_TYPE_STREAM_NOT_FOUND = 3;
  ERROR_TYPE_INTERNAL_SERVER_ERROR = 4;
  ERROR_TYPE_PERMISSION_DENIED = 5;
}

// In payload/v1/payloads.proto

message Error {
  common.v1.ErrorType error_type = 1;
  string message = 2;
  optional uint64 original_context_id = 3; // To correlate with the original request
}
```

## 3. Improved Message Structure

**Problem:** The purpose of `context_id` is not explicitly documented.

**Proposal:** Add a comment to the `Message` definition to clarify the role of `context_id`.

```proto
// In payload/v1/message.proto

// Represents a root schema used in a primary channel
message Message {
  // A unique identifier for the context of the message.
  // Used to correlate requests and responses.
  uint64 context_id = 1;
  Payload payload = 2;
}
```

## 4. Adding a Security Layer

**Problem:** The protocol currently lacks any form of encryption or authentication, which is a major security risk.

**Proposal:** Integrate a security layer using a standard and widely-accepted protocol like TLS. This would provide encryption, authentication, and data integrity.

The handshake process would be modified to include a TLS handshake:

1.  **Client -> Server:** `ClientHello` (as is)
2.  **Server -> Client:** `ServerHello` (as is)
3.  **TLS Handshake:** A standard TLS handshake would be performed over the established connection.
4.  **Encrypted Communication:** All subsequent Protofish messages would be sent over the encrypted TLS channel.

This proposal avoids reinventing the wheel for security and leverages a well-vetted and robust solution.

## 5. Efficient Data Streaming

**Problem:** The current `ArbitaryData` (or the proposed `StreamData`) message sends a single chunk of bytes. This is inefficient for large data transfers.

**Proposal:** Introduce a mechanism for chunking data within the `StreamData` message.

```proto
// In payload/v1/payloads.proto

message StreamData {
  uint64 stream_id = 1;
  bytes data = 2;
  bool is_last_chunk = 3; // Indicates if this is the last chunk of data for the stream
}
```

This allows the sender to break large data into smaller chunks and the receiver to reassemble them. The `is_last_chunk` flag signals the end of the data transfer for that stream.

These proposed changes will make the Protofish protocol more robust, secure, and efficient.
