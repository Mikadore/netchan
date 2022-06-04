# NetChan Protocol V0.1.0

## Abstract 

NetChan is a simple, binary protocol for communication between Rust programs.
The protocol runs over TCP, and employs optional encryption. For initiation,
the protocol distinguishes between client (connection initiator) and server
(connection receiver).  After that, data can be send bidirectionally at any 
time. NetChan exchanges serialized structs called messages. Both endpoints
must (de)serialize to and from the same struct. (Compatible serde versions, same 
field types and order). 

## Initiation

The client starts by sending 18 bytes with the following meaning:

### Connection request

| Value         | Length (bytes) | Format       |
|---------------|----------------|--------------|
| "NETCHAN\0"   | 8              | ASCII string | 
| Major Version | 2              | LE Integer   |
| Minor Version | 2              | LE Integer   |
| Patch Version | 2              | LE Integer   |
| Encryption    | 4              | LE Integer   | 

The protocol identifies itself with the magic string "NETCHAN\0"
(ASCII, null terminated).
The version integers correspond to the semver version of the protocol
implemented (Seen at the top of the document).
The encryption integer selects the encryption scheme used for the rest of
the communication, see **Encryption** for more information.

### Response

| Value       | Length (bytes) | Format       |
|-------------|----------------|--------------|
| "NETCHAN\0" | 8              | ASCII string |
| Error code  | 4              | LE Integer   |

Upon receiving a connection request, the server will respond
restating the protocol magic, and indicating an error code.
The error code can have these possible values:

| Value | Meaning                                                                       |
|-------|-------------------------------------------------------------------------------|
| 0     | Success: The server understood the connection request and is able to continue |
| 1     | Error: The client's data is not conformant with the spec                      |
| 2     | Error: Incompatible Versions                                                  |
| 3     | Error: Unsupported encryption scheme                                          |


## Encryption

The client specifies an encryption scheme to be used, possible values are:

| Value | Protocol   | Custom |
|-------|------------|--------|
| 0     | None       | N/A    |
| 1     | Simple AES | Yes    |
| 2     | TLS 1.3    | No     |

(None: No encryption is used)

After receiving a response indicating success, the client will initiate
the chosen protocol with the server. Negotiation may fail, in that case
the connection is closed. All communication coming after this point is 
built ontop the chosen encryption scheme.

## Format confirmation

NetChan uses the [bincode](https://github.com/bincode-org/bincode) serialization
format for fast and efficiet encoding of structs. To ensure the same structs
are used on both endpoints, after establishing the encrypted connection, the client
sends the format identifier, an arbitrary amount of bytes to 
identify the format/struct used (Those bytes can be ASCII strings, random, or a cryptographic hash).
Importantly, this hash is stored/calculated on both the server and the client.

| Value             | Length (bytes)       | Format     |
|-------------------|----------------------|------------|
| Identifier Length | 4                    | LE Integer |
| Format Identifier | {Indentifier Length} | Bytes      |

The server will now compare its hash with the client's. It sends back a single byte:

| Value   | Length (bytes) | Format |
|---------|----------------|--------|
| Success | 1              | Byte   |


| Value | Meaning |
|-------|---------|
| 0     | Success |
| 1     | Fail    |

## Messaging

If the format is confirmed, the protocol was successfully initiated.
Both endpoints can now send messages back and forth. A simple framing
format is used:

| Value  | Length (bytes) | Format     |
|--------|----------------|------------|
| Length | 4              | LE Integer |
| Data   | {Length}       | Bincode    |

All communication and synchronization occurs over messages. 
The protocol doesn't define any form of communication, except 
for a shutdown signal: If the value of **Length** is zero, this 
signifies to close the connection. 

Note: If you require confirmation that a message has been received, 
you are required to build that onto another communication layer. E.g.
sending a confirmation message.

Note: If you're building a communication protocol, and don't just want to send data,
you can use enums. Bincode is space efficient, and a message will only be as long
as required by the encoded enum variant.