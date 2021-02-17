# NetWorks protocol version 1 draft

## Primitive types

### Logical boolean

| Type   | Type ID  | Size    | Endianness | Signed   | Value   | Range |
|--------|----------|---------|------------|----------|---------|-------|
| `byte` | `byte` 0 | 8 bits  | -          | Unsigned | `false` | 0     |
| `byte` | `byte` 0 | 8 bits  | -          | Unsigned | `true`  | 1-255 |

### Integral numbers

| Type    | Type ID  | Size    | Endianness | Signed   | Minimum              | Maximum             |
|---------|----------|---------|------------|----------|----------------------|---------------------|
| `byte`  | `byte` 1 | 8 bits  | -          | Unsigned | 0                    | 255                 |
| `short` | `byte` 2 | 16 bits | Big        | Signed   | -32768               | 32767               |
| `int`   | `byte` 3 | 32 bits | Big        | Signed   | -2147483648          | 2147483647          |
| `long`  | `byte` 4 | 64 bits | Big        | Signed   | -9223372036854775808 | 9223372036854775807 |

### Floating point numbers

| Type     | Type ID  | Size    | Endianness | Signed |
|----------|----------|---------|------------|--------|
| `float`  | `byte` 5 | 32 bits | Big        | Signed |
| `double` | `byte` 6 | 64 bits | Big        | Signed |

## Type layout

### Primitives

| Type   | Parameter | Role                                        |
|--------|-----------|---------------------------------------------|
| `byte` | Type ID   | Type id to identify the type when reading   |
|        | Value     | What should I say? The role is obvious here |

### Array

| Type   | Parameter    | Value | Role                                                 |
|--------|--------------|-------|------------------------------------------------------|
| `byte` | Type ID      | 7     | Type id of array                                     |
| `byte` | Array type   |       | Type id to identify the array type when reading      |
| `byte` | Item type    |       | Type id to identify the array item type when reading |
| `int`  | Length       |       | Number of items in the array                         |
|        | Values       |       | Values following each other in their type layout     |

### String

| Type   | Parameter | Value | Role                                           |
|--------|-----------|-------|------------------------------------------------|
| `byte` | Type ID   | 8     | Type id of string                              |
| `int`  | Length    |       | Number of bytes in the string                  |
|        | Value     |       | UTF-8 encoded bytes (not in the byte[] layout) |

### Object (Set, Dictionary)

| Type     | Parameter      | Value | Role                                               |
|----------|----------------|-------|----------------------------------------------------|
| `byte`   | Type ID        | 255   | Type id of object                                  |
| `string` | Type name      |       | Type name of the type to identify when reading     |
| `int`    | Property count |       | Number of properties the object has                |
|          | Properties     |       | Properties following each other in property layout |

### Property

| Type     | Parameter     | Role                                                       |
|----------|---------------|------------------------------------------------------------|
| `string` | Property name | Identify the property                                      |
| `byte`   | Type ID       | Type id to identify the property value type when reading   |
|          | Value         | Property values following each other in their type layout  |

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

### Method invocation

Value: `byte` 6

This command instucts the remote party to invoke a method and return its result.

| Type       | Parameter         | Role                                |
|------------|-------------------|-------------------------------------|
| `string`   | Method name       | Method name to invoke               |
| `object[]` | Method parameters | Parameters to pass to that method\* |

\* The `object[]` can contain any type of objects.

\* To pass a single array as the only parameter, it is needed to be added inside an `object[]`.
The `object[]`'s length will be 1 and its only item will be that array.

### Method invocation success

Value: `byte` 7

This command is sent when a method was successfully invoked.

| Type     | Parameter    | Role                                        |
|----------|--------------|---------------------------------------------|
| `object` | Return value | Optional value returned from the method\*\* |

\*\* The object can be any type depending on its type ID.

### Method invocation failure

Value: `byte` 8

This command is sent when a method could not be invoked.

| Type     | Parameter         | Role                                        |
|----------|-------------------|---------------------------------------------|
| `byte`   | Error code        | Error code (why couldn't be invoked)        |
| `string` | Error message     | Detailed information about what happened    |

### Error codes

| Value     | Meaning                                                              |
|-----------|----------------------------------------------------------------------|
| `byte` 0  | Success                                                              |
| `byte` 1  | Not connected. A connection must be requested and accepted first     |
| `byte` 2  | This happened or was initiated by the user                           |
| `byte` 3  | Not supported                                                        |
| `byte` 4  | An unknown command was received (see [commands](#commands))          |
| `byte` 5  | Unexpected command (the command is valid but unexpected)             |
| `byte` 6  | Unexpected parameter (the command is valid, but a parameter is not)  |
| `byte` 7  | No method was found to be invoked                                    |
| `byte` 8  | An exception occured in the invocation of the method                 |
| `byte` 9  | Cannot (de)serialize an object                                       |
| `byte` 10 | A method invocation or (de)serialization failed for security reasons |
