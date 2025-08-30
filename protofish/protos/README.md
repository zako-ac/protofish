# Protofish Definitions
This directory contains `.proto` files representing Protobuf schema definitions in Protofish.

## Structure
`.proto` file structure is defined as in below. The root directory (`/`) is where this file is located.
- `/common/` Contains common data schema definitions.
    - `/common/schema.proto` Contains various utility schemas like version.
- `/payload/` Contains definitions related to the protocol message.
    - `/payload/message.proto` Contains abstract envelope schema for all kind of payloads.
    - `/payload/payloads.proto` Contains every payloads.