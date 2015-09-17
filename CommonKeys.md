# OWIN Common Keys

Author: OWIN Working Group.

Last updated: 12 March 2015


## Contents

1. [Overview][1]
2. [Key usage guidelines][2]
3. [Naming conventions][3]
4. [Value conventions][4]
5. [Capabilities announcement and detection][5]
6. [Common keys][6]


## 1.  Overview

This document contains guidelines for adding functionality to OWIN based servers or application via additional keys in the IDictionary collections (e.g. the request environment or the startup properties).  This includes naming conventions, value conventions, and a list of know common keys and their semantics.

These guidelines and keys are provided independently of the OWIN specification as the list of common keys below is expected to grow independently of the OWIN standard.

The following guidelines apply to the keys and values specifically listed below, as well as for custom keys and values defined by individual implementations.


## 2\. Key usage guidelines

The IDictionary collections can be used to extend the OWIN interface with additional functionality. 

* All keys MUST be compared using StringComparer.Ordinal.

* All keys not defined in the OWIN standard are considered extension keys and are strictly optional.

* Implementers SHOULD clearly document what keys their component will populate, under what conditions, and the types and semantics of the associated values.


## 3\. Naming conventions

The IDictionary collections are open and mutable for storing arbitrary data.  To avoid collisions between components, the following key naming conventions SHOULD be honored.

* Key names are Ordinal (case sensitive, culture insensitive) strings. 

* Most keys SHOULD be composed of a lower case prefix, a period, and a Pascal cased descriptor (e.g. owin.RequestMethod).

* Keys defined directly in the OWIN standard use the owin.* prefix.  Additional keys with this prefix MUST NOT be defined except by future versions of the OWIN standard.

* A number of common keys are listed later in this document.  These keys use general prefixes such as server.* to indicate that this is data that may be provided by many different implementations.  Implementers SHOULD favor using such shared keys rather than defining their own.

* Keys that refer to a common technology group SHOULD be prefixed with a name that identifies that technology. E.g. ssl.* or websocket.*.

* Keys that expose functionality specific to a single implementation SHOULD use a prefix that clearly identifies that implementation.  E.g. iis.*, httplistener.*, webapi.*, mymiddleware.*.

* Implementers SHOULD avoid defining keys that differ only subtly from existing keys (e.g. casing, spelling, etc.).

* Implementers SHOULD include a technology specific version key that clearly identifies not only the version of the underlying component, but also the version of any OWIN specific wrapper of that component.  E.g. mshttplistener.Version: .NET 4.0, OWIN wrapper 1.0.1.  This is intended to help consumers cross reference documentation with implementations, facilitating both development and debugging.
  * Where the version includes the .NET framework version, specify the version compiled against, not the version currently running.  E.g. if the component targeted .NET 4.0 and is running on .NET 4.5, specify 4.0.  The framework version currently running is readily available in System.Environment.Version.

* The fully qualified name of a type MAY be used as the key, but only if it is reasonable to assume that there will only ever be one instance of this type per request.  E.g. System.Net.HttpListenerContext is guaranteed to be unique per request, whereas System.Security.Cryptography.X509Certificates.X509Certificate identifies a specific type but it is ambiguous if the value represents the client certificate, the server certificate, or a certificate for some other purpose.


## 4\. Value conventions

* The type of each value SHOULD be clearly documented by the implementer.

* Values MAY be of a type derived from the type specified in documentation, but the documentation SHOULD state under what conditions it is reasonable to expect such types to be available.  Consumers may attempt to cast to more specific derived types (safely using is or as rather than an explicit cast), but SHOULD fall back to the documented base type wherever possible.

* Null values or empty strings SHOULD be treated the same as if the key were not present. Every key SHOULD have at least one defined value, even if the value is the string "true".

* Implementers SHOULD avoid populating keys associated with null values or empty strings, but SHOULD be tolerant of such values when retrieving data.

* Implementers MUST NOT overload a single key with multiple types (other than derived types as discussed above).  If there are multiple possible representations, use separate keys to clearly identify them.

