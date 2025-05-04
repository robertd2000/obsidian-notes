https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#what_requests_use_cors

**Cross-Origin Resource Sharing** ([CORS](https://developer.mozilla.org/en-US/docs/Glossary/CORS)) is an [HTTP](https://developer.mozilla.org/en-US/docs/Glossary/HTTP)-header based mechanism that allows a server to indicate any [origins](https://developer.mozilla.org/en-US/docs/Glossary/Origin) (domain, scheme, or port) other than its own from which a browser should permit loading resources. CORS also relies on a mechanism by which browsers make a "preflight" request to the server hosting the cross-origin resource, in order to check that the server will permit the actual request. In that preflight, the browser sends headers that indicate the HTTP method and headers that will be used in the actual request.

An example of a cross-origin request: the front-end JavaScript code served from `https://domain-a.com` uses [`fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/fetch) to make a request for `https://domain-b.com/data.json`.

For security reasons, browsers restrict cross-origin HTTP requests initiated from scripts. For example, `fetch()` and [`XMLHttpRequest`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) follow the [same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy). This means that a web application using those APIs can only request resources from the same origin the application was loaded from unless the response from other origins includes the right CORS headers.

![Diagrammatic representation of CORS mechanism](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS/cors_principle.png)

The CORS mechanism supports secure cross-origin requests and data transfers between browsers and servers. Browsers use CORS in APIs such as `fetch()` or `XMLHttpRequest` to mitigate the risks of cross-origin HTTP requests.

## [What requests use CORS?](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#what_requests_use_cors)

This [cross-origin sharing standard](https://fetch.spec.whatwg.org/#http-cors-protocol) can enable cross-origin HTTP requests for:

- Invocations of `fetch()` or `XMLHttpRequest`, as discussed above.
- Web Fonts (for cross-domain font usage in `@font-face` within CSS), [so that servers can deploy TrueType fonts that can only be loaded cross-origin and used by websites that are permitted to do so.](https://www.w3.org/TR/css-fonts-3/#font-fetching-requirements)
- [WebGL textures](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API/Tutorial/Using_textures_in_WebGL).
- Images/video frames drawn to a canvas using [`drawImage()`](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/drawImage "drawImage()").
- [CSS Shapes from images.](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_shapes/Shapes_from_images)

This is a general article about Cross-Origin Resource Sharing and includes a discussion of the necessary HTTP headers.

## [Functional overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#functional_overview)

The Cross-Origin Resource Sharing standard works by adding new [HTTP headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers) that let servers describe which origins are permitted to read that information from a web browser. Additionally, for HTTP request methods that can cause side-effects on server data (in particular, HTTP methods other than [`GET`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/GET), or [`POST`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST) with certain [MIME types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types)), the specification mandates that browsers "preflight" the request, soliciting supported methods from the server with the HTTP [`OPTIONS`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/OPTIONS) request method, and then, upon "approval" from the server, sending the actual request. Servers can also inform clients whether "credentials" (such as [Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies) and [HTTP Authentication](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)) should be sent with requests.

CORS failures result in errors but for security reasons, specifics about the error _are not available to JavaScript_. All the code knows is that an error occurred. The only way to determine what specifically went wrong is to look at the browser's console for details.

Subsequent sections discuss scenarios, as well as provide a breakdown of the HTTP headers used.

## [Examples of access control scenarios](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#examples_of_access_control_scenarios)

We present three scenarios that demonstrate how Cross-Origin Resource Sharing works. All these examples use [`fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/fetch), which can make cross-origin requests in any supporting browser.

### [Simple requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#simple_requests)

Some requests don't trigger a [CORS preflight](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request). Those are called _simple requests_ from the obsolete [CORS spec](https://www.w3.org/TR/2014/REC-cors-20140116/#terminology), though the [Fetch spec](https://fetch.spec.whatwg.org/) (which now defines CORS) doesn't use that term.

The motivation is that the [`<form>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form) element from HTML 4.0 (which predates cross-site [`fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/fetch) and [`XMLHttpRequest`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)) can submit simple requests to any origin, so anyone writing a server must already be protecting against [cross-site request forgery](https://developer.mozilla.org/en-US/docs/Glossary/CSRF) (CSRF). Under this assumption, the server doesn't have to opt-in (by responding to a preflight request) to receive any request that looks like a form submission, since the threat of CSRF is no worse than that of form submission. However, the server still must opt-in using [`Access-Control-Allow-Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) to _share_ the response with the script.

A _simple request_ is one that **meets all the following conditions**:

- One of the allowed methods:
    - [`GET`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/GET)
    - [`HEAD`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/HEAD)
    - [`POST`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST)
- Apart from the headers automatically set by the user agent (for example, [`Connection`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Connection), [`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent), or [the other headers defined in the Fetch spec as a _forbidden header name_](https://fetch.spec.whatwg.org/#forbidden-header-name)), the only headers which are allowed to be manually set are [those which the Fetch spec defines as a CORS-safelisted request-header](https://fetch.spec.whatwg.org/#cors-safelisted-request-header), which are:
    - [`Accept`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept)
    - [`Accept-Language`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Language)
    - [`Content-Language`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Language)
    - [`Content-Type`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type) (please note the additional requirements below)
    - [`Range`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Range) (only with a [simple range header value](https://fetch.spec.whatwg.org/#simple-range-header-value); e.g., `bytes=256-` or `bytes=127-255`)
- The only type/subtype combinations allowed for the [media type](https://developer.mozilla.org/en-US/docs/Glossary/MIME_type) specified in the [`Content-Type`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type) header are:
    - `application/x-www-form-urlencoded`
    - `multipart/form-data`
    - `text/plain`
- If the request is made using an [`XMLHttpRequest`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) object, no event listeners are registered on the object returned by the [`XMLHttpRequest.upload`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/upload) property used in the request; that is, given an [`XMLHttpRequest`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) instance `xhr`, no code has called `xhr.upload.addEventListener()` to add an event listener to monitor the upload.
- No [`ReadableStream`](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream) object is used in the request.

**Note:** WebKit Nightly and Safari Technology Preview place additional restrictions on the values allowed in the [`Accept`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept), [`Accept-Language`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Language), and [`Content-Language`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Language) headers. If any of those headers have "nonstandard" values, WebKit/Safari does not consider the request to be a "simple request". What values WebKit/Safari consider "nonstandard" is not documented, except in the following WebKit bugs:

- [Require preflight for non-standard CORS-safelisted request headers Accept, Accept-Language, and Content-Language](https://webkit.org/b/165178)
- [Allow commas in Accept, Accept-Language, and Content-Language request headers for simple CORS](https://webkit.org/b/165566)
- [Switch to a blacklist model for restricted Accept headers in simple CORS requests](https://webkit.org/b/166363)

No other browsers implement these extra restrictions because they're not part of the spec.

For example, suppose web content at `https://foo.example` wishes to fetch JSON content from domain `https://bar.other`. Code of this sort might be used in JavaScript deployed on `foo.example`:

JSCopy to Clipboard

```
const fetchPromise = fetch("https://bar.other");

fetchPromise
  .then((response) => response.json())
  .then((data) => {
    console.log(data);
  });
```

This operation performs a simple exchange between the client and the server, using CORS headers to handle the privileges:

![Diagram of simple CORS GET request](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS/simple-req.png)

Let's look at what the browser will send to the server in this case:

HTTPCopy to Clipboard

```
GET /resources/public-data/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Origin: https://foo.example
```

The request header of note is [`Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Origin), which shows that the invocation is coming from `https://foo.example`.

Now let's see how the server responds:

HTTPCopy to Clipboard

```
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 00:23:53 GMT
Server: Apache/2
Access-Control-Allow-Origin: *
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml

[…XML Data…]
```

In response, the server returns a [`Access-Control-Allow-Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) header with `Access-Control-Allow-Origin: *`, which means that the resource can be accessed by **any** origin.

HTTPCopy to Clipboard

```
Access-Control-Allow-Origin: *
```

This pattern of the [`Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Origin) and [`Access-Control-Allow-Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) headers is the simplest use of the access control protocol. If the resource owners at `https://bar.other` wished to restrict access to the resource to requests _only_ from `https://foo.example` (i.e., no domain other than `https://foo.example` can access the resource in a cross-origin manner), they would send:

HTTPCopy to Clipboard

```
Access-Control-Allow-Origin: https://foo.example
```

**Note:** When responding to a [credentialed requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#requests_with_credentials) request, the server **must** specify an origin in the value of the `Access-Control-Allow-Origin` header, instead of specifying the "`*`" wildcard.

### [Preflighted requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#preflighted_requests)

Unlike [_simple requests_](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#simple_requests), for "preflighted" requests the browser first sends an HTTP request using the [`OPTIONS`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/OPTIONS) method to the resource on the other origin, in order to determine if the actual request is safe to send. Such cross-origin requests are preflighted since they may have implications for user data.

The following is an example of a request that will be preflighted:

JSCopy to Clipboard

```
const fetchPromise = fetch("https://bar.other/doc", {
  method: "POST",
  mode: "cors",
  headers: {
    "Content-Type": "text/xml",
    "X-PINGOTHER": "pingpong",
  },
  body: "<person><name>Arun</name></person>",
});

fetchPromise.then((response) => {
  console.log(response.status);
});
```

The example above creates an XML body to send with the `POST` request. Also, a non-standard HTTP `X-PINGOTHER` request header is set. Such headers are not part of HTTP/1.1, but are generally useful to web applications. Since the request uses a `Content-Type` of `text/xml`, and since a custom header is set, this request is preflighted.

![Diagram of a request that is preflighted](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS/preflight_correct.png)

**Note:** As described below, the actual `POST` request does not include the `Access-Control-Request-*` headers; they are needed only for the `OPTIONS` request.

Let's look at the full exchange between client and server. The first exchange is the _preflight request/response_:

HTTPCopy to Clipboard

```
OPTIONS /doc HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Origin: https://foo.example
Access-Control-Request-Method: POST
Access-Control-Request-Headers: content-type,x-pingother

HTTP/1.1 204 No Content
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2
Access-Control-Allow-Origin: https://foo.example
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
Access-Control-Max-Age: 86400
Vary: Accept-Encoding, Origin
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
```

Lines 1 - 10 above represent the preflight request with the [`OPTIONS`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/OPTIONS) method. The browser determines that it needs to send this based on the request parameters that the JavaScript code snippet above was using, so that the server can respond whether it is acceptable to send the request with the actual request parameters. OPTIONS is an HTTP/1.1 method that is used to determine further information from servers, and is a [safe](https://developer.mozilla.org/en-US/docs/Glossary/Safe/HTTP) method, meaning that it can't be used to change the resource. Note that along with the OPTIONS request, two other request headers are sent (lines 9 and 10 respectively):

HTTPCopy to Clipboard

```
Access-Control-Request-Method: POST
Access-Control-Request-Headers: content-type,x-pingother
```

The [`Access-Control-Request-Method`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Request-Method) header notifies the server as part of a preflight request that when the actual request is sent, it will do so with a `POST` request method. The [`Access-Control-Request-Headers`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Request-Headers) header notifies the server that when the actual request is sent, it will do so with `X-PINGOTHER` and `Content-Type` custom headers. Now the server has an opportunity to determine whether it can accept a request under these conditions.

Lines 12 - 21 above are the response that the server returns, which indicate that the request method (`POST`) and request headers (`X-PINGOTHER`) are acceptable. Let's have a closer look at lines 15-18:

HTTPCopy to Clipboard

```
Access-Control-Allow-Origin: https://foo.example
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
Access-Control-Max-Age: 86400
```

The server responds with `Access-Control-Allow-Origin: https://foo.example`, restricting access to the requesting origin domain only. It also responds with `Access-Control-Allow-Methods`, which says that `POST` and `GET` are valid methods to query the resource in question (this header is similar to the [`Allow`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Allow) response header, but used strictly within the context of access control).

The server also sends `Access-Control-Allow-Headers` with a value of "`X-PINGOTHER, Content-Type`", confirming that these are permitted headers to be used with the actual request. Like `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers` is a comma-separated list of acceptable headers.

Finally, [`Access-Control-Max-Age`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Max-Age) gives the value in seconds for how long the response to the preflight request can be cached without sending another preflight request. The default value is 5 seconds. In the present case, the max age is 86400 seconds (= 24 hours). Note that each browser has a [maximum internal value](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Max-Age) that takes precedence when the `Access-Control-Max-Age` exceeds it.

Once the preflight request is complete, the real request is sent:

HTTPCopy to Clipboard

```
POST /doc HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
X-PINGOTHER: pingpong
Content-Type: text/xml; charset=UTF-8
Referer: https://foo.example/examples/preflightInvocation.html
Content-Length: 55
Origin: https://foo.example
Pragma: no-cache
Cache-Control: no-cache

<person><name>Arun</name></person>

HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:40 GMT
Server: Apache/2
Access-Control-Allow-Origin: https://foo.example
Vary: Accept-Encoding, Origin
Content-Encoding: gzip
Content-Length: 235
Keep-Alive: timeout=2, max=99
Connection: Keep-Alive
Content-Type: text/plain

[Some XML payload]
```

#### Preflighted requests and redirects

Not all browsers currently support following redirects after a preflighted request. If a redirect occurs after such a request, some browsers currently will report an error message such as the following:

> The request was redirected to `https://example.com/foo`, which is disallowed for cross-origin requests that require preflight. Request requires preflight, which is disallowed to follow cross-origin redirects.

The CORS protocol originally required that behavior but [was subsequently changed to no longer require it](https://github.com/whatwg/fetch/commit/0d9a4db8bc02251cc9e391543bb3c1322fb882f2). However, not all browsers have implemented the change, and thus still exhibit the originally required behavior.

Until browsers catch up with the spec, you may be able to work around this limitation by doing one or both of the following:

- Change the server-side behavior to avoid the preflight and/or to avoid the redirect
- Change the request such that it is a [simple request](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#simple_requests) that doesn't cause a preflight

If that's not possible, then another way is to:

1. Make a [simple request](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#simple_requests) (using [`Response.url`](https://developer.mozilla.org/en-US/docs/Web/API/Response/url) for the Fetch API, or [`XMLHttpRequest.responseURL`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/responseURL)) to determine what URL the real preflighted request would end up at.
2. Make another request (the _real_ request) using the URL you obtained from `Response.url` or `XMLHttpRequest.responseURL` in the first step.

However, if the request is one that triggers a preflight due to the presence of the `Authorization` header in the request, you won't be able to work around the limitation using the steps above. And you won't be able to work around it at all unless you have control over the server the request is being made to.

### [Requests with credentials](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#requests_with_credentials)

**Note:** When making credentialed requests to a different domain, third-party cookie policies will still apply. The policy is always enforced regardless of any setup on the server and the client as described in this chapter.

The most interesting capability exposed by both [`fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/fetch) or [`XMLHttpRequest`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) and CORS is the ability to make "credentialed" requests that are aware of [HTTP cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies) and HTTP Authentication information. By default, in cross-origin `fetch()` or `XMLHttpRequest` calls, browsers will _not_ send credentials.

To ask for a `fetch()` request to include credentials, set the [`credentials`](https://developer.mozilla.org/en-US/docs/Web/API/Request/Request#credentials) option in the [`Request()`](https://developer.mozilla.org/en-US/docs/Web/API/Request/Request "Request()") constructor to `"include"`.

To ask for an `XMLHttpRequest` request to include credentials, set the [`XMLHttpRequest.withCredentials`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/withCredentials) property to `true`.

In this example, content originally loaded from `https://foo.example` makes a simple GET request to a resource on `https://bar.other` which sets Cookies. Content on foo.example might contain JavaScript like this:

JSCopy to Clipboard

```
const url = "https://bar.other/resources/credentialed-content/";

const request = new Request(url, { credentials: "include" });

const fetchPromise = fetch(request);
fetchPromise.then((response) => console.log(response));
```

This code creates a [`Request`](https://developer.mozilla.org/en-US/docs/Web/API/Request) object, setting the `credentials` option to `"include"` in the constructor, then passes this request into `fetch()`. Since this is a simple `GET` request, it is not preflighted but the browser will **reject** any response that does not have the [`Access-Control-Allow-Credentials`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials)`: true` header, and **not** make the response available to the invoking web content.

![Diagram of a simple GET request with Access-Control-Allow-Credentials](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS/cred-req-updated.png)

Here is a sample exchange between client and server:

HTTPCopy to Clipboard

```
GET /resources/credentialed-content/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Referer: https://foo.example/examples/credential.html
Origin: https://foo.example
Cookie: pageAccess=2

HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:34:52 GMT
Server: Apache/2
Access-Control-Allow-Origin: https://foo.example
Access-Control-Allow-Credentials: true
Cache-Control: no-cache
Pragma: no-cache
Set-Cookie: pageAccess=3; expires=Wed, 31-Dec-2008 01:34:53 GMT
Vary: Accept-Encoding, Origin
Content-Encoding: gzip
Content-Length: 106
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain

[text/plain payload]
```

Although line 10 contains the Cookie destined for the content on `https://bar.other`, if bar.other did not respond with an [`Access-Control-Allow-Credentials`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials)`: true` (line 16), the response would be ignored and not made available to the web content.

#### Preflight requests and credentials

CORS-preflight requests must never include credentials. The _response_ to a preflight request must specify `Access-Control-Allow-Credentials: true` to indicate that the actual request can be made with credentials.

**Note:** Some enterprise authentication services require that TLS client certificates be sent in preflight requests, in contravention of the [Fetch](https://fetch.spec.whatwg.org/#cors-protocol-and-credentials) specification.

Firefox 87 allows this non-compliant behavior to be enabled by setting the preference: `network.cors_preflight.allow_client_cert` to `true` ([Firefox bug 1511151](https://bugzil.la/1511151)). Chromium-based browsers currently always send TLS client certificates in CORS preflight requests ([Chrome bug 775438](https://crbug.com/775438)).

#### Credentialed requests and wildcards

When responding to a credentialed request:

- The server **must not** specify the "`*`" wildcard for the `Access-Control-Allow-Origin` response-header value, but must instead specify an explicit origin; for example: `Access-Control-Allow-Origin: https://example.com`
- The server **must not** specify the "`*`" wildcard for the `Access-Control-Allow-Headers` response-header value, but must instead specify an explicit list of header names; for example, `Access-Control-Allow-Headers: X-PINGOTHER, Content-Type`
- The server **must not** specify the "`*`" wildcard for the `Access-Control-Allow-Methods` response-header value, but must instead specify an explicit list of method names; for example, `Access-Control-Allow-Methods: POST, GET`
- The server **must not** specify the "`*`" wildcard for the `Access-Control-Expose-Headers` response-header value, but must instead specify an explicit list of header names; for example, `Access-Control-Expose-Headers: Content-Encoding, Kuma-Revision`

If a request includes a credential (most commonly a `Cookie` header) and the response includes an `Access-Control-Allow-Origin: *` header (that is, with the wildcard), the browser will block access to the response, and report a CORS error in the devtools console.

But if a request does include a credential (like the `Cookie` header) and the response includes an actual origin rather than the wildcard (like, for example, `Access-Control-Allow-Origin: https://example.com`), then the browser will allow access to the response from the specified origin.

Also note that any `Set-Cookie` response header in a response would not set a cookie if the `Access-Control-Allow-Origin` value in that response is the "`*`" wildcard rather an actual origin.

#### Third-party cookies

Note that cookies set in CORS responses are subject to normal third-party cookie policies. In the example above, the page is loaded from `foo.example` but the cookie on line 19 is sent by `bar.other`, and would thus not be saved if the user's browser is configured to reject all third-party cookies.

Cookie in the request (line 10) may also be suppressed in normal third-party cookie policies. The enforced cookie policy may therefore nullify the capability described in this chapter, effectively preventing you from making credentialed requests whatsoever.

Cookie policy around the [SameSite](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#samesitesamesite-value) attribute would apply.

## [The HTTP response headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#the_http_response_headers)

This section lists the HTTP response headers that servers return for access control requests as defined by the Cross-Origin Resource Sharing specification. The previous section gives an overview of these in action.

### [Access-Control-Allow-Origin](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#access-control-allow-origin)

A returned resource may have one [`Access-Control-Allow-Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) header with the following syntax:

HTTPCopy to Clipboard

```
Access-Control-Allow-Origin: <origin> | *
```

`Access-Control-Allow-Origin` specifies either a single origin which tells browsers to allow that origin to access the resource; or else — for requests **without** credentials — the "`*`" wildcard tells browsers to allow any origin to access the resource.

For example, to allow code from the origin `https://mozilla.org` to access the resource, you can specify:

HTTPCopy to Clipboard

```
Access-Control-Allow-Origin: https://mozilla.org
Vary: Origin
```

If the server specifies a single origin (that may dynamically change based on the requesting origin as part of an allowlist) rather than the "`*`" wildcard, then the server should also include `Origin` in the [`Vary`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Vary) response header to indicate to clients that server responses will differ based on the value of the [`Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Origin) request header.

### [Access-Control-Expose-Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#access-control-expose-headers)

The [`Access-Control-Expose-Headers`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Expose-Headers) header adds the specified headers to the allowlist that JavaScript (such as [`Response.headers`](https://developer.mozilla.org/en-US/docs/Web/API/Response/headers)) in browsers is allowed to access.

HTTPCopy to Clipboard

```
Access-Control-Expose-Headers: <header-name>[, <header-name>]*
```

For example, the following:

HTTPCopy to Clipboard

```
Access-Control-Expose-Headers: X-My-Custom-Header, X-Another-Custom-Header
```

…would allow the `X-My-Custom-Header` and `X-Another-Custom-Header` headers to be exposed to the browser.

### [Access-Control-Max-Age](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#access-control-max-age)

The [`Access-Control-Max-Age`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Max-Age) header indicates how long the results of a preflight request can be cached. For an example of a preflight request, see the above examples.

HTTPCopy to Clipboard

```
Access-Control-Max-Age: <delta-seconds>
```

The `delta-seconds` parameter indicates the number of seconds the results can be cached.

### [Access-Control-Allow-Credentials](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#access-control-allow-credentials)

The [`Access-Control-Allow-Credentials`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials) header indicates whether or not the response to the request can be exposed when the `credentials` flag is true. When used as part of a response to a preflight request, this indicates whether or not the actual request can be made using credentials. Note that simple `GET` requests are not preflighted, and so if a request is made for a resource with credentials, if this header is not returned with the resource, the response is ignored by the browser and not returned to web content.

HTTPCopy to Clipboard

```
Access-Control-Allow-Credentials: true
```

[Credentialed requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#requests_with_credentials) are discussed above.

### [Access-Control-Allow-Methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#access-control-allow-methods)

The [`Access-Control-Allow-Methods`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Methods) header specifies the method or methods allowed when accessing the resource. This is used in response to a preflight request. The conditions under which a request is preflighted are discussed above.

HTTPCopy to Clipboard

```
Access-Control-Allow-Methods: <method>[, <method>]*
```

An example of a [preflight request](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request) is given above, including an example which sends this header to the browser.

### [Access-Control-Allow-Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#access-control-allow-headers)

The [`Access-Control-Allow-Headers`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Headers) header is used in response to a [preflight request](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request) to indicate which HTTP headers can be used when making the actual request. This header is the server side response to the browser's [`Access-Control-Request-Headers`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Request-Headers) header.

HTTPCopy to Clipboard

```
Access-Control-Allow-Headers: <header-name>[, <header-name>]*
```

## [The HTTP request headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#the_http_request_headers)

This section lists headers that clients may use when issuing HTTP requests in order to make use of the cross-origin sharing feature. Note that these headers are set for you when making invocations to servers. Developers making cross-origin requests do not have to set any cross-origin sharing request headers programmatically.

### [Origin](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#origin)

The [`Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Origin) header indicates the origin of the cross-origin access request or preflight request.

HTTPCopy to Clipboard

```
Origin: <origin>
```

The origin is a URL indicating the server from which the request is initiated. It does not include any path information, only the server name.

**Note:** The `origin` value can be `null`.

Note that in any access control request, the [`Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Origin) header is **always** sent.

### [Access-Control-Request-Method](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#access-control-request-method)

The [`Access-Control-Request-Method`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Request-Method) is used when issuing a preflight request to let the server know what HTTP method will be used when the actual request is made.

HTTPCopy to Clipboard

```
Access-Control-Request-Method: <method>
```

Examples of this usage can be [found above.](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#preflighted_requests)

### [Access-Control-Request-Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#access-control-request-headers)

The [`Access-Control-Request-Headers`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Request-Headers) header is used when issuing a preflight request to let the server know what HTTP headers will be used when the actual request is made (for example, by passing them as the [`headers`](https://developer.mozilla.org/en-US/docs/Web/API/Request/Request#headers) option to the [`Request()`](https://developer.mozilla.org/en-US/docs/Web/API/Request/Request "Request()") constructor). This browser-side header will be answered by the complementary server-side header of [`Access-Control-Allow-Headers`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Headers).

HTTPCopy to Clipboard

```
Access-Control-Request-Headers: <field-name>[,<field-name>]*
```

Examples of this usage can be [found above](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#preflighted_requests).

## [Specifications](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#specifications)

|Specification|
|---|
|[Fetch Standard  <br># http-access-control-allow-origin](https://fetch.spec.whatwg.org/#http-access-control-allow-origin)|

## [Browser compatibility](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#browser_compatibility)

[Report problems with this compatibility data on GitHub](https://github.com/mdn/browser-compat-data/issues/new?mdn-url=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FHTTP%2FCORS&metadata=%3C%21--+Do+not+make+changes+below+this+line+--%3E%0A%3Cdetails%3E%0A%3Csummary%3EMDN+page+report+details%3C%2Fsummary%3E%0A%0A*+Query%3A+%60http.headers.Access-Control-Allow-Origin%60%0A*+Report+started%3A+2024-06-06T08%3A59%3A20.085Z%0A%0A%3C%2Fdetails%3E&title=http.headers.Access-Control-Allow-Origin+-+%3CSUMMARIZE+THE+PROBLEM%3E&template=data-problem.yml "Report an issue with this compatibility data")

||desktop|   |   |   |   |mobile|   |   |   |   |   |
|---|---|---|---|---|---|---|---|---|---|---|---|
||Chrome|Edge|Firefox|Opera|Safari|Chrome Android|Firefox for Android|Opera Android|Safari on iOS|Samsung Internet|WebView Android|
|---|---|---|---|---|---|---|---|---|---|---|---|
|`Access-Control-Allow-Origin`|4<br><br>Toggle history|12<br><br>Toggle history|3.5<br><br>Toggle history|12<br><br>Toggle history|4<br><br>Toggle history|Yes<br><br>Toggle history|4<br><br>Toggle history|12<br><br>Toggle history|3.2<br><br>Toggle history|Yes<br><br>Toggle history|2<br><br>Toggle history|

### Legend

Tip: you can click/tap on a cell for more information.