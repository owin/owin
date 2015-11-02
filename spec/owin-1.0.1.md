# OWIN: Open Web Server Interface for .NET

**Version**  
1.0.1-draft  
**Author**  
[OWIN working group][WGForum]  
**Copyright**  
OWIN contributors  
**License**  
[Creative Commons Attribution 3.0 Unported License][CCLicense]  
**Last updated**  
24 October 2014  

---

1. [Overview](#1-overview)
2. [Definitions](#2-definitions)
3. [Request Execution](#3-request-execution)
  1. [Application Delegate](#3-1-application-delegate)
  2. [Environment](#3-2-environment)
  3. [Headers][sec-headers]
  4. [Request Body][sec-req-body]
  5. [Response Body][sec-res-body]
  6. [Request Lifetime](#3-6-request-lifetime)
4. [Application Startup](#4-application-startup)
5. [URI Reconstruction](#5-uri-reconstruction)
  1. [URI Scheme][sec-uri-scheme]
  2. [Hostname][sec-hostname]
  3. [Paths][sec-paths]
  4. [URI Reconstruction Algorithm](#5-4-uri-reconstruction-algorithm)
  5. [Percent-encoding](#5-5-percent-encoding)
6. [Error Handling](#6-error-handling)
  1. [Application Errors](#6-1-application-errors)
  2. [Server Errors](#6-2-server-errors)
7. [Versioning][sec-versioning]

---

## 1. Overview

This document defines OWIN, a standard interface between .NET web servers and web applications. The goal of OWIN is to decouple server and application and, by being an open standard, stimulate the open source ecosystem of .NET web development tools.

OWIN is defined in terms of a delegate structure. There is no assembly called `OWIN.dll` or similar. Implementing either the host or application side the OWIN spec does not introduce a dependency to a project.

In this document, the C# `Action`/`Func` syntax is used to notate some delegate structures. However, the delegate structure could be equivalently represented with F# native functions, CLR interfaces, or named delegates. This is by design; when implementing OWIN, choose a delegate representation that works for you and your stack.

> The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][rfc2119].

**Normative sections are highlighted like the key words paragraph above.**

## 2. Definitions

This document refers to the following software actors:

1. **Server** &mdash; The HTTP server that directly communicates with the client and then uses OWIN semantics to process requests.  Servers may require an adapter layer that converts to OWIN semantics.

2. **Web Framework** &mdash; A self-contained component on top of OWIN exposing its own object model or API that applications may use to facilitate request processing. Web Frameworks may require an adapter layer that converts from OWIN semantics.

3. **Web Application** &mdash; A specific application, possibly built on top of a Web Framework, which is run using OWIN compatible Servers.

4. **Middleware** &mdash; Pass through components that form a pipeline between a server and application to inspect, route, or modify request and response messages for a specific purpose.

5. **Host** &mdash; The process an application and server execute inside of, primarily responsible for application startup. Some Servers are also Hosts.

## 3. Request Execution

Broadly speaking, a server invokes an application (providing as arguments an environment dictionary with request and response headers and bodies); an application either populates a response or indicates an error.

### 3.1. Application Delegate

The primary interface in OWIN is called the application delegate or `AppFunc`. An application delegate takes the `IDictionary<string, object>` environment and returns a `Task` when it has finished processing.

```c#
using AppFunc = Func<
    IDictionary<string, object>, // Environment
    Task>; // Done
```

> The application MUST eventually complete the returned Task, or throw an exception.

### 3.2. Environment

The Environment dictionary stores information about the request, the response, and any relevant server state. The server is responsible for providing body streams and header collections for both the request and response in the initial call. The application then populates the appropriate fields with response data, writes the response body, and returns when done.

> * The environment dictionary MUST be non-null, mutable and MUST contain the keys listed as required in the tables below.
 
> * Keys MUST be compared using `StringComparer.Ordinal`.

> * The values associated with the keys MUST be non-null, unless otherwise specified.

In addition to these keys, the host, server, middleware, application, etc. may add arbitrary data associated with the request or response to the environment dictionary

 Guidelines for additional keys and a list of commonly defined keys can be found in the [CommonKeys addendum][CommonKeys] to this spec.

### 3.2.1 Request Data

| Required | Key Name                  | Value Description |
|----------|---------------------------|-------------------|
| **Yes**  | `owin.RequestBody`        | A `Stream` with the request body, if any. `Stream.Null` MAY be used as a placeholder if there is no request body. See [Request Body](sec-req-body). |
| **Yes**  | `owin.RequestHeaders`     | An `IDictionary<string, string[]>` of request headers. See [Headers][sec-headers]. |
| **Yes**  | `owin.RequestMethod`      | A `string` containing the HTTP request method of the request (e.g., `"GET"`, `"POST"`). |
| **Yes**  | `owin.RequestPath`        | A `string` containing the request path. The path MUST be relative to the "root" of the application delegate. See [Paths][sec-paths]. |
| **Yes**  | `owin.RequestPathBase`    | A `string` containing the portion of the request path corresponding to the "root" of the application delegate; see [Paths][sec-paths]. |
| **Yes**  | `owin.RequestProtocol`    | A `string` containing the protocol name and version (e.g. `"HTTP/1.0"` or `"HTTP/1.1"`). |
| **Yes**  | `owin.RequestQueryString` | A `string` containing the query string component of the HTTP request URI, without the leading "?" (e.g., `"foo=bar&amp;baz=quux"`). The value may be an empty string. |
| **Yes**  | `owin.RequestScheme`      | A `string` containing the URI scheme used for the request (e.g., `"http"`, `"https"`); see [URI Scheme][sec-uri-scheme]. |
| **No**   | `owin.RequestId `         | An optional `string` that uniquely identifies a request. The value is opaque and SHOULD have some level of uniqueness. A Host MAY specify this value. If it is not specified, middleware MAY set it. Once set, it SHOULD NOT be modified. |
| **No** | `owin.RequestUser`          | An optional identity that represents the user associated with a request. The identity MUST be a `ClaimsPrincipal`. Middleware MAY specify this value. If it is not specified, middleware MAY set it. Once set, it MAY BE modified. |

### 3.2.2 Response Data

| Required | Key Name                  | Value Description |
|----------|---------------------------|-------------------|
| **Yes**  | `owin.ResponseBody`       | A `Stream` used to write out the response body, if any. See [Response Body][sec-res-body]. |
| **Yes**  | `owin.ResponseHeaders`    | An `IDictionary<string, string[]>` of response headers. See [Headers][sec-headers]. |
| **No**   | `owin.ResponseStatusCode` | An optional `int` containing the HTTP response status code as defined in [RFC 2616][rfc2616] section 6.1.1. The default is 200. |
| **No**  | `owin.ResponseReasonPhrase`| An optional `string` containing the reason phrase associated the given status code. If none is provided then the server SHOULD provide a default as described in [RFC 2616][rfc2616] section 6.1.1 |
| **No**  | `owin.ResponseProtocol`    | An optional `string` containing the protocol name and version (e.g. `"HTTP/1.0"` or `"HTTP/1.1"`). If none is provided then the `"owin.RequestProtocol"` key's value is the default. |

### 3.2.3 Other Data

| Required | Key Name                  | Value Description |
|----------|---------------------------|-------------------|
| **Yes**  | `owin.CallCancelled`      | A `CancellationToken` indicating if the request has been canceled/aborted. See [Request Lifetime][sec-req-lifetime]. |
| **Yes**  | `owin.Version`            | A `string` indicating the OWIN version. See [Versioning][sec-versioning]. |

### 3.3. Headers

The headers of HTTP request and response messages are represented by objects of type `IDictionary<string, string[]>`. The requirements below are predicated on [RFC 2616 section 4.2][rfc2616-42].

> * The dictionary MUST be mutable.

> * Keys MUST be HTTP field-names without `':'` or whitespace.

> * Keys MUST be compared using `StringComparer.OrdinalIgnoreCase`.

> * All characters in key and value strings SHOULD be within the ASCII codepage.

> * The value array returned is assumed to be a copy of the data. Any intended changes to the value array MUST be persisted back to the headers dictionary manually by via `headers[headerName] = modifiedArray;` or `headers.Remove(header)`.

> * Header values are assumed to be in a mixed format, meaning that a normally comma separated header may appear as a single entry in the values array, one entry per value, or a mixture of the two.
> (e.g. `new string[1] {"value, value, value"}`, `new string[3] {"value", "value", "value"}`, or `new string[2] {"value, value", "value"}`)


> * Servers, applications, and intermediaries SHOULD NOT split or merge header values unnecessarily. While the three formats are supposed to be interchangeable, in practice many existing implementations only support one specific format. Developers should have the flexibility to support existing implementations by producing or consuming a selected format without interference.

### 3.4. Request body, 100 Continue, and Completed Semantics

If the request indicates there is an associated body the server SHOULD provide a `Stream` in the `"owin.RequestBody"` key to access the body data. `Stream.Null` MAY be used as a placeholder if there is no request body data expected. If the request Expect header indicates the client requests a `100 Continue`, it is up to the server to provide this. The application MUST NOT set `"owin.ResponseStatusCode"` to `100`. `100 Continue` is only an intermediate response and using it would prevent the application from providing a final response (e.g. `200 OK`). The server SHOULD send the `100 Continue` on behalf of the application if the application starts reading from the stream before data starts arriving.

> * The application delegate SHOULD NOT complete its returned `Task` and return control to the server until it is finished with the request body. Once the `AppFunc` `Task` is complete the application SHOULD NOT continue to read from the request stream.

> * The application MUST signal completion or failure of the response body by completing its returned `Task` or throwing an exception. After completing the `Task`, the application SHOULD NOT write any further data to the stream.

> * If the server signals the `"owin.CallCancelled"` `CancellationToken` during the execution of the application delegate, the application SHOULD NOT attempt further reads from the stream, and SHOULD promptly complete the application delegate `Task`.

> * The application SHOULD NOT close or dispose the given stream unless it has completely consumed the request body. The stream owner (e.g. the server or middleware) MUST do any necessary cleanup once the application delegate's Task completes.

> * Any exceptions thrown from the request body stream are fatal and SHOULD be returned to the server by being thrown synchronously from the `AppFunc` or by failing the asynchronous `Task` with the given exception(s).

### 3.5. Response Body

The server provides a response body `Stream` with the `"owin.ResponseBody"` key in the initial environment dictionary. The headers, status code, reason phrase, etc., can be modified up until the first write to the response body stream. Upon first write, the server validates and sends the headers. Applications MAY choose to buffer response data to delay the header finalization.

> * The application MUST signal completion or failure of the body by completing its returned `Task` or throwing an exception. After completing the `Task`, the application SHOULD NOT write any further data to the stream.

> * If the server signals the `"owin.CallCancelled"` `CancellationToken` during the execution of the application delegate, the application SHOULD NOT attempt further writes to the stream, and SHOULD promptly complete the application delegate `Task`.

> * The application SHOULD NOT close or dispose the given stream as middleware may append additional data. The stream owner (e.g. the server or middleware) MUST perform any necessary cleanup once the application delegate's `Task` completes.

An application SHOULD NOT assume the given stream supports multiple outstanding asynchronous writes. The application developer SHOULD verify that the server and all middleware in use support this pattern before attempting to use it.

### 3.6. Request lifetime

The full scope or lifetime of a request is limited by the several factors, including the client, server, and application delegate. In the simplest scenario a request's lifetime ends when the application delegate has completed and the server gracefully ends the request. Failures at any level may cause the request to terminate prematurely, or may be handled internally and allow the request to continue. 

The `"owin.CallCancelled"` key is associated with a `CancellationToken` that the server uses to signal if the request has been aborted. This SHOULD be triggered if the request becomes faulted before the `AppFunc` `Task` is completed. It MAY be triggered at any point at the providers discretion. Middleware MAY replace this token with their own to provide added granularity or functionality, but they SHOULD chain their new token with the one originally provided to them.

## 4. Application Startup

When the host process starts there are a number of steps it goes through to set up the application. 

1. The host creates a Properties `IDictionary<string, object>` and populates any startup data or capabilities provided by the host.

2. The host selects which server will be used and provides it with the Properties collection so it can similarly announce any capabilities.

3. The host locates the application setup code and invokes it with the Properties collection.

4. The application reads and/or sets configuration in the Properties collection, constructs the desired request processing pipeline, and returns the resulting application delegate.

5. The host invokes the server startup code with the given application delegate and the Properties dictionary. The server finishes configuring itself, starts accepting requests, and invokes the application delegate to process those requests.

The Properties dictionary may be used to read or set any configuration parameters supported by the host, server, middleware, or application. 

> * The startup properties dictionary MUST be non-null, mutable and MUST contain the keys listed as required in the table below.

> * Keys MUST be compared using StringComparer.Ordinal.

> * The values associated with the keys MUST be non-null, unless otherwise specified.

> | Required | Key Name       | Value Description |
|----------|----------------|-------------------|
| **Yes**  | `owin.Version` | A `string` indicating the OWIN version. Added by the server in step 2 above. See [Versioning][sec-versioning]. |

In addition to these keys the host, server, middleware, application, etc. may add arbitrary data associated with the application configuration to the properties dictionary. Guidelines for additional keys and a list of commonly defined keys can be found in the [CommonKeys addendum][CommonKeys] to this spec.

## 5. URI Reconstruction

Applications often require the ability to reconstruct the complete URI of a request. This process cannot be perfect since HTTP clients do not usually transmit the complete URI which they are requesting, but OWIN makes provisions for the purpose of reconstructing the approximate URI of a request.

### 5.1. URI Scheme

This information is usually not transmitted by an HTTP client and depending on network configuration it may not be possible for an OWIN server to determine a correct value. In these cases, the server may have to manually configure or compute a value.

> Servers MUST provide a best-guess value for `"owin.RequestScheme"`.

### 5.2. Hostname

In the context of an HTTP/1.1 request, the name of the server to which the client is making a request is usually indicated in the Host header field-value of the request, although it might be specified using an absolute Request-URI (see RFC 2616, sections [5.1.2][rfc2616-512], [19.6.1.1][rfc2616-19611]).

> A server MUST provide a value for the `"Host"` key in the request header dictionary. The format of the value MUST be `"<hostname>[:<port>]"`. The value SHOULD be deduced by the host using the following steps:
> 
> 1. If the Request-URI of the incoming request is an absolute URI, the value of the `"Host"` key MUST be taken from the host part of the absolute URI.
> 2. If the Request-URI of the incoming request is not an absolute URI, the value of the `"Host"` key MUST be taken from the Host header field-value of the incoming request.
> 3. If the Host header is not present in the incoming request (as in an HTTP/1.0 request), or if its value consists entirely of linear whitespace, the server MUST provide a sensible best-guess value for the `"Host"` key.

### 5.3. Paths

Servers may have the ability to map application delegates to some base path. For example, a server might have an application delegate configured to respond to requests beginning with `"/my-app"`, in which case it should set the value of `"owin.RequestPathBase"` in the environment dictionary to `"/my-app"`. If this server receives a request for `"/my-app/foo"`, the `"owin.RequestPath"` value of the environment dictionary provided to the application configured to respond at `"/my-app"` should be `"/foo"`.

> * The value associated with the environment key `"owin.RequestPathBase"` MUST NOT end with a slash and MUST either start with a slash or be `String.Empty`.

> * The value associated with the environment key `"owin.RequestPath"` MUST start with a slash, or MAY be `String.Empty` if the value associated with `"owin.RequestPathBase"` is not `String.Empty`.

### 5.4. URI Reconstruction Algorithm

The following algorithm can be used to approximate the complete URI of the current request:

```c#
var uri =
   (string)Environment["owin.RequestScheme"] + 
   "://" +
   Headers["Host"].First() +
   (string)Environment["owin.RequestPathBase"] +
   (string)Environment["owin.RequestPath"];

if(Environment["owin.RequestQueryString"] != "")
{
   uri += "?" + (string)Environment["owin.RequestQueryString"];
}
```

The result of this algorithm may not be identical to the URI the client used to make the request; for example, the server may have done some rewriting to canonicalize the request. Further, it is subject to the caveats described in the [URI Scheme][sec-uri-scheme] and [Hostname][sec-hostname] sections above.

### 5.5. Percent-encoding

A URI uses percent encoding to transport characters outside its normally allowed character rules ([RFC 3986][rfc3986] section 2). Percent-encoding is used to express the underlying octets present in a URI component, whose octets are interpreted via UTF-8 encoding. Most web server implementations will perform percent-decoding on paths in order to perform request routing (as they rightly may, see: RFC 2616 section [5.1.2][rfc2616-512], also [3.2.3][rfc2616-323]), and OWIN follows this precedent. The request query string in OWIN is presented in its percent-encoded form; a percent-decoded query string could contain `'?'` or `'='` characters, which would render the string unparseable.

> * The server MUST provide percent-decoded values for `"owin.RequestPath"` and `"owin.RequestPathBase"`.

> * The server MUST provide a percent-encoded value for `"owin.RequestQueryString"`

## 6. Error Handling

While there are standard exceptions such as `ArgumentException` and `IOException` that may be expected in normal request processing scenarios, handling only such exceptions is insufficient when building a robust server or application. If a server wishes to be robust it SHOULD consistently address all exception types thrown or returned from the application delegate or body delegate. The handling mechanism (e.g. logging, crashing and restarting, swallowing, etc.) is up to the server and host process.

### 6.1. Application Errors

An application may generate an exception in the following places:

* Thrown from an invocation of the application delegate.
* Provided as the result of the application delegate `Task`.

An application SHOULD attempt to trap its own internal errors and generate an appropriate (possibly 500-level) response rather than propagating an exception up to the server.

After an application provides a response, the server SHOULD wait to receive at least one write from the response `Stream` before writing the response headers to the underlying transport. In this way, if instead of a write to the `Stream` the server gets an exception from the application delegate `Task`, the server will still be able to generate a 500-level response. If the server gets a write to the `Stream` first, it can safely assume that the application has caught as many of its internal errors as possible; the server can begin sending the response. If an exception is subsequently received, the server MAY handle it as it sees fit (e.g. logging, write a textual description of the error to the underlying transport, and/or close the connection).

### 6.2. Server Errors

When a server encounters errors during a request's lifetime it SHOULD signal the `CancellationToken` it provided in `"owin.CallCancelled"`. The server MAY then take any necessary actions to terminate the request, but it SHOULD be tolerant of completion delays of the application delegate.

## 7. Versioning

Future updates to this standard may contain breaking changes (e.g. signature changes, key additions or modifications, etc.) or non-breaking additions. While addressing specific changes is the responsibility of later versions of the standard, here are initial guidelines for expected changes:

* This standard uses [Semantic Versioning][semver] (e.g. `Major.Minor.Patch`).

* Breaking changes to the API signatures or existing keys will require incrementing the major version number (e.g. OWIN 2.0)

* Adding new keys or delegates in a backwards compatible way only requires incrementing the minor version number (e.g. OWIN 1.1).

* Making corrections and clarifications to the document alone only requires incrementing the patch version number and last modified date (e.g. OWIN 1.0.1).

* The `"owin.Version"` key in the startup Properties and request Environment dictionaries indicates the latest version of the standard implemented by the server and may be used to dynamically adjust behaviors of applications.

* All implementers SHOULD clearly document the full version number(s) of the OWIN standard they support.

* The keys listed in the [CommonKeys addendum][CommonKeys] to this spec are strictly optional. Additions may be made there without directly affecting the OWIN standard or version number.

----

[WGForum]: https://github.com/owin/owin.github.com/issues
[CCLicense]: http://creativecommons.org/licenses/by/3.0/deed.en_US
[CommonKeys]: http://owin.org/spec/CommonKeys.html
[sec-headers]: #3-3-headers
[sec-req-body]: #3-4-request-body-100-continue-and-completed-semantics
[sec-res-body]: #3-5-response-body
[sec-paths]: #5-3-paths
[sec-uri-scheme]: #5-1-uri-scheme
[sec-hostname]: #5-2-hostname
[sec-versioning]: #7-versioning
[semver]: http://semver.org/
[rfc2119]: http://www.ietf.org/rfc/rfc2119.txt
[rfc2616]: http://www.ietf.org/rfc/rfc2616.txt
[rfc3986]: http://www.ietf.org/rfc/rfc3986.txt
[rfc2616-42]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec4.html#sec4.2
[rfc2616-512]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html#sec5.1.2
[rfc2616-19611]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec19.html#sec19.6.1.1
[rfc2616-323]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.2.3