* Avoid using types that are likely to change signatures between versions. 

* Prefer base types (e.g. int, string, etc.) or common types in the .NET framework that have existed for multiple releases (e.g. Stream).

* Values in the request Environment collection are assumed to be alive and accurate only for the lifetime of the request.  Consumers SHOULD NOT retain or attempt to use values after a request has completed or failed.  Where a value advertises a specific capability, that capability is only assumed to be available for that request instance and SHOULD be re-verified on future requests.
  * Global capabilities or data SHOULD be communicated as described in section 5 below.

* Where any specific value cleanup is required (e.g. Dispose), such cleanup is the responsibility of the component that originally added the value.  This should be taken care of as the stack unwinds after a request has completed or failed.


## 5. Capabilities announcement and detection

It is important for applications to be able to determine if a specific feature is supported by the current server or middleware.  The following pattern is recommended for announcing and detecting feature/extension support.

* At startup the server SHOULD create a server.Capabilities IDictionary in the startup Properties dictionary.

* This capabilities dictionary is for static capability details that do not change on a per-request basis.

* The same instance of the capabilities dictionary SHOULD also be included in each request environment dictionary.

* Each extension SHOULD add to the capabilities IDictionary a featurename.Version key with the associated string value of the latest version of that extension supported (e.g. 1.2).

* If extensions have defined additional keys used to indicate details, the implementer SHOULD add to the capabilities IDicitonary a featurename.Support IDictionary containing the detailed featurename.FeatureDetail keys and their associated values.

* Per-request feature keys and values should be included directly in the environment dictionary.


## 6\. Common keys

Keys and values listed here are those that are anticipated to be common across multiple implementations, but are not strictly required for basic operations.

| Key                     | Type                                 | Startup | Shutdown | Request | Description |
| ----------------------- | ------------------------------------ |:-------:|:--------:|:-------:| ----------- |
| ssl.ClientCertificate   | `X509Certificate`                    |         |          | X       | The client certificate provided during HTTPS SSL negotiation. |
| server.RemoteIpAddress  | `String`                             |         |          | X       | The IP Address of the remote client. E.g. 192.168.1.1 or ::1 |
| server.RemotePort       | `String`                             |         |          | X       | The port of the remote client. E.g. 1234 |
| server.LocalIpAddress   | `String`                             |         |          | X       | The local IP Address the request was received on. E.g. 127.0.0.1 or ::1 |
| server.LocalPort        | `String`                             |         |          | X       | The port the request was received on. E.g. 80 |
| server.IsLocal          | `Boolean`                            |         |          | X       | Was the request sent from the same machine? E.g. true or false. |
| host.TraceOutput        | `TextWriter`                         | X       |          | X       | A tracing output that may be provided by the host. |
| host.Addresses          | `IList<IDictionary<string, object>>` | X       |          |         | A list of per-address server configuration.  The following keys are defined with string values: scheme, host, port, path. |
| server.Capabilities     | `IDictionary<string, object>`        | X       |          | X       | Global capabilities that do not change on a per-request basis.  See Section 5 above. |
| server.OnSendingHeaders | `Action<Action<object>, object>`     |         |          | X       | Allows the caller to register an Action callback that fires as a last chance to modify response headers, status code, reason phrase, or protocol. The object parameter is an optional state object that will passed to the callback. |
| server.OnInit           | `Action<Func<Task>>`                 | X       |          |         | An `Action<Func<Task>>` that allows middleware to register a callback that the server will call once when initializing. |
| server.OnDispose        | `CancellationToken`                  |         | X        |         | A `CancellationToken` that represents when the server is disposing. |
| websocket.*             |                                      | X       |          | X       | See the WebSocket extension. |
| sendfile.*              |                                      | X       |          | X       | See the SendFile extension. |
| opaque.*                |                                      | X       |          | X       | See the Opaque extension. |
 

[1]: #Overview
[2]: #_2._Key_usage
[3]: #_3._Naming_conventions
[4]: #_4._Value_conventions
[5]: #_5._Capabilities_announcement
[6]: #_6._Common_keys
