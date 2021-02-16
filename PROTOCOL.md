# NetWorks protocol version 1 draft

## Types

Only the following primitive types are supported:

### Logical boolean

1 byte. `0` is `false`, everything else (1-255) is `true`

### Integral numbers

| Type    | Size    | Endianness | Signed   | Minimum              | Maximum             |
|---------|---------|------------|----------|----------------------|---------------------|
| `byte`  | 8 bits  | -          | Unsigned | 0                    | 255                 |
| `short` | 16 bits | Big        | Signed   | -32768               | 32767               |
| `int`   | 32 bits | Big        | Signed   | -2147483648          | 2147483647          |
| `long`  | 64 bits | Big        | Signed   | -9223372036854775808 | 9223372036854775807 |

### Floating point numbers

| Type     | Size    | Endianness | Signed |
|----------|---------|------------|--------|
| `float`  | 32 bits | Big        | Signed |
| `double` | 64 bits | Big        | Signed |

### Type[]

| Type   | What                         |
|--------|------------------------------|
| `int`  | Array length                 |
| `Type` | Nth item, array length times |

### String

UTF-8-encoded byte[]

## Message layout

| Type   | Parameter          | Role                                                           |
|--------|--------------------|----------------------------------------------------------------|
| `long` | Message ID         | ID of the message to optionally respond to, starting from 0    |
| `long` | Respond to         | ID of the message to respond to or negative for not to respond |
| `byte` | Command            | Command byte value                                             |
|        | Command parameters | Command parameters (no break between them)                     |

## Commands

### Discovery broadcast

Value: `byte` 0

This command is sent through UDP to notify other devices about a NetWorks server running on a machine.

| Type     | Parameter                 | Role                                                   |
|----------|---------------------------|--------------------------------------------------------|
| `int`    | NetWorks version          | To know what protocol version to use for communication |
| `string` | NetWorks application      | To avoid detection other applications using NetWorks   |
| `string` | NetWorks application info | To information about the application, like version     |

### Connection request

Value: `byte` 1

This command informs the server about an incoming connection request to accept or reject and sends information about the client trying to connect.

This command must be invoked first.

| Type     | Parameter        | Role                                                                                           |
|----------|------------------|------------------------------------------------------------------------------------------------|
| `int`    | NetWorks version | To know what protocol version to use for communication                                         |
| `string` | Client name      | Name of the connectng application or device                                                    |
| `string` | Client info      | Optional information about the client application                                              |
| `string` | Client token     | To be used during reconnection. The server identifies the client with it after connection loss |

### Accept connection request

Value: `byte` 2

This command informs the client that the connection request was accepted and sends information about the sever.

This command must be an answer to a client's connection request.

| Type     | Parameter        | Role                                                                                           |
|----------|------------------|------------------------------------------------------------------------------------------------|
| `int`    | NetWorks version | To know what protocol version to use for communication                                         |
| `string` | Server name      | Name of the connectng application or device                                                    |
| `string` | Server info      | Optional information about the client application                                              |
| `string` | Server token     | To be used during reconnection. The server identifies the client with it after connection loss |

### Reject connection request

Value: `byte` 3

This command informs the client that the connection request was rejected.

This command must be an answer to a client's connection request.

| Type     | Parameter     | Role                                            |
|----------|---------------|-------------------------------------------------|
| `int`    | Error code    | To inform the client about the rejection reason |
| `string` | Error message | To inform the client more about the rejection   |

### Disconnect

Value: `byte` 4

This command informs the other party about the disconnection.

| Type     | Parameter             | Role                                               |
|----------|-----------------------|----------------------------------------------------|
| `int`    | Disconnection reason  | To inform the client about the disconection reason |
| `string` | Disconnection message | To inform the client more about the disconnection  |

### Ping

Value: `byte` 5

This command pings the other party.

This command can be an answer to another ping command.

| Type     | Parameter | Role                                           |
|----------|-----------|------------------------------------------------|
| `long`   | Time      | Milliseconds sinch the Unix epoch (1970-01-01) |

### Quick command invocation

Value: `byte` 6

This command instucts the remote party to invoke a method and return its result. The method "short" name should be specified explicitly (like method attribute or annotation).

| Type   | Parameter         | Role                                             |
|--------|-------------------|--------------------------------------------------|
| `byte` | Method name       | Short name for a method (for speed optimization) |
|        | Method parameters | Primitive parameters to pass to that method      |

### Command invocation

Value: `byte` 7

This command instucts the remote party to invoke a method and return its result.

| Type     | Parameter         | Role                                        |
|----------|-------------------|---------------------------------------------|
| `string` | Method name       | Method name to invoke                       |
|          | Method parameters | Primitive parameters to pass to that method |

### Error codes

| Value    | Meaning                                                             |
|----------|---------------------------------------------------------------------|
| `byte` 0 | Success                                                             |
| `byte` 1 | Not connected. A connection must be requested and accepted first    |
| `byte` 2 | This happened or was initiated by the user                          |
| `byte` 3 | Not supported                                                       |
| `byte` 4 | An unknown command was received (see [commands](#commands))         |
| `byte` 5 | Unexpected command (the command is valid but unexpected)            |
| `byte` 6 | Unexpected parameter (the command is valid, but a parameter is not) |
