If we positioned it after our rate limiter, for example, any cross-origin requests that exceed the rate limit would _not have the `Access-Control-Allow-Origin` header set_. This means that they would be blocked by the client’s web browser due to the same-origin policy, rather than the client receiving a `429 Too Many Requests` response like they should.

So because of this we should make sure to always set a `Vary: Origin` response header to warn any caches that the response may be different. This is actually really important, and it can be the cause of subtle bugs [like this one](https://textslashplain.com/2018/08/02/cors-and-vary/) if you forget to do it. As a rule of thumb:

_If your code makes a decision about what to return based on the content of a request header, you should include that header name in your `Vary` response header — even if the request didn’t include that header._

### Additional Information

#### Partial origin matches

If you have a lot of trusted origins that you want to support, then you might be tempted to check for a partial match on the origin to see if it ‘starts with’ or ‘ends with’ a specific value, or matches a regular expression. If you do this, you must take a lot of care to avoid any unintentional matches.

As a simple example, if `http://example.com` and `http://www.example.com` are your trusted origins, your first thought might check that the request `Origin` header ends with `example.com`. This would be a bad idea, as an attacker could register the domain name `attackerexample.com` and any requests from that origin would pass your check.

This is just one simple example — and the following blog posts discuss some of the other vulnerabilities that can arise when using partial match or regular expression checks:

- [Security Risks of CORS](https://medium.com/@ehayushpathak/security-risks-of-cors-e3f4a25c04d7)
- [Exploiting CORS misconfigurations for Bitcoins and bounties](https://portswigger.net/research/exploiting-cors-misconfigurations-for-bitcoins-and-bounties)

Generally, it’s best to check the `Origin` request header against an explicit safelist of full-length trusted origins, like we have done in this chapter.

#### The null origin

It’s important to never include the value `"null"` as a trusted origin in your safelist. This is because the request header `Origin: null` can be forged by an attacker by sending a request from a [sandboxed iframe](https://stackoverflow.com/a/44765536).

#### Authentication and CORS

If your API endpoint requires _credentials_ (cookies or HTTP basic authentication) you should also set an [`Access-Control-Allow-Credentials: true`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials) header in your responses. If you don’t set this header, then the web browser will prevent any cross-origin responses with credentials from being read by JavaScript.

Importantly, you must never use the wildcard `Access-Control-Allow-Origin: *` header in conjunction with `Access-Control-Allow-Credentials: true`, as this would allow any website to make a credentialed cross-origin request to your API.

Also, importantly, if you want credentials to be sent with a cross-origin request then you’ll need to explicitly specify this in your JavaScript. For example, with `fetch()` you should set the `credentials` value of the request to `'include'`. Like so:

`fetch("https://api.example.com", {credentials: 'include'}).then( ... );`

Or if using [`XMLHTTPRequest`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) you should set the [`withCredentials`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/withCredentials) property to `true`. For example:
```
var xhr = new XMLHttpRequest();
xhr.open('GET', 'https://api.example.com');
xhr.withCredentials = true;
xhr.send(null);
```


## Preflight CORS Requests

The cross-origin request that we made from JavaScript in the previous chapter is known as a _simple_ cross-origin request. Broadly speaking, cross-origin requests are classified as ‘simple’ when _all_ the following conditions are met:

- The request HTTP method is one of the three CORS-safe methods: `HEAD`, `GET` or `POST`.
- The request headers are all either [forbidden headers](https://developer.mozilla.org/en-US/docs/Glossary/Forbidden_header_name) or one of the four CORS-safe headers:
    - `Accept`
    - `Accept-Language`
    - `Content-Language`
    - `Content-Type`
- The value for the `Content-Type` header (if set) is one of:
    - `application/x-www-form-urlencoded`
    - `multipart/form-data`
    - `text/plain`

When a cross-origin request doesn’t meet these conditions, then the web browser will trigger an initial ‘preflight’ request _before the real request_. The purpose of this preflight request is to determine whether the _real_ cross-origin request will be permitted or no

