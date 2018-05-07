# textstream-rpc
A simple RPC system.

## Introduction

A `textstream-rpc` session is a text stream consisting of lines of text. Each line has an operation, an object and a list of arguments. An example would like like this:

    FExample::Print:"Hello, World!"

This would call the **F**unction *Print* in the namespace *Example*, passing the string "Hello, World!" as an argument.

## Syntax

The first letter of a line is an operation. After that comes the namespace, C++-like two colons, an event name or a function name, and a list of arguments delimited by a colon (:). So if we were to take our previous example and split it up it would look like this:

          FExample::Print:"Hello, World!"
          ^    ^        ^           ^
   operation namespace function  argument

Namespace and function names mustn't begin with a digit and can only contain digits and ASCII letters.

## Data types

### Booleans

A boolean is a value that is represented by either `true` or `false`

Examples:

    true
    false

### Null

A `null` literal satisfies all types.

Examples:

    null

### Strings

A string must be quoted and properly escaped, like in C. It accepts the following escape sequences:

* \\a - alert (bell)
* \\b - backspace
* \\n - newline
* \\r - carriage return
* \\t - horizontal tab
* \\v - vertical tab
* \\\\ - literal backslash
* \\" - literal quote
* \\xHH - the eight-bit character whose walue is HH (hexadecimal)

Examples:

    "\xff\r\n"
    "Hello, World!"
    "Supports Unicode, so we can write with 漢字 if we want to"

### Numbers

Numbers by default are in the decimal system, however they can be prefixed with the following symbols to change to a different one:

* 0x - hexadecimal
* 0b - binary
* 0  - octal

The number may also include floating point values, in which case it can only be decimal.

Examples:

    0xffffff
    0b101011
    0222222222222
    23
    0.5

### Lists

Lists are always in square brackets ([]). Their values are seperated by a comma. A list can include any type described here, including another list.

Examples:

    [97, 98, 99]
    ["a", "b", "c"]
    [2, 3, 0xabc, "Hello, World!", [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]]

### Maps

Maps are denoted by curly braces ({}). Maps consist of key:value pairs, where both key and value can be of any type. The key:value pairs are delimited by a comma.

Examples:

    {"a": "d", "b": "e"}
    {[2, 3, 4]: 23, 2: 5, 0x23232: "tyvm"}

## Operations

There are 6 operations in textstream-rpc:

* [**O**K](#OK)
* [**V**ersion](#Version)
* [**E**rror](#Error)
* [**F**unction](#Function)
* [**R**eturn](#Return)
* [**S**ubscribe](#Subscribe)
* [**N**otify](#Notify)
* [**I**ntrospect](#Introspect)

### OK

An OK command is just that: a confirmation that the peer recieved the previous command. It's a special command: it doesn't have an object parameter.

Example:

    O

### Version

All streams must begin with a line containing this operation. All clients must emit it before any communication takes place. The initialization for textsteam-rpc V1.0 (the current one) is this:

    Vtextstream-rpc::V1

Optionally, arguments denoting compatible and supported versions may be added, like this:

    Vtextstream-rpc::V2:["textstream-rpc::V1"]
    Vtextstream-rpc::V3:["textstream-rpc::V2"]:["textstream-rpc::V1"]

### Error

Error is used to notify the other peer about an error. The operation takes an error as its object. Here are the builtin errors:

* `builtin::notfound
* `builtin::sigdoesnotmatch

Custom errors are allowed.

Examples:

    Ebuiltin::notfound
    Ehttp::err404


### Function

Function takes the function and its namespace as its object, and passes it the arguments. A function has a certain signature. If the signature does not match, it will respond with the error `builtin::sigdoesnotmatch`. If the function is not found, it will respond with the error `builtin::notfound`. 

Examples:

    FExampleNamespace::ExampleFunction:2:3:4:5
    FExampleNamespace::ComplicatedFunction:{2:3}:null:true:false:"nice\x12":"lol no generics"

### Return

Return returns the functions return value. It uses the function name as an object and the return value as an argument.

Examples:

    RExampleNamespace::Returns5:5
    RExampleNamespace::BruteforceHelloWorld:"Hello, World!"

### Subscribe

Subscribe subscribes to the event specified by its object.

Example:

    SExampleNamespace::ExampleEvent

### Notify

Notify notifies about a subscribed event, together with any params the event might have.

Example (following the previous one):

    NExampleNamespace::ExampleEvent:5:0x23232

### Introspect

Introspect includes some functions that help introspect variables. Here are they:

* all::GetNamespaces - prints a list literal of strings of namespaces
* $NAMESPACE::GetFuncs - prints a list literal of functions in namespace $NAMESPACE
* $NAMESPACE::GetEvents - prints a list literal of events in namespace $NAMESPACE
* $NAMESPACE::GetFuncSig:string - prints the function signature - like so:

    ExampleNamespace::ExampleFunction:string:number:bool:list:map :: type

* $NAMESPACE::GetEventSig:string - same as the previous one, but specifies the params an event can pass instead.

# An example:

    Client: Vtextstream-rpc::V1.0
    Server: Vtextstream-rpc::V1.0
    Client: O
    Server: O
    Client: Iall::GetNamespaces
    Server: ["ExampleServerAPI", "ExampleServerInternalAPI"]
    Server: O
    Client: O
    (author's note: from here on the OK acknowledgements will be omitted)
    Client: IExampleServerAPI::GetFuncs
    Server: ["Print"]
    Client: IExampleServerAPI::GetFuncSig:"Print"
    Server: ExampleServerAPI::Print:string :: bool
    Client: FExampleServerAPI::Print:"\a"
    Server: RExampleServerAPI::Print:true
    Client: IExampleServerAPI::GetEvents
    Server: ["HTTPGetEvent"]
    Client: IExampleServerAPI::GetEventSig
    Server: ExampleServerAPI::HTTPGetEvent:string
    Client: SExampleServerAPI::HTTPGetEvent
    (after a while)
    Server: NExampleServerAPI::HTTPGetEvent:"/bin"
    (showing the acknowledgements here)
    Client: O
