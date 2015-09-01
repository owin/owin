# OWIN Middlewares

**Version**
1.0.0-draft.1

**Authors**
Sebastien Lambla (Caffeine IT)

**License**
[Creative Commons Attribution 3.0 Unported License][CCLicense]

**Last updated**
1 October 2015

## Abstract

The [OWIN Specification][owin-spec] defines a standard interface between .NET web servers
and web applications. This document specifies additional standard interfaces
for reusable OWIN Middleware components.

## Status

This is a working draft.

This document is a submission to the OWIN Working Group and is valid for a
period of six months and may be updated, replaced, or obsoleted by other
documents at any time. It is inappropriate to use working drafts as reference
material or to cite them other than as "work in progress"

This working draft will expire on 1 March 2016.

## Copyright

Copyright (c) 2014 the persons identified as the document authors. The work
is licensed as per the
[Creative Commons Attribution 3.0 Unported License][CCLicense] in effect on the
date of publication of this document.

## Table of Content

1. [Introduction](#1-introduction)  
  1.1. [Requirements Notation](#11-requirements-notation)
2. [Definitions](#2-definitions)
3. [Middleware](#3-middleware)  
  3.1. [Examples](#31-examples)  
  3.2. [Middleware Operation](#32-middleware-operation)  
  3.3. [Signature](#33-signature)  
  3.4. [Code Sample](#34-code-sample)
4. [Middleware Registration](#4-middleware-registration)  
  4.1. [Signature](#41-signature)  
    4.1.1. [Registration Sample](#411-registration-sample)  
  4.2. [Discovery and Registration of Middleware](#42-discovery-and-registration-of-middleware)  
    4.2.1. [Extension Method Sample](#421-extension-method-sample)  
5. [Versioning](#5-versioning)
6. [Acknowledgements](#6-acknowledgements)

## 1. Introduction

The [OWIN] specification was created to decouple web servers and web
applications on .NET platforms by creating a standardised abstract signature
that can be implemented by Applications and Servers in a vendor-neutral fashion.

[Layering][FieldingLayering] components conforming to a uniform interface
is a natural architectural style for HTTP-based application, and many
application frameworks provide such abstractions in the form of reusable
software components.

This specification provides a standardised signature for such components in the
OWIN ecosystem, hereby referred to as **Middleware**.

OWIN Middleware is defined in terms of a delegate structure building upon the
OWIN: Open Web Server Interface for .NET [Owin 1.0], with no
dependency on external libraries.

### 1.1. Requirements Notation

 The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119][rfc2119].

A component is deemed to be compliant with this specification if all absolute
requirements and prohibitions are conformed to.

Components and terms defined in this and other specifications are always
capitalized, so as to clarify between a general term and the term as used in a
specification.

## 2. Definitions

This document refers to the following terms:

 - **Application**: a process or operation inspecting, modifying or otherwise
   accessing HTTP messages, as an AppFunc signature.

 - **Middleware**: Pass through components, capable of mediating request and
   response messages, in a pipeline model which, when
   composed, forms an Application.

 - **Builder**: The extension point lambda used by developers to add Middleware
   components to a pipeline.

 - **Server**: xxx

## 3. Middleware

An OWIN Application (as defined by an AppFunc) can be modelled as
Middleware components layered in a pipeline.

Each Middleware receiving a request may perform some processing on the request,
may forward the request to the next Middleware and return the resulting
response, or may decide to return a response immediately and not call the rest
of the chain.

In other words, a Middleware functions as a proxy of sort, completely
encapsulating and hiding the next Middleware in the chain, transparently to the
caller.

### 3.1. Examples

For example, a Middleware responsible for logging all requests and responses
could execute as the first component of a pipeline, and log all requests and
responses as they pass through.

```
        request      > (log request)      > (process request)
SERVER =============== LOGGER ============= WEB APPLICATION
        response     < (log response)     < (return response)
```

Other types of Middleware would respond immediately, without calling the next
component in the pipeline, such as for an authorization module denying a request
before a web application can respond.

```
        request      > (authenticate)
SERVER =============== LOGGER ------X------ WEB APPLICATION
        response     < (deny)
```

### 3.2. Middleware Operation

The whole set of Middleware composing an Application are first chained together
in a pipeline in a composition phase. The result of Middleware composition is an
Application (OWIN AppFunc).

A Server as defined by OWIN will call the resulting Application, unaware of the
composition of the pipeline it is calling.

At compose time, each Middleware is passed a reference to the next Middleware in
the chain, and should return an Application.

At run time, a Middleware can inspect and manipulate HTTP messages using the
usual environment dictionary, and in by doing so leverages existing knowledge in
building OWIN-compliant components.

### 3.3. Signature

```csharp
using MidFunc = Func<
                     AppFunc, // next Middleware in the chain
                     AppFunc // returns the resulting application
                    >
```

### 3.4. Code Sample

The following example is a Middleware logging the request and response headers.

```csharp
// MidFunc
public static AppFunc LogMiddleware(AppFunc nextMiddleware)
{
  AppFunc composedApplication = async environment => {

    // log the request headers
    Log.RequestHeaders(environment["owin.RequestHeaders"]);

    // pass the request to the next Middleware
    await next(environment);

    // log the response headers
    Log.RequestHeaders(environment["owin.ResponseHeaders"]);

  }

  return composedApplication;
}
```

> **Note**
>
> This specification does not preclude alternative modeling of the Middleware
> functionality from being provided
> by vendors (e.g. object-oriented model, procedural, or separate before/after
> functions), but they MUST provide a way to expose such alternate representations
> as a MidFunc-compatible function.

## 4. Middleware Registration

The component responsible for producing the Application needs to be aware of all
registered Middleware.

In order to provide for capability discovery, configuration and shared
initialisation, this specification introduces a factory function, taking as a
parameter the Startup properties as defined by the
[OWIN specification][owin-spec], and a builder function accepting that factory
as a parameter.

### 4.1. Signature

```csharp
using MidFactory = Func<
                        IDictionary<string, object>, // startup properties
                        MidFunc // returns the Middleware to use
                       >;
using BuildFunc = Action<
                         MidFactory // the factory for a middleware
                        >;
```

#### 4.1.1. Registration Sample

The following example adds the `LogMiddleware` component defined earlier and
registers it with the Builder.

```csharp
// as previously defined
public static AppFunc LogMiddleware(AppFunc next) { /*  ... */ }

// Definition of the factory
public static MidFunc ConfigureLogMiddleware(IDictionary<string, object> startupProperties)
{
  // optionally read and write to the startup properties
  return LogMiddleware;
}
```

### 4.2. Discovery and Registration of Middleware

A common requirement for Middleware authors is to enable users to discover
Middleware after installing a software package in their Application.

In order to deliver this functionality, a Middleware MUST implement an extension
method defined for the Builder function.

Integrated development environments tend to provide API discovery in the form of
graphical navigators. As functions usually have a few other members defined, and
to allow easier navigation, a Middleware SHOULD prefix the extension method with
`Use`, and SHOULDN'T include the word Middleware.

As an Application is often composed of many Middleware components, authors
SHOULD return the BuildFunc in their extension method, to allow for easy
chaining.

#### 4.2.1. Extension Method Sample

Building on previous examples, the following defines the extension method for
the logging Middleware.

```csharp
// as previously defined
public static AppFunc LogMiddleware(AppFunc next) { /*  ... */ }
public static MidFunc ConfigureLogMiddleware(IDictionary<string, object> startupProperties) { /* ... */ }

// definition of the Builder extension method
public static BuildFunc UseHeaderLogging(this BuildFunc builder)
{
  builder(ConfiguereLogMiddleware);
  // unfolded to:
  // builder(startupProperties => ConfigureLogMiddleware(startupProperties));
  // builder(startupProperties => LogMiddleware);
  return builder;
}

// usage
public static void ConfigureWebApplication(BuildFunc builder)
{
  builder.UseHeaderLogging();
}
```

## 5. Versioning

This document specifies version 1.0.0 of this specification. As defined in
[Versioning in OWIN][semver], the patch number will be revised for any
update to the documentation, the minor number will be revised for any addition
of optional and backward and forward compatible changes to this standard,
and the major number will be revised for any breaking change to the protocol
defined in this standard.

## 6. Acknowledgements

Thanks to Kristian Hellang (@khellang) for his valuable feedback.

----
[FieldingLayering]: http://www.ics.uci.edu/~fielding/pubs/dissertation/net_arch_styles.htm#sec_3_4_2
[WGForum]: https://github.com/owin/owin.github.com/issues
[CCLicense]: http://creativecommons.org/licenses/by/3.0/deed.en_US
[CommonKeys]: http://owin.org/spec/CommonKeys.html
[owin-spec]: http://owin.org/spec/owin-1.0.1-draft.1.md
[semver]: http://semver.org/
[rfc2119]: http://www.ietf.org/rfc/rfc2119.txt
[rfc2616]: http://www.ietf.org/rfc/rfc2616.txt
[rfc3986]: http://www.ietf.org/rfc/rfc3986.txt
[rfc2616-42]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec4.html#sec4.2
[rfc2616-512]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html#sec5.1.2
[rfc2616-19611]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec19.html#sec19.6.1.1
[rfc2616-323]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.2.3
