---
title: "Understanding Cross-Origin Resource Sharing (CORS)"
date: 2020-10-22T20:11:41+08:00
draft: false
tags:
- web
---

## What is CORS
Cross-Origin Resource Sharing (CORS) is a mechanism that uses additional HTTP headers to tell browsers to give a web application running at one origin, access to selected resources from a different origin. A web application executes a cross-origin HTTP request when it requests a resource that has a different origin (domain, protocol, or port) from its own.

An example of a cross-origin request: the front-end JavaScript code served from `https://domain-a.com` uses XMLHttpRequest to make a request for `https://domain-b.com/data.json`.

For security reasons, browsers restrict cross-origin HTTP requests initiated from scripts. For example, XMLHttpRequest and the Fetch API follow the same-origin policy. This means that a web application using those APIs can only request resources from the same origin the application was loaded from unless the response from other origins includes the right CORS headers.

{{< pagebundle/img "**CORS_principle.png" "CORS principle" >}}

The CORS mechanism supports secure cross-origin requests and data transfers between browsers and servers. Modern browsers use CORS in APIs such as XMLHttpRequest or Fetch to mitigate the risks of cross-origin HTTP requests.

## What requests use CORS?
This cross-origin sharing standard can enable cross-site HTTP requests for:

* Invocations of the XMLHttpRequest or Fetch APIs, as discussed above.
* Web Fonts (for cross-domain font usage in @font-face within CSS), so that servers can deploy TrueType fonts that can only be cross-site loaded and used by web sites that are permitted to do so.
* WebGL textures.
* Images/video frames drawn to a canvas using drawImage().
* CSS Shapes from images.

## Functional overview
The Cross-Origin Resource Sharing standard works by adding new HTTP headers that let servers describe which origins are permitted to read that information from a web browser. Additionally, for HTTP request methods that can cause side-effects on server data (in particular, HTTP methods other than **GET**, or **POST** with certain MIME types), the specification mandates that browsers "preflight" the request, soliciting supported methods from the server with the HTTP **OPTIONS** request method, and then, upon "approval" from the server, sending the actual request. Servers can also inform clients whether "credentials" (such as Cookies and HTTP Authentication) should be sent with requests.

CORS failures result in errors, but for security reasons, specifics about the error are not available to JavaScript. All the code knows is that an error occurred. The only way to determine what specifically went wrong is to look at the browser's console for details.

## The HTTP response headers
This section lists the HTTP response headers that servers send back for access control requests as defined by the Cross-Origin Resource Sharing specification. The previous section gives an overview of these in action.

### Access-Control-Allow-Origin
A returned resource may have one Access-Control-Allow-Origin header, with the following syntax:
> Access-Control-Allow-Origin: <origin> | *

**Access-Control-Allow-Origin** specifies either a single origin, which tells browsers to allow that origin to access the resource; or else — for requests without credentials — the "*" wildcard, to tell browsers to allow any origin to access the resource.

For example, to allow code from the origin `https://mozilla.org` to access the resource, you can specify:

> Access-Control-Allow-Origin: `https://mozilla.org`
> Vary: Origin

If the server specifies a single origin (that may dynamically change based on the requesting origin as part of a white-list) rather than the "*" wildcard, then the server should also include Origin in the Vary response header — to indicate to clients that server responses will differ based on the value of the Origin request header.

### Access-Control-Expose-Headers
The Access-Control-Expose-Headers header lets a server whitelist headers that Javascript (such as getResponseHeader()) in browsers are allowed to access.

> Access-Control-Expose-Headers: `<header-name>[, <header-name>]*`

For example, the following:

> Access-Control-Expose-Headers: X-My-Custom-Header, X-Another-Custom-Header
…would allow the X-My-Custom-Header and X-Another-Custom-Header headers to be exposed to the browser.

### Access-Control-Max-Age
The Access-Control-Max-Age header indicates how long the results of a preflight request can be cached. For an example of a preflight request, see the above examples.

> Access-Control-Max-Age: `<delta-seconds>`

The delta-seconds parameter indicates the number of seconds the results can be cached.

### Access-Control-Allow-Credentials
The Access-Control-Allow-Credentials header Indicates whether or not the response to the request can be exposed when the credentials flag is true. When used as part of a response to a preflight request, this indicates whether or not the actual request can be made using credentials. Note that simple GET requests are not preflighted, and so if a request is made for a resource with credentials, if this header is not returned with the resource, the response is ignored by the browser and not returned to web content.

> Access-Control-Allow-Credentials: `true`

Credentialed requests are discussed above.

### Access-Control-Allow-Methods
The Access-Control-Allow-Methods header specifies the method or methods allowed when accessing the resource. This is used in response to a preflight request. The conditions under which a request is preflighted are discussed above.

> Access-Control-Allow-Methods: `<method>[, <method>]*`

An example of a preflight request is given above, including an example which sends this header to the browser.

### Access-Control-Allow-Headers
The Access-Control-Allow-Headers header is used in response to a preflight request to indicate which HTTP headers can be used when making the actual request.

> Access-Control-Allow-Headers: `<header-name>[, <header-name>]*`

## The HTTP request headers
This section lists headers that clients may use when issuing HTTP requests in order to make use of the cross-origin sharing feature. Note that these headers are set for you when making invocations to servers. Developers using cross-site XMLHttpRequest capability do not have to set any cross-origin sharing request headers programmatically.

### Origin
The Origin header indicates the origin of the cross-site access request or preflight request.

> Origin: `<origin>`

The origin is a URI indicating the server from which the request initiated. It does not include any path information, but only the server name.
{{< notice info >}}
Note: The origin value can be **null**, or a **URI**.
Note that in any access control request, the Origin header is always sent.
{{< /notice >}}

### Access-Control-Request-Method
The Access-Control-Request-Method is used when issuing a preflight request to let the server know what HTTP method will be used when the actual request is made.

> Access-Control-Request-Method: `<method>`

Examples of this usage can be found above.

### Access-Control-Request-Headers
The Access-Control-Request-Headers header is used when issuing a preflight request to let the server know what HTTP headers will be used when the actual request is made.

> Access-Control-Request-Headers: `<field-name>[, <field-name>]*`


## Reference
1. https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
2. https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy