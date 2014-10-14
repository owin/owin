# OWIN Middlewares

**Version**
1.0.0-draft.1

**Author**
Sebastien Lambla (Caffeine IT)

**License**
[Creative Commons Attribution 3.0 Unported License][CCLicense]

**Last updated**
14 October 2014


## Abstract

The OWIN Specification defines a standard interface between .NET web servers
and web applications. This document specifies additional standard interfaces
for reusable OWIN middleware components.

## Status

This is a working draft.

This document is a submission to the  OWIN Working Group and is valid for a
period of six months and may be updated, replaced, or obsoleted by other
documents at any time. It is inappropriate to use working drafts as reference
material or to cite them other than as "work in progress"

This working draft will expire on INSERT DATE HERE ON PULL REQUEST.

## Copyright

Copyright (c) 2014 the persons identified as the document authors. The work
is licensed as per the [Creative Commons Attribution 3.0 Unported License]
[CCLicense] in effect on the date of publication of this document.

## Table of Content


1. [Introduction](#1-introduction)  
  1.1. [Requirements Notation](#1.1-requirements-notation)
2. [Definitions](#2-definitions)
3. [Middleware components](#3-middleware-components)  
  3.1. [Examples](#31-examples)  
  3.2. [Middleware Operation](#32-middleware-operation)  
  3.3. [Signature](#33-signature)  
  3.4. [Code Sample](#34-code-sample)
4. [Registration of middleware components](#4-Registration-of-middleware components)  
  4.1. [Signature](#41-signature)  
    4.1.1. [Registration sample](#411-registration-sample)  
  4.2. [Discovery of middlewares](#42-discovery-of-middlewares)  
    4.2.1. [Extension method sample](#421-extension-method-sample)  
5. [Versioning](#5-versioning)

## 1. Introduction

The [OWIN] specification was created to decouple web servers and web
applications on .NET platforms by creating a standardised abstract signature
that can be implemented in a vendor-neutral fashion.

[Layering][FieldingLayering] components conforming to a uniform interface
is a natural architectural style for HTTP-based application, and many
application frameworks provide such abstractions in the form of reusable
software components.

This specification provides a standardised signature for such components in the
OWIN ecosystem, hereby referred to as `middlewares`.

OWIN middlewares are defined in terms of a delegate structure building upon the
OWIN: Open Web Server Interface for .NET [Owin 1.0], with no
dependency on external libraries.

### 1.1 Requirements Notation

 The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119][rfc2119].

A component is deemed to be compliant with this specification if all absolute
requirements and prohibitions are conformed to.

## 2. Definitions

This document refers to the following terms:

 - **Application**: a process or operation inspecting, modifying or otherwise
   accessing HTTP messages, as an AppFunc signature.

 - **Middleware**: Pass through components that form a pipeline between a server
   and a terminating application to mediate request and response messages.

 - **Builder**: The extension point used by developers to add middleware
   components to a pipeline.

## 3. Middleware components

An OWIN application (as defined by an AppFunc) can be modelled as a set of
middleware components layered in a pipeline.

Each middleware receiving a request may perform some processing on the request,
may forward the request to the next middleware and return the resulting
response, or may decide to return a response immediately and not call the rest
of the chain.

In other words, a middleware functions as a proxy of sort, completely
encapsulating and hiding the next middleware in the chain, transparently to the
caller.

### 3.1. Examples

For example, a middleware responsible for logging all requests and responses
could execute as the first component of a pipeline, and log all requests and
responses as they pass through.

```
        request      > (log request)      > (process request)
SERVER =============== LOGGER ============= WEB APPLICATION
        response     < (log response)     < (return response)
```

Other types of middleware would respond immediately, without calling the next
component in the pipeline, such as for an authorization module.

```
        request      > (authenticate)
SERVER =============== LOGGER ------X------ WEB APPLICATION
        response     < (deny)
```

### 3.2. Middleware Operation

Middlewares are first chained together in a pipeline in a composition phase. The
result of middelware composition is an application (OWIN AppFunc).

A server as defined by OWIN will call the resulting application, unaware of the
composition of the pipeline it is calling.

At compose time, each middleware is passed a reference to the next middleware in
the chain, and should return an application.

At run time, middlewares can inspect and manipulate HTTP messages using the
usual environment dictionary, and in by doing so leverages existing knowledge in
building OWIN-compliant components.

### 3.3. Signature

```csharp
using MidFunc = Func<
                     AppFunc, // next middleware in the chain
                     AppFunc // the resulting application
                    >
```

### 3.4 Code Sample

The following example is a middleware logging the request and response headers.

```csharp
  // MidFunc
  public static AppFunc LogMiddleware(AppFunc nextMiddleware)
  {
    AppFunc composedApplication = environment => {

      // log the request headers
      Log.RequestHeaders(environment["owin.RequestHeaders"]);

      // pass the request to the next middleware
      next(environment);

      // log the response headers
      Log.RequestHeaders(environment["owin.ResponseHeaders"]);

    }

  return composedApplication;
  }
```

> **Note**
>
This specification does not preclude alternative modelings from being provided
by vendors, but they MUST provide a way to expose such alternate representations
as a MidFunc-compatible function.

## 4. Registration of middleware components

For the final application to be composed, a component need to be aware of all the middlewares in a system.

This specification defines a builder function that receives middlewares to compose, typically at the start of the application.

In order to provide for capability discovery, configuration and shared initialisation, the parameter to the builder is a function taking the startup properties as defined by the owin specification.


### 4.1. Signature


```csharp
using BuildFunc = Action<
        Func<
             IDictionary<string, object>, // startup properties
             MidFunc // Middleware to use
             >;
```

#### 4.1.1. Registration sample

The following example adds the `LogMiddleware` component defined earlier and registers it with the builder.

```csharp
  public static AppFunc LogMiddleware(AppFunc next) { /*  ... */ }
  public static void ConfigureWebApplication(BuildFunc builder)
  {
    builder(startupProperties => LogMiddleware);
  }
```

### 4.2. Discovery of middlewares

A common requirement for middleware authors is to enable users to discover middlewares after installing a software package in their solution.

In order to deliver this functionality, a middleware MUST implement an extension method on the builder function.

Integrated development environments tend to provide API discovery in the form of graphical navigators. As functions usually have a few other memers defined, and to allow easier navigation, a middleware SHOULD prefix the extension method with `Use`, and SHOULDN'T include the word middleware.

As an application is often composed of many middlewares, authors SHOULD return the BuildFunc in their extension method, to allow for easy chaining.

#### 4.2.1. Extension method sample

```csharp
  // definition
  public static BuildFunc UseHeaderLogging(this BuildFunc builder)
  {
    builder(settings => LogMiddleware);
  }

  // usage
  public static void ConfigureWebApplication(BuildFunc builder)
  {
    builder.UseHeaderLogging();
  }
}
```

## 5. Versioning

This document specifies version 1.0.0 of this specification.

todo: add link to semver, what major means and what minor revisions would entail, see discussions on the value of owin 1.0.1.

----
[FieldingLayering]: http://www.ics.uci.edu/~fielding/pubs/dissertation/net_arch_styles.htm#sec_3_4_2
[WGForum]: https://github.com/owin/owin.github.com/issues
[CCLicense]: http://creativecommons.org/licenses/by/3.0/deed.en_US
[CommonKeys]: http://owin.org/spec/CommonKeys.html
[sec-req-body]: #34-request-body-100-continue-and-completed-semantics
[sec-res-body]: #35-response-body
[sec-headers]: #33-headers
[sec-paths]: #53-paths
[sec-uri-scheme]: #51-uri-scheme
[sec-hostname]: #52-hostname
[sec-versioning]: #7-versioning
[semver]: http://semver.org/
[rfc2119]: http://www.ietf.org/rfc/rfc2119.txt
[rfc2616]: http://www.ietf.org/rfc/rfc2616.txt
[rfc3986]: http://www.ietf.org/rfc/rfc3986.txt
[rfc2616-42]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec4.html#sec4.2
[rfc2616-512]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html#sec5.1.2
[rfc2616-19611]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec19.html#sec19.6.1.1
[rfc2616-323]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.2.3
