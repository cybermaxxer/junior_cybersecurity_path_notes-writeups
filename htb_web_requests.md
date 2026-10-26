> ⚠️ This writeup covers retired/free HTB content only, per HTB's Terms of Service.

# HTB Academy: HTTP Web Requests, My Notes

## Table of Contents

* [1. The 80/20 Digest](#1-the-8020-digest)
   * [1.1 HyperText Transfer Protocol (HTTP)](#11-hypertext-transfer-protocol-http)
   * [1.2 Hypertext Transfer Protocol Secure (HTTPS)](#12-hypertext-transfer-protocol-secure-https)
   * [1.3 HTTP Requests and Responses](#13-http-requests-and-responses)
   * [1.4 HTTP Headers](#14-http-headers)
   * [1.5 HTTP Methods and Codes](#15-http-methods-and-codes)
   * [1.6 GET](#16-get)
   * [1.7 POST](#17-post)
   * [1.8 CRUD API](#18-crud-api)
* [2. Structured Breakdown (Cheat Sheet)](#2-structured-breakdown-cheat-sheet)
* [3. Feynman Checks](#3-feynman-checks)
* [4. Omitted Chuff](#4-omitted-chuff)

---

## 1. The 80/20 Digest

i kept every section of the module here, in the original order, with nothing trimmed. the only things i added are a quick check after each block and a "my personal actions" part where i actually ran something myself.

---

### 1.1 HyperText Transfer Protocol (HTTP)

today most of the applications i use, web and mobile, constantly interact with the internet. most of that traffic is web requests made through the HTTP protocol. HTTP is an application level protocol used to access World Wide Web resources. the term stands for hypertext: text containing links to other resources, and text that readers can easily interpret.

HTTP communication has a client and a server. the client requests a resource from the server, the server processes the request and returns the requested resource. the default port for HTTP is port `80`, though it can be changed to any other port depending on the web server configuration. the same requests are used whenever i visit a website: i enter a Fully Qualified Domain Name (FQDN) as a Uniform Resource Locator (URL) to reach the site i want, like `www.hackthebox.com`.

Q: default HTTP port? A: 80, but the server config can change it.

#### URL structure

resources over HTTP are accessed via a URL, which offers many more specifications than just naming a website i want to visit. the module shows a diagram of the URL structure here, worth a look. here is what each component stands for:

| Component | Example | Description |
|---|---|---|
| Scheme | `http://` `https://` | identifies the protocol being accessed by the client, and ends with a colon and a double slash (`://`) |
| User Info | `admin:password@` | optional component that contains the credentials (separated by a colon `:`) used to authenticate to the host, and is separated from the host with an at sign (`@`) |
| Host | `inlanefreight.com` | the host signifies the resource location. this can be a hostname or an IP address |
| Port | `:80` | the port is separated from the host by a colon (`:`). if no port is specified, `http` schemes default to port `80` and `https` default to port `443` |
| Path | `/dashboard.php` | points to the resource being accessed, which can be a file or a folder. if there is no path specified, the server returns the default index (e.g. `index.html`) |
| Query String | `?login=true` | the query string starts with a question mark (`?`), and consists of a parameter (e.g. `login`) and a value (e.g. `true`). multiple parameters can be separated by an ampersand (`&`) |
| Fragments | `#status` | fragments are processed by the browsers on the client side to locate sections within the primary resource (e.g. a header or section on the page) |

not all components are required to access a resource. the main mandatory fields are the scheme and the host, without which the request would have no resource to request.

Q: which two URL parts are mandatory? A: scheme and host.

#### HTTP Flow

the diagram in the module presents the anatomy of an HTTP request at a very high level (look at it). the first time a user enters the URL (`inlanefreight.com`) into the browser, it sends a request to a DNS (Domain Name System) server to resolve the domain and get its IP. the DNS server looks up the IP address for `inlanefreight.com` and returns it. all domain names need to be resolved this way, as a server can't communicate without an IP address.

> note: browsers usually first look up records in the local `/etc/hosts` file, and if the requested domain does not exist within it, they contact other DNS servers. i can use `/etc/hosts` to manually add records for DNS resolution, by adding the IP followed by the domain name.

once the browser gets the IP address linked to the requested domain, it sends a `GET` request to the default HTTP port (e.g. `80`), asking for the root path `/`. then the web server receives the request and processes it. by default, servers are configured to return an index file when a request for `/` is received.

in this case, the contents of `index.html` are read and returned by the web server as an HTTP response. the response also contains the status code (e.g. `200 OK`), which indicates that the request was successfully processed. the web browser then renders the contents of `index.html` and presents it to the user.

> note: this module is mainly focused on HTTP web requests. for more on HTML and web applications, the Introduction to Web Applications module is the place to go.

Q: which file gets checked before DNS servers? A: `/etc/hosts`.

#### cURL

in this module i send web requests through two of the most important tools for any web penetration tester: a web browser, like chrome or firefox, and the command line tool cURL.

cURL (client URL) is a command line tool and library that primarily supports HTTP along with many other protocols. this makes it a good candidate for scripts as well as automation, making it essential for sending various types of web requests from the command line, which is necessary for many types of web penetration tests.

for this example i use http://info.cern.ch/ which is widely recognized as the first website ever created. it was created in 1990 and made publicly accessible in August 1991 by Tim Berners Lee at CERN. a restored version of the site is still available today, preserving its original content and accessible over HTTP.

i can send a basic HTTP request to any URL by using it as an argument for cURL, as follows:

```shellsession
cybermaxxer@htb[/htb]$ curl http://info.cern.ch/

<html><head></head><body><header>
<title>http://info.cern.ch</title>
</header>

<h1>http://info.cern.ch - home of the first website</h1>
...SNIP...
```

cURL does not render the HTML/JavaScript/CSS code, unlike a web browser, but prints it in its raw format. as a penetration tester i'm mainly interested in the request and response context, which usually becomes much faster and more convenient than a web browser.

i can also use cURL to download a page or a file and output the content into a file using the `-O` flag. if i want to specify the output file name, i use the `-o` flag and specify the name. otherwise, i use `-O` and cURL uses the remote file name, as follows:

```shellsession
cybermaxxer@htb[/htb]$ curl -O http://info.cern.ch/index.html

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   646  100   646    0     0   9773      0 --:--:-- --:--:-- --:--:--  9787

cybermaxxer@htb[/htb]$ ls
index.html
```

the output was not printed this time but saved into `index.html`. cURL still printed some status while processing the request. i can silence the status with the `-s` flag, as follows:

```shellsession
cybermaxxer@htb[/htb]$ curl -s -O http://info.cern.ch/index.html
```

this time, cURL did not print anything, as the output was saved into the `index.html` file. finally, i can use the `-h` flag to see what other options i may use with cURL:

```shellsession
cybermaxxer@htb[/htb]$ curl -h

Usage: curl [options...] <url>
 -d, --data <data>          HTTP POST data
 -h, --help <category>      Get help for commands
 -i, --include              Include protocol response headers in the output
 -o, --output <file>        Write to file instead of stdout
 -O, --remote-name          Write output to a file named as the remote file
 -s, --silent               Silent mode
 -u, --user <user:password> Server user and password
 -A, --user-agent <name>    Send User-Agent <name> to server
 -v, --verbose              Make the operation more talkative

This is not the full help, this menu is stripped into categories.
Use "--help category" to get an overview of all categories.
Use the user manual `man curl` or the "--help all" flag for all options.
```

as the message mentions, i can use `--help all` to print a more detailed help menu, or `--help category` (e.g. `-h http`) to print the detailed help of a specific flag. if i ever need to read more detailed documentation, i use `man curl` to view the full cURL manual page.

in the upcoming sections i cover most of the above flags and see where to use each of them.

| flag | what it does |
|---|---|
| `-O` | save output to a file named like the remote file |
| `-o <file>` | save output to a file name i pick |
| `-s` | silent, hides the progress status |
| `-h` | help menu (`--help all` for everything, `--help category` for one area) |

Q: difference between `-O` and `-o`? A: remote file name versus my own name.

#### exercise questions

the module has question 1 for this section. the question text didn't make it into my copy of the page, what i ran is under my personal actions below.

#### my personal actions

```bash
curl -O (ip address)/download.php
```

`(ip address)` is the spawned target's IP, left as a placeholder.

---

### 1.2 Hypertext Transfer Protocol Secure (HTTPS)

in the previous section i covered how HTTP requests are sent and processed. one of the significant drawbacks of HTTP is that all data is transferred in clear text. this means anyone between the source and destination can perform a Man in the Middle (MiTM) attack to view the transferred data.

to counter this issue, the HTTPS (HTTP Secure) protocol was created, in which all communications are transferred in an encrypted format, so even if a third party does intercept the request, they would not be able to extract the data out of it. for this reason, HTTPS has become the mainstream scheme for websites on the internet, and HTTP is being phased out, and soon most web browsers will not allow visiting HTTP websites.

Q: why was HTTPS created? A: HTTP sends everything in clear text.

#### HTTPS Overview

if i examine an HTTP request, i can see the effect of not enforcing secure communications between a web browser and a web application. for example, the module shows the content of an HTTP login request (look at the screenshot).

the login credentials can be viewed in clear text. this would make it easy for someone on the same network (such as a public wireless network) to capture the request and reuse the credentials for malicious purposes.

in contrast, when someone intercepts and analyzes traffic from an HTTPS request, they would see something like the second screenshot in the module: the data is transferred as a single encrypted stream, which makes it very difficult for anyone to capture information such as credentials or any other sensitive data.

websites that enforce HTTPS can be identified through `https://` in their URL (e.g. https://www.google.com), as well as the lock icon in the address bar of the web browser, to the left of the URL (the module has a screenshot of it). so, if i visit a website that uses HTTPS, like google, all traffic is encrypted.

> note: although the data transferred through the HTTPS protocol may be encrypted, the request may still reveal the visited URL if it contacted a clear text DNS server. for this reason, it is recommended to use encrypted DNS servers (e.g. 8.8.8.8 or 1.1.1.1), or use a VPN service to ensure all traffic is properly encrypted.

Q: does HTTPS hide the domain from clear text DNS? A: no, the lookup still leaks it.

#### HTTPS Flow

this is how HTTPS operates at a high level (the module has a flow diagram, look at it).

if i type `http://` instead of `https://` to visit a website that enforces HTTPS, the browser attempts to resolve the domain and redirects me to the webserver hosting the target website. a request is sent to port `80` first, which is the unencrypted HTTP protocol. the server detects this and redirects the client to secure HTTPS port `443` instead. this is done via the `301 Moved Permanently` response code, which i cover in an upcoming section.

next, the client (web browser) sends a "client hello" packet, giving information about itself. after this, the server replies with "server hello", followed by a key exchange to exchange SSL certificates. the client verifies the key/certificate and sends one of its own. after this, an encrypted handshake is initiated to confirm whether the encryption and transfer are working correctly.

once the handshake completes successfully, normal HTTP communication is continued, which is encrypted after that. this is a very high level overview of the key exchange, which is beyond this module's scope.

> note: depending on the circumstances, an attacker may be able to perform an HTTP downgrade attack, which downgrades HTTPS communication to HTTP, making the data transferred in clear text. this is done by setting up a Man In The Middle (MITM) proxy to transfer all traffic through the attacker's host without the user's knowledge. however, most modern browsers, servers, and web applications protect against this attack.

| step | what happens |
|---|---|
| 1 | browser sends the request to port `80` (plain HTTP) |
| 2 | server answers `301 Moved Permanently` pointing to port `443` |
| 3 | client hello, then server hello |
| 4 | key exchange, certificates verified both ways |
| 5 | encrypted handshake confirms everything works |
| 6 | normal HTTP continues, now encrypted |

Q: which status code redirects `80` to `443`? A: `301 Moved Permanently`.

#### cURL for HTTPS

cURL should automatically handle all HTTPS communication standards and perform a secure handshake and then encrypt and decrypt data automatically. however, if i ever contact a website with an invalid SSL certificate or an outdated one, then cURL by default would not proceed with the communication to protect against the earlier mentioned MITM attacks:

```shellsession
cybermaxxer@htb[/htb]$ curl https://inlanefreight.com

curl: (60) SSL certificate problem: Invalid certificate chain
More details here: https://curl.haxx.se/docs/sslcerts.html

...SNIP...
```

modern web browsers would do the same, warning the user against visiting a website with an invalid SSL certificate.

i may face such an issue when testing a local web application or a web application hosted for practice purposes, as such web applications may not yet have implemented a valid SSL certificate. to skip the certificate check with cURL, i use the `-k` flag:

```shellsession
cybermaxxer@htb[/htb]$ curl -k https://www.inlanefreight.com

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
...SNIP...
```

the request went through this time, and i received the response data.

Q: which flag skips the certificate check? A: `-k`.

---

### 1.3 HTTP Requests and Responses

HTTP communications mainly consist of an HTTP request and an HTTP response. an HTTP request is made by the client (e.g. cURL/browser), and is processed by the server (e.g. web server). the requests contain all of the details i require from the server, including the resource (e.g. URL, path, parameters), any request data, headers or options i specify, and many other options i cover throughout this module.

once the server receives the HTTP request, it processes it and responds by sending the HTTP response, which contains the response code, as discussed in a later section, and may contain the resource data if the requester has access to it.

Q: who sends the response code? A: the server.

#### HTTP Request

here is an example HTTP request (the module has a screenshot, look at it). it shows an HTTP GET request to the URL `http://inlanefreight.com/users/login.html`.

the first line of any HTTP request contains three main fields separated by spaces:

| Field | Example | Description |
|---|---|---|
| Method | `GET` | the HTTP method or verb, which specifies the type of action to perform |
| Path | `/users/login.html` | the path to the resource being accessed. this field can also be suffixed with a query string (e.g. `?username=user`) |
| Version | `HTTP/1.1` | the third and final field is used to denote the HTTP version |

the next set of lines contain HTTP header value pairs, like `Host`, `User-Agent`, `Cookie`, and many other possible headers. these headers are used to specify various attributes of a request. the headers are terminated with a new line, which is necessary for the server to validate the request. finally, a request may end with the request body and data.

> note: HTTP version 1.X sends requests as clear text, and uses a new line character to separate different fields and different requests. HTTP version 2.X, on the other hand, sends requests as binary data in a dictionary form.

Q: which HTTP version sends binary data? A: 2.X.

#### HTTP Response

once the server processes my request, it sends its response. the module shows an example HTTP response (screenshot).

the first line of an HTTP response contains two fields separated by spaces. the first is the HTTP version (e.g. `HTTP/1.1`), and the second denotes the HTTP response code (e.g. `200 OK`).

response codes are used to determine the request's status, as discussed in a later section. after the first line, the response lists its headers, similar to an HTTP request. both request and response headers are discussed in the next section.

finally, the response may end with a response body, which is separated by a new line after the headers. the response body is usually defined as HTML code. however, it can also respond with other code types such as JSON, website resources such as images, style sheets or scripts, or even a document such as a PDF document hosted on the webserver.

Q: where does the response body start? A: after the new line following headers.

#### cURL

in my earlier examples with cURL, i only specified the URL and got the response body in return. however, cURL also allows me to preview the full HTTP request and the full HTTP response, which can become very handy when performing web penetration tests or writing exploits. to view the full HTTP request and response, i add the `-v` verbose flag to my earlier commands, and it prints both the request and response:

```shellsession
cybermaxxer@htb[/htb]$ curl inlanefreight.com -v

*   Trying SERVER_IP:80...
* TCP_NODELAY set
* Connected to inlanefreight.com (SERVER_IP) port 80 (#0)
> GET / HTTP/1.1
> Host: inlanefreight.com
> User-Agent: curl/7.65.3
> Accept: */*
> Connection: close
>
* Mark bundle as not supporting multiuse
< HTTP/1.1 401 Unauthorized
< Date: Tue, 21 Jul 2020 05:20:15 GMT
< Server: Apache/X.Y.ZZ (Ubuntu)
< WWW-Authenticate: Basic realm="Restricted Content"
< Content-Length: 464
< Content-Type: text/html; charset=iso-8859-1
<
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
...SNIP...
```

this time i get the full HTTP request and response. the request simply sent `GET / HTTP/1.1` along with the `Host`, `User-Agent` and `Accept` headers. in return, the HTTP response contained `HTTP/1.1 401 Unauthorized`, which indicates that i do not have access to the requested resource, as i see in an upcoming section. similar to the request, the response also contained several headers sent by the server, including `Date`, `Content-Length`, and `Content-Type`. finally, the response contained the response body in HTML, which is the same one i received earlier when using cURL without the `-v` flag.

> exercise for me: the `-vvv` flag shows an even more verbose output. try it to see what extra request and response details get displayed.

Q: in `-v` output, what do `>` and `<` mark? A: request and response lines.

#### Browser DevTools

most modern web browsers come with built in developer tools (DevTools), which are mainly intended for developers to test their web applications. however, as a web penetration tester, these tools are a vital asset in any web assessment, as a browser (and its DevTools) are among the assets i am most likely to have in every web assessment exercise. in this module i also discuss how to use some of the basic browser devtools to assess and monitor different types of web requests.

whenever i visit any website or access any web application, my browser sends multiple web requests and handles multiple HTTP responses to render the final view i see in the browser window. to open the browser devtools in either chrome or firefox, i press `CTRL+SHIFT+I` or simply `F12`. the devtools contain multiple tabs, each with its own use. i mostly focus on the `Network` tab in this module, as it is responsible for web requests.

if i click on the Network tab and refresh the page, i see the list of requests sent by the page (screenshot in the module).

the devtools show at a glance the response status (i.e. response code), the request method used (`GET`), the requested resource (i.e. URL/domain), along with the requested path. furthermore, i can use `Filter URLs` to search for a specific request, in case the website loads too many to go through.

> exercise for me: click on any of the requests to view their details. then click on the `Response` tab to view the response body, and click on the `Raw` button to view the raw (unrendered) source code of the response body.

Q: which devtools tab shows web requests? A: Network.

#### exercise questions

**question 1:** what is the HTTP method used while intercepting the request? (case-sensitive)

answer recorded in the module: `get`

**question 2:** send a GET request to the above server, and read the response headers to find the version of Apache running on the server, then submit it as the answer. (answer format: X.Y.ZZ)

answer recorded in the module: `2.4.41`

#### my personal actions

```bash
curl -I (ip address)
```

`-I` sends a `HEAD` request and prints only the response headers, so the `Server` header gives me the apache version.

---

### 1.4 HTTP Headers

i have seen examples of HTTP request and response headers in the previous section. such HTTP headers pass information between the client and the server. some headers are only used with either requests or responses, while some other general headers are common to both.

headers can have one or multiple values, appended after the header name and separated by a colon. i can divide headers into the following categories:

1. General Headers
2. Entity Headers
3. Request Headers
4. Response Headers
5. Security Headers

it is important to note that this five way split comes from the original HTTP/1.1 specification, RFC 2616, which was obsoleted back in 2014 by RFC 7230 to 7235, which were themselves obsoleted in 2022 by the current specification, RFC 9110. RFC 9110 no longer organizes fields this way. it groups them by function instead (representation metadata, content negotiation, authentication, caching, and so on). the categories below are kept as a simplified mental model for learning; each section notes where that type of header actually lives in the current specs.

Q: which RFC is the current HTTP spec? A: RFC 9110.

#### General Headers

General headers is an RFC 2616 term for fields used in both HTTP requests and responses. they are contextual and are used to describe the message rather than its contents. RFC 9110 dropped this category and instead defines fields individually; Section 5 covers field syntax in general.

| Header | Example | Description |
|---|---|---|
| `Date` | `Date: Wed, 16 Feb 2022 10:38:44 GMT` | holds the date and time at which the message originated. it's preferred to convert the time to the standard UTC time zone |
| `Connection` | `Connection: close` | dictates if the current network connection should stay alive after the request finishes. two commonly used values for this header are `close` and `keep-alive`. the `close` value from either the client or server means that they would like to terminate the connection, while the `keep-alive` header indicates that the connection should remain open to receive more data and input |
| `Cache-Control` | `Cache-Control: no-store` | controls whether and how the message may be cached by browsers and intermediate proxies. from a security standpoint, responses containing sensitive data (session tokens, personal information) should set `no-store` so that data isn't written to disk or shared caches |

Q: which `Cache-Control` value keeps secrets off disk? A: `no-store`.

#### Entity Headers

similar to general headers, Entity Headers is another RFC 2616 term, for fields common to both the request and response. these headers are used to describe the content (entity) transferred by a message. they are usually found in responses and POST or PUT requests. RFC 9110 replaced this category with Section 8, Representation Data and Metadata, calling these representation header fields rather than entity headers, but the older term is still widely used in practice.

| Header | Example | Description |
|---|---|---|
| `Content-Type` | `Content-Type: text/html` | used to describe the media type of the resource being transferred, e.g. `text/html`, `application/json`, or `application/octet-stream`. the value is automatically added by the browser on the client side and returned in the server response. the `charset` field denotes the encoding standard, such as UTF-8 |
| `Content-Length` | `Content-Length: 385` | holds the size of the entity being passed, in bytes. this header is necessary as the server uses it to read data from the message body, and is automatically generated by the browser and tools like cURL |
| `Content-Encoding` | `Content-Encoding: gzip` | data can undergo multiple transformations before being passed. for example, large amounts of data can be compressed to reduce the message size. the type of encoding being used should be specified using the `Content-Encoding` header |
| `Transfer-Encoding` | `Transfer-Encoding: chunked` | specifies an encoding applied to safely transfer the body over HTTP/1.1, independent of the underlying content. the `chunked` value lets a server stream a response without knowing its total length upfront. inconsistent handling of `Transfer-Encoding` and `Content-Length` between a front end proxy and back end server is the root cause of HTTP request smuggling attacks |
| `Content-Type` Boundary | `Content-Type: multipart/form-data; boundary=b4e4fbd93540` | a parameter of the `Content-Type` header, rather than a standalone header, used as a marker to separate content when a message contains more than one part, such as a multipart form submission. the boundary value (`b4e4fbd93540`) is then used as `--` to separate the different parts of the form |

Q: what attack comes from mismatched length handling? A: HTTP request smuggling.

#### Request Headers

the client sends Request Headers in an HTTP transaction (another RFC 2616 category). these headers are used in an HTTP request and do not relate to the content of the message. RFC 9110 no longer groups them together: most are defined next to the concept they relate to (`Host` in RFC 9110 §7.2, `Accept` under content negotiation, `Authorization` under HTTP Authentication, `Cookie` in RFC 6265), with only `Referer` and `User-Agent` left in RFC 9110's Request Context Fields section. the following headers are commonly seen in HTTP requests.

| Header | Example | Description |
|---|---|---|
| `Host` | `Host: www.inlanefreight.com` | used to specify the host being queried for the resource. this can be a domain name or an IP address. HTTP servers can be configured to host different websites, which are revealed based on the hostname. this makes the host header an important enumeration target, as it can indicate the existence of other hosts on the target server, and a common target for Host header injection attacks |
| `User-Agent` | `User-Agent: curl/7.77.0` | used to describe the client requesting resources. this header can reveal a lot about the client, such as the browser, its version, and the operating system |
| `Referer` | `Referer: http://www.inlanefreight.com/` | denotes where the current request is coming from. for example, clicking a link from google search results would make `https://google.com` the referer. trusting this header can be dangerous as it can be easily manipulated, leading to unintended consequences |
| `Accept` | `Accept: */*` | describes which media types the client can understand. it can contain multiple media types separated by commas. the `*/*` value signifies that all media types are accepted |
| `Cookie` | `Cookie: PHPSESSID=b4e4fbd93540` | contains cookie value pairs in the format `name=value`. a cookie is a piece of data stored on the client side and on the server, which acts as an identifier. these are passed to the server per request, thus maintaining the client's access. cookies can also serve other purposes, such as saving user preferences or session tracking. there can be multiple cookies in a single header, separated by a semicolon |
| `Authorization` | `Authorization: Basic cGFzc3dvcmQK` | another method for the server to identify clients. common schemes include `Basic`, `Bearer` (used for tokens such as JWTs), and `Digest`. unlike a cookie, the value is generated and attached by the client on every request rather than being stored and sent automatically by the browser |
| `X-Forwarded-For` | `X-Forwarded-For: 203.0.113.5` | added by proxies and load balancers to record the originating IP address of a client whose connection is being relayed. since it is client controllable, it can often be spoofed to bypass IP based access controls, rate limiting, or blocklists |

since fields are now scattered across multiple documents, the authoritative, continuously updated list of every registered HTTP header, request and response alike, is the IANA HTTP Field Name Registry, which links each name to the RFC that defines it.

Q: which request header is a spoofable IP claim? A: `X-Forwarded-For`.

#### Response Headers

Response Headers is the response side equivalent RFC 2616 category, used in an HTTP response and do not relate to the content of the message. `Location`, `Server`, `Allow`, and `Retry-After` are grouped under RFC 9110's Response Context Fields, while `Set-Cookie` is defined separately in RFC 6265 and `WWW-Authenticate` under HTTP Authentication. the following headers are commonly seen in HTTP responses.

| Header | Example | Description |
|---|---|---|
| `Server` | `Server: Apache/2.4.41 (Ubuntu)` | contains information about the HTTP server, which processed the request. it can be used to gain information about the server, such as its version, and enumerate it further. many hardening guides recommend suppressing or genericizing this header to reduce information disclosure |
| `Location` | `Location: https://www.inlanefreight.com/login` | used with redirect status codes (`3xx`) to tell the client where the requested resource can be found. if the redirect target is built from unvalidated user input, this header can lead to open redirect vulnerabilities |
| `Set-Cookie` | `Set-Cookie: PHPSESSID=b4e4fbd93540; Secure; HttpOnly; SameSite=Strict` | sends a cookie for the client to store and return on future requests, using the same `name=value` format as the `Cookie` request header. security relevant attributes can be appended: `Secure` (only send over HTTPS), `HttpOnly` (inaccessible to client side JavaScript, mitigating cookie theft via XSS), and `SameSite` (`Strict`, `Lax`, or `None`, restricting when the cookie is sent on cross site requests as a CSRF defense) |
| `WWW-Authenticate` | `WWW-Authenticate: Basic realm="localhost"` | notifies the client about the type of authentication required to access the requested resource, and is sent alongside a `401 Unauthorized` status code |

Q: which cookie attribute blocks JavaScript from reading it? A: `HttpOnly`.

#### Security Headers

Security Headers are a practical, industry coined grouping (popularized by the OWASP Secure Headers Project) for response headers that instruct the browser to follow certain security rules and policies.

| Header | Example | Description |
|---|---|---|
| `Content-Security-Policy` | `Content-Security-Policy: script-src 'self'` | dictates the website's policy towards externally injected resources. this could be JavaScript code as well as script resources. this header instructs the browser to accept resources only from certain trusted domains, hence preventing attacks such as Cross site scripting (XSS) |
| `Strict-Transport-Security` | `Strict-Transport-Security: max-age=31536000` | prevents the browser from accessing the website over the plaintext HTTP protocol, and forces all communication to be carried over the secure HTTPS protocol. this prevents attackers from sniffing web traffic and accessing protected information such as passwords or other sensitive data |
| `X-Content-Type-Options` | `X-Content-Type-Options: nosniff` | tells the browser to strictly follow the declared `Content-Type` and never try to guess (sniff) a resource's actual MIME type. without it, a browser may render a file as a different type than declared, which has historically been abused to turn file upload functionality into stored XSS |
| `X-Frame-Options` | `X-Frame-Options: DENY` | controls whether the page can be rendered inside a `<frame>`, `<iframe>`, or `<object>`, protecting against clickjacking attacks. largely superseded by the `frame-ancestors` directive in `Content-Security-Policy`, but still widely deployed for legacy browser support |
| `Access-Control-Allow-Origin` | `Access-Control-Allow-Origin: https://www.inlanefreight.com` | part of Cross Origin Resource Sharing (CORS), it tells the browser which origins are allowed to read the response to a cross origin request. misconfigurations, such as reflecting the requester's `Origin` header or using a wildcard (`*`) alongside credentials, are a common source of CORS based vulnerabilities |
| `Referrer-Policy` | `Referrer-Policy: origin` | dictates whether the browser should include the value specified via the `Referer` header or not. it can help in avoiding disclosing sensitive URLs and information while browsing the website |

> note: this section only mentions a small subset of commonly seen HTTP headers. there are many other contextual headers that can be used in HTTP communications. it's also possible for applications to define custom headers based on their requirements. a complete list of standard HTTP headers is in the IANA registry mentioned above.

Q: which header fights clickjacking? A: `X-Frame-Options`.

#### cURL

in the previous section i saw how using the `-v` flag with cURL shows me the full details of the HTTP request and response. if i'm only interested in seeing the response headers, i use the `-I` flag to send a `HEAD` request and only display the response headers. furthermore, i use the `-i` flag to display both the headers and the response body (e.g. HTML code). the difference between the two is that `-I` sends a `HEAD` request (as i see in the next section), while `-i` sends any request i specify and prints the headers as well.

the following command shows an example output of using the `-I` flag. this output is an illustrative composite assembled to demonstrate the headers discussed above; it isn't a literal capture, and a real target will only ever return the headers it's actually configured to send. i should try the command myself and compare:

```shellsession
cybermaxxer@htb[/htb]$ curl -I https://www.inlanefreight.com

Host: www.inlanefreight.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_5) AppleWebKit/605.1.15 (KHTML, like Gecko)
Cookie: cookie1=298zf09hf012fh2; cookie2=u32t4o3tb3gg4
Accept: text/plain
Referer: https://www.inlanefreight.com/
Authorization: Basic cGFzc3dvcmQK
X-Forwarded-For: 203.0.113.5

Date: Sun, 06 Aug 2020 08:49:37 GMT
Connection: keep-alive
Cache-Control: no-store
Content-Length: 26012
Content-Type: text/html; charset=UTF-8
Content-Encoding: gzip

Server: Apache/2.4.41 (Ubuntu)
Set-Cookie: sessionId=e8c2d9f1a4; Secure; HttpOnly; SameSite=Strict
WWW-Authenticate: Basic realm="localhost"

Content-Security-Policy: script-src 'self'
Strict-Transport-Security: max-age=31536000
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Access-Control-Allow-Origin: https://www.inlanefreight.com
Referrer-Policy: origin
```

> exercise for me: go through all of the above headers, and see whether i can recall the usage for each of them.

in addition to viewing headers, cURL also allows me to set request headers with the `-H` flag, as i see in a later section. some headers, like the `User-Agent` or `Cookie` headers, have their own flags. for example, i can use the `-A` flag to set my `User-Agent`, as follows:

```shellsession
cybermaxxer@htb[/htb]$ curl https://www.inlanefreight.com -A 'Mozilla/5.0'

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
...SNIP...
```

> exercise for me: try the `-I` or the `-v` flags with the above example, to ensure that i did change my User-Agent with the `-A` flag.

| flag | what it does |
|---|---|
| `-I` | sends a `HEAD` request, prints only response headers |
| `-i` | sends whatever request i specify, prints headers plus body |
| `-H` | sets a request header |
| `-A` | sets the `User-Agent` |

Q: which flag prints headers and body? A: `-i`.

#### Browser DevTools

i can preview the HTTP headers using the browser devtools. just as in the previous section, i go to the `Network` tab to view the different requests made by the page. i can click on any of the requests to view its details (screenshot in the module).

in the first tab, `Headers`, i see both the HTTP request and HTTP response headers. the devtools automatically arrange the headers into sections, but i can click on the `Raw` button to view them in their raw format. furthermore, i can check the `Cookies` tab to see any cookies used by the request, as discussed in an upcoming section.

Q: which button shows headers unformatted? A: Raw.

#### exercise questions

**question 1:** the server above loads the flag after the page is loaded. use the Network tab in the browser devtools to see what requests are made by the page, and find the request to the flag.

answer recorded in the module: `HTB{p493_r3qu3$t$_m0n!t0r}`

#### my personal actions

1. insert the url
2. go to the network tab via the `F12` button
3. reload the page
4. look for something that starts with the flag, there will be a txt file, double click it and find the flag

---

### 1.5 HTTP Methods and Codes

HTTP supports multiple methods for accessing a resource. in the HTTP protocol, several request methods allow the browser to send information, forms, or files to the server. these methods are used, among other things, to tell the server how to process the request i send and how to reply.

i saw different HTTP methods used in the HTTP requests i tested in the previous sections. with cURL, if i use `-v` to preview the full request, the first line contains the HTTP method (e.g. `GET / HTTP/1.1`), while with browser devtools, the HTTP method is shown in the `Method` column. furthermore, the response headers also contain the HTTP status code, which indicates the result of processing my HTTP request.

Q: where does devtools show the method? A: the Method column.

#### Request Methods

the following are some of the commonly used methods:

| Method | Description |
|---|---|
| `GET` | requests a specific resource. additional data can be passed to the server via query strings in the URL (e.g. `?param=value`) |
| `POST` | sends data to the server. it can handle multiple types of input, such as text, PDFs, and other forms of binary data. this data is appended in the request body present after the headers. the POST method is commonly used when sending information (e.g. forms/logins) or uploading data to a website, such as images or documents |
| `HEAD` | requests the headers that would be returned if a GET request was made to the server. it doesn't return the request body and is usually made to check the response length before downloading resources |
| `PUT` | creates new resources on the server. allowing this method without proper controls can lead to uploading malicious resources |
| `DELETE` | deletes an existing resource on the webserver. if not properly secured, can lead to Denial of Service (DoS) by deleting critical files on the web server |
| `OPTIONS` | returns information about the server, such as the methods accepted by it |
| `PATCH` | applies partial modifications to the resource at the specified location |

the list only highlights a few of the most commonly used HTTP methods. the availability of a particular method depends on the server as well as the application configuration. a full list of HTTP methods is available through the link in the module.

> note: most modern web applications mainly rely on the `GET` and `POST` methods. however, any web application that uses REST APIs also relies on `PUT` and `DELETE`, which are used to update and delete data on the API endpoint, respectively. the Introduction to Web Applications module has more details.

Q: which method checks response length without the body? A: `HEAD`.

#### Status Codes

HTTP status codes are used to tell the client the status of their request. an HTTP server can return five classes of status codes:

| Class | Description |
|---|---|
| `1xx` | provides information and does not affect the processing of the request |
| `2xx` | returned when a request succeeds |
| `3xx` | returned when the server redirects the client |
| `4xx` | signifies improper requests from the client. for example, requesting a resource that doesn't exist or requesting a bad format |
| `5xx` | returned when there is some problem with the HTTP server itself |

the following are some of the commonly seen examples from each of the above HTTP status code classes:

| Code | Description |
|---|---|
| `200 OK` | returned on a successful request, and the response body usually contains the requested resource |
| `302 Found` | redirects the client to another URL. for example, redirecting the user to their dashboard after a successful login |
| `400 Bad Request` | returned on encountering malformed requests such as requests with missing line terminators |
| `403 Forbidden` | signifies that the client doesn't have appropriate access to the resource. it can also be returned when the server detects malicious input from the user |
| `404 Not Found` | returned when the client requests a resource that doesn't exist on the server |
| `500 Internal Server Error` | returned when the server cannot process the request |

a full list of standard HTTP status codes is available through the link in the module. apart from the standard HTTP codes, various servers and providers such as Cloudflare or AWS implement their own codes.

Q: a `4xx` code means whose fault? A: the client's.

---

### 1.6 GET

whenever i visit any URL, my browser defaults to a GET request to obtain the remote resources hosted at that URL. once the browser receives the initial page it is requesting, it may send other requests using various HTTP methods. this can be observed through the Network tab in the browser devtools, as seen in the previous section.

> exercise for me: pick any website, and monitor the Network tab in the browser devtools as i visit it to understand what the page is performing. this technique can be used to thoroughly understand how a web application interacts with its backend, which can be an essential exercise for any web application assessment or bug bounty exercise.

Q: what method does a browser default to? A: GET.

#### HTTP Basic Auth

when i visit the exercise found at the end of this section, it prompts me to enter a username and a password. unlike the usual login forms, which use HTTP parameters to validate the user credentials (e.g. a POST request), this type of authentication uses a basic HTTP authentication, which is handled directly by the webserver to protect a specific page/directory, without directly interacting with the web application.

to access the page, i have to enter a valid pair of credentials, which are `admin : admin` in this case (the module shows the prompt at `http://<SERVER_IP>:<PORT>/`, look at it). once i enter the credentials, i get access to the page (second screenshot).

let me try to access the page with cURL, and i add `-i` to view the response headers:

```shellsession
cybermaxxer@htb[/htb]$ curl -i http://<SERVER_IP>:<PORT>/

HTTP/1.1 401 Authorization Required
Date: Mon, 21 Feb 2022 13:11:46 GMT
Server: Apache/2.4.41 (Ubuntu)
Cache-Control: no-cache, must-revalidate, max-age=0
WWW-Authenticate: Basic realm="Access denied"
Content-Length: 13
Content-Type: text/html; charset=UTF-8

Access denied
```

i get `Access denied` in the response body, and i also get `Basic realm="Access denied"` in the `WWW-Authenticate` header, which confirms that this page indeed uses basic HTTP auth, as discussed in the Headers section. to provide the credentials through cURL, i use the `-u` flag, as follows:

```shellsession
cybermaxxer@htb[/htb]$ curl -u admin:admin http://<SERVER_IP>:<PORT>/

<!DOCTYPE html>
<html lang="en">

<head>
...SNIP...
```

this time i do get the page in the response. there is another method to provide the basic HTTP auth credentials, which is directly through the URL as (`username:password@URL`), as discussed in the first section. if i try the same with cURL or my browser, i get access to the page as well:

```shellsession
cybermaxxer@htb[/htb]$ curl http://admin:admin@<SERVER_IP>:<PORT>/

<!DOCTYPE html>
<html lang="en">

<head>
...SNIP...
```

i can also visit the same URL in a browser, and i get authenticated as well.

> exercise for me: view the response headers by adding `-i` to the above request, and see how an authenticated response differs from an unauthenticated one.

Q: which header announces basic auth is needed? A: `WWW-Authenticate`.

#### HTTP Authorization Header

if i add the `-v` flag to either of my earlier cURL commands:

```shellsession
cybermaxxer@htb[/htb]$ curl -v http://admin:admin@<SERVER_IP>:<PORT>/

*   Trying <SERVER_IP>:<PORT>...
* Connected to <SERVER_IP> (<SERVER_IP>) port PORT (#0)
* Server auth using Basic with user 'admin'
> GET / HTTP/1.1
> Host: <SERVER_IP>
> Authorization: Basic YWRtaW46YWRtaW4=
> User-Agent: curl/7.77.0
> Accept: */*
>
* Mark bundle as not supporting multiuse

< HTTP/1.1 200 OK
< Date: Mon, 21 Feb 2022 13:19:57 GMT
< Server: Apache/2.4.41 (Ubuntu)
< Cache-Control: no-store, no-cache, must-revalidate
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Pragma: no-cache
< Vary: Accept-Encoding
< Content-Length: 1453
< Content-Type: text/html; charset=UTF-8
<
<!DOCTYPE html>
<html lang="en">

<head>
...SNIP...
```

as i am using basic HTTP auth, i see that my HTTP request sets the `Authorization` header to `Basic YWRtaW46YWRtaW4=`, which is the base64 encoded value of `admin:admin`. if i were using a modern method of authentication (e.g. JWT), the `Authorization` would be of type `Bearer` and would contain a longer encrypted token.

let me try to manually set the `Authorization`, without supplying the credentials, to see if it does allow me access to the page. i can set the header with the `-H` flag, and i use the same value from the above HTTP request. i can add the `-H` flag multiple times to specify multiple headers:

```shellsession
cybermaxxer@htb[/htb]$ curl -H 'Authorization: Basic YWRtaW46YWRtaW4=' http://<SERVER_IP>:<PORT>/

<!DOCTYPE html
<html lang="en">

<head>
...SNIP...
```

this also gave me access to the page. these are a few methods i can use to authenticate to the page. most modern web applications use login forms built with the back end scripting language (e.g. PHP), which use HTTP POST requests to authenticate the users and then return a cookie to maintain their authentication.

| way to send basic auth with curl | example |
|---|---|
| `-u` flag | `curl -u admin:admin http://<SERVER_IP>:<PORT>/` |
| credentials in the URL | `curl http://admin:admin@<SERVER_IP>:<PORT>/` |
| manual header with `-H` | `curl -H 'Authorization: Basic YWRtaW46YWRtaW4=' http://<SERVER_IP>:<PORT>/` |

Q: what is `YWRtaW46YWRtaW4=`? A: base64 of `admin:admin`.

#### GET Parameters

once i am authenticated, i get access to a `City Search` function, in which i enter a search term and get a list of matching cities (screenshot at `http://<SERVER_IP>:<PORT>/` in the module).

as the page returns my results, it may be contacting a remote resource to obtain the information, and then display them on the page. to verify this, i open the browser devtools and go to the Network tab, or use the shortcut `CTRL+SHIFT+E` to get to the same tab. before i enter my search term and view the requests, i may need to click on the `trash` icon on the top left, to ensure i clear any previous requests and only monitor newer requests (screenshot in the module).

after that, i enter any search term and hit enter, and i immediately notice a new request being sent to the backend (screenshot in the module).

when i click on the request, it gets sent to `search.php` with the GET parameter `search=le` used in the URL. this helps me understand that the search function requests another page for the results.

now, i can send the same request directly to `search.php` to get the full search results, though it will probably return them in a specific format (e.g. JSON) without having the HTML layout shown in the above screenshot.

to send a GET request with cURL, i can use the exact same URL seen in the above screenshots since GET requests place their parameters in the URL. however, browser devtools provide a more convenient method of obtaining the cURL command. i can right click on the request and select `Copy>Copy as cURL`. then, i paste the copied command in my terminal and execute it, and i get the exact same response:

```shellsession
cybermaxxer@htb[/htb]$ curl 'http://<SERVER_IP>:<PORT>/search.php?search=le' -H 'Authorization: Basic YWRtaW46YWRtaW4='

Leeds (UK)
Leicester (UK)
```

> note: the copied command will contain all headers used in the HTTP request. however, i can remove most of them and only keep necessary authentication headers, like the `Authorization` header.

i can also repeat the exact request right within the browser devtools, by selecting `Copy>Copy as Fetch`. this copies the same HTTP request using the JavaScript Fetch library. then, i go to the JavaScript console tab by clicking `CTRL+SHIFT+K`, paste my Fetch command and hit enter to send the request (screenshot in the module).

the browser sent my request, and i can see the response returned after it. i can click on the response to view its details, expand various details, and read them.

Q: where do GET parameters live? A: in the URL query string.

#### exercise questions

**question 1:** the exercise above seems to be broken, as it returns incorrect results. use the browser devtools to see what is the request it is sending when we search, and use cURL to search for 'flag' and obtain the flag. authenticate with user "admin" and password "admin".

answer recorded in the module: `HTB{curl_g3773r}`

#### my personal actions

1. go to the website, open dev tools (press `CTRL+SHIFT+E`), copy the http request that was invoked as a curl by right clicking on it and choosing the corresponding option, then curl the website with the credentials like this:

```bash
curl (url) -u login:password
```

2. try different cities, somehow curling yields correct results.
3. instead of the city, insert `flag` in the form (the thing that goes in the `?search=` part of the url like this):

```bash
curl http://(url)/search.php?search=flag -u login:password -v
```

4. call it a day.

---

### 1.7 POST

in the previous section, i saw how GET requests may be used by web applications for functionalities like search and accessing pages. however, whenever web applications need to transfer files or move the user parameters from the URL, they use POST requests.

unlike HTTP GET, which places user parameters within the URL, HTTP POST places user parameters within the HTTP request body. this has three main benefits:

* **Lack of Logging:** as POST requests may transfer large files (e.g. file upload), it would not be efficient for the server to log all uploaded files as part of the requested URL, as would be the case with a file uploaded through a GET request.
* **Less Encoding Requirements:** URLs are designed to be shared, which means they need to conform to characters that can be converted to letters. the POST request places data in the body which can accept binary data. the only characters that need to be encoded are those that are used to separate parameters.
* **More data can be sent:** the maximum URL length varies between browsers (Chrome/Firefox/IE), web servers (IIS, Apache, nginx), Content Delivery Networks (Fastly, Cloudfront, Cloudflare), and even URL shorteners (bit.ly, amzn.to). generally speaking, a URL's length should be kept below 2,000 characters, and so it cannot handle a lot of data.

so, let me see some examples of how POST requests work, and how i can use tools like cURL or browser devtools to read and send POST requests.

Q: why can POST carry more data than GET? A: body has no URL length limit.

#### Login Forms

the exercise at the end of this section is similar to the example i saw in the GET section. however, once i visit the web application, i see that it uses a PHP login form instead of HTTP basic auth (screenshot at `http://<SERVER_IP>:<PORT>/` in the module).

if i try to login with `admin : admin`, i get in and see a similar search function to the one i saw earlier in the GET section (screenshot).

if i clear the Network tab in my browser devtools and try to log in again, i see many requests being sent. i can filter the requests by my server IP, so it only shows requests going to the web application's web server (i.e. filter out external requests), and i notice a POST request being sent (screenshot in the module).

i can click on the request, click on the `Request` tab (which shows the request body), and then click on the `Raw` button to show the raw request data. i see the following data is being sent as the POST request data:

```
username=admin&password=admin
```

with the request data at hand, i can try to send a similar request with cURL, to see whether this would allow me to login as well. furthermore, as i did in the previous section, i can simply right click on the request and select `Copy>Copy as cURL`. however, it is important to be able to craft POST requests manually, so let me try to do so.

i use the `-X POST` flag to send a `POST` request. then, to add my POST data, i use the `-d` flag and add the above data after it, as follows:

```shellsession
cybermaxxer@htb[/htb]$ curl -X POST -d 'username=admin&password=admin' http://<SERVER_IP>:<PORT>/

...SNIP...
        <em>Type a city name and hit <strong>Enter</strong></em>
...SNIP...
```

if i examine the HTML code, i will not see the login form code, but will see the search function code, which indicates that i did indeed get authenticated.

> tip: many login forms would redirect me to a different page once authenticated (e.g. `/dashboard.php`). if i want to follow the redirection with cURL, i use the `-L` flag.

Q: which flag follows redirects? A: `-L`.

#### Authenticated Cookies

if i was successfully authenticated, i should have received a cookie so my browser can persist my authentication, and i don't need to login every time i visit the page. i can use the `-v` or `-i` flags to view the response, which should contain the `Set-Cookie` header with my authenticated cookie:

```shellsession
cybermaxxer@htb[/htb]$ curl -X POST -d 'username=admin&password=admin' http://<SERVER_IP>:<PORT>/ -i

HTTP/1.1 200 OK
Date:
Server: Apache/2.4.41 (Ubuntu)
Set-Cookie: PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1; path=/
...SNIP...
        <em>Type a city name and hit <strong>Enter</strong></em>
...SNIP...
```

with my authenticated cookie, i should now be able to interact with the web application without needing to provide my credentials every time. to test this, i set the above cookie with the `-b` flag in cURL, as follows:

```shellsession
cybermaxxer@htb[/htb]$ curl -b 'PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1' http://<SERVER_IP>:<PORT>/

...SNIP...
        <em>Type a city name and hit <strong>Enter</strong></em>
...SNIP...
```

i was indeed authenticated and got to the search function. it is also possible to specify the cookie as a header, as follows:

```bash
curl -H 'Cookie: PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1' http://<SERVER_IP>:<PORT>/
```

i may also try the same thing with my browser. let me first logout, and then i should get back to the login page. then, i go to the `Storage` tab in the devtools with `SHIFT+F9`. in the Storage tab, i click on `Cookies` in the left pane and select my website to view my current cookies. i may or may not have existing cookies, but if i was logged out, then my PHP cookie should not be authenticated, which is why i get the login form and not the search function (screenshot in the module).

now, let me try to use my earlier authenticated cookie, and see if i do get in without needing to provide my credentials. to do so, i simply replace the cookie value with my own. otherwise, i can right click on the cookie and select `Delete All`, and then click on the `+` icon to add a new cookie. after that, i enter the cookie name, which is the part before the `=` (`PHPSESSID`), and then the cookie value, which is the part after the `=` (`c1nsa6op7vtk7kdis7bcnbadf1`). then, once my cookie is set, i refresh the page, and i see that i do indeed get authenticated without needing to login, simply by using an authenticated cookie (screenshot in the module).

having a valid cookie may be enough to get authenticated into many web applications. this can be an essential part of some web attacks, like Cross Site Scripting.

| way to send the cookie | example |
|---|---|
| curl `-b` flag | `curl -b 'PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1' http://<SERVER_IP>:<PORT>/` |
| curl `-H` header | `curl -H 'Cookie: PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1' http://<SERVER_IP>:<PORT>/` |
| browser | devtools `Storage` tab (`SHIFT+F9`), replace the cookie value |

Q: which flag sends a cookie in curl? A: `-b`.

#### JSON Data

finally, let me see what requests get sent when i interact with the `City Search` function. to do so, i go to the Network tab in the browser devtools, and then click on the trash icon to clear all requests. then, i make any search query to see what requests get sent (screenshot in the module).

the search form sends a POST request to `search.php`, with the following data:

```json
{"search":"london"}
```

the POST data appear to be in JSON format, so my request must have specified the `Content-Type` header to be `application/json`. i can confirm this by right clicking on the request, and selecting `Copy>Copy Request Headers`:

```
POST /search.php HTTP/1.1
Host: server_ip
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:97.0) Gecko/20100101 Firefox/97.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://server_ip/index.php
Content-Type: application/json
Origin: http://server_ip
Content-Length: 19
DNT: 1
Connection: keep-alive
Cookie: PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1
```

indeed, i do have `Content-Type: application/json`. let me try to replicate this request as i did earlier, but include both the cookie and content type headers, and send my request to `search.php`:

```shellsession
cybermaxxer@htb[/htb]$ curl -X POST -d '{"search":"london"}' -b 'PHPSESSID=c1nsa6op7vtk7kdis7bcnbadf1' -H 'Content-Type: application/json' http://<SERVER_IP>:<PORT>/search.php

["London (UK)"]
```

i was able to interact with the search function directly without needing to login or interact with the web application front end. this can be an essential skill when performing web application assessments or bug bounty exercises, as it is much faster to test web applications this way.

> exercise for me: repeat the above request without adding the cookie or content type headers, and see how the web app would act differently.

finally, let me repeat the same above request by using Fetch, as i did in the previous section. i right click on the request and select `Copy>Copy as Fetch`, and then go to the `Console` tab and execute my code there (screenshot in the module).

my request successfully returns the same data i got with cURL.

> exercise for me: try to search for different cities by directly interacting with `search.php` through Fetch or cURL.

Q: which header tells the server the body is JSON? A: `Content-Type: application/json`.

#### exercise questions

**question 1:** obtain a session cookie through a valid login, and then use the cookie with cURL to search for the flag through a JSON POST request to `/search.php`. authenticate with user "admin" and password "admin".

answer recorded in the module: `HTB{p0$t_r3p34t3r}`

#### my personal actions

1. go to the website
2. open the network tab
3. log in
4. go to the storage tab
5. copy the cookie
6. write `london` in the search, copy as curl
7. insert it in the terminal, remove the user agent flag, edit the json search value to be `flag`
8. get the flag

---

### 1.8 CRUD API

i saw examples of a `City Search` web application that uses PHP parameters to search for a city name in the previous sections. this section looks at how such a web application may use APIs to perform the same thing, and i directly interact with the API endpoint.

Q: what am i talking to in this section? A: the API endpoint directly.

#### APIs

there are several types of APIs. many APIs are used to interact with a database, such that i can specify the requested table and the requested row within my API query, and then use an HTTP method to perform the operation needed. for example, for the `api.php` endpoint in my example, if i wanted to update the `city` table in the database, and the row i am updating has a city name of `london`, then the URL would look something like this:

```bash
curl -X PUT http://<SERVER_IP>:<PORT>/api.php/city/london
...SNIP...
```

Q: in `/api.php/city/london`, which part is the table? A: `city`.

#### CRUD

i can easily specify the table and the row i want to perform an operation on through such APIs. then i can use different HTTP methods to perform different operations on that row. in general, APIs perform 4 main operations on the requested database entity:

| Operation | HTTP Method | Description |
|---|---|---|
| Create | `POST` | adds the specified data to the database table |
| Read | `GET` | reads the specified entity from the database table |
| Update | `PUT` | updates the data of the specified database table |
| Delete | `DELETE` | removes the specified row from the database table |

these four operations are mainly linked to the commonly known CRUD APIs, but the same principle is also used in REST APIs and several other types of APIs. of course, not all APIs work in the same way, and the user access control will limit what actions i can perform and what results i can see. the Introduction to Web Applications module further explains these concepts.

mnemonic: **CRUD** is already the memory hook (Create, Read, Update, Delete), and the methods line up as POST, GET, PUT, DELETE in the same order.

Q: which method maps to Update? A: `PUT`.

#### Read

the first thing i do when interacting with an API is reading data. as mentioned earlier, i can simply specify the table name after the API (e.g. `/city`) and then specify my search term (e.g. `/london`), as follows:

```shellsession
cybermaxxer@htb[/htb]$ curl http://<SERVER_IP>:<PORT>/api.php/city/london

[{"city_name":"London","country_name":"(UK)"}]
```

the result is sent as a JSON string. to have it properly formatted in JSON format, i pipe the output to the `jq` utility, which formats it properly. i also silence any unneeded cURL output with `-s`, as follows:

```shellsession
cybermaxxer@htb[/htb]$ curl -s http://<SERVER_IP>:<PORT>/api.php/city/london | jq

[
  {
    "city_name": "London",
    "country_name": "(UK)"
  }
]
```

i got the output nicely formatted. i can also provide a search term and get all matching results:

```shellsession
cybermaxxer@htb[/htb]$ curl -s http://<SERVER_IP>:<PORT>/api.php/city/le | jq

[
  {
    "city_name": "Leeds",
    "country_name": "(UK)"
  },
  {
    "city_name": "Dudley",
    "country_name": "(UK)"
  },
  {
    "city_name": "Leicester",
    "country_name": "(UK)"
  },
  ...SNIP...
]
```

finally, i can pass an empty string to retrieve all entries in the table:

```shellsession
cybermaxxer@htb[/htb]$ curl -s http://<SERVER_IP>:<PORT>/api.php/city/ | jq

[
  {
    "city_name": "London",
    "country_name": "(UK)"
  },
  {
    "city_name": "Birmingham",
    "country_name": "(UK)"
  },
  {
    "city_name": "Leeds",
    "country_name": "(UK)"
  },
  ...SNIP...
]
```

> exercise for me: try visiting any of the above links using my browser, to see how the result is rendered.

Q: how do i list the whole table? A: end the URL with `/city/`.

#### Create

to add a new entry, i use an HTTP POST request, which is quite similar to what i did in the previous section. i simply POST my JSON data, and it will be added to the table. as this API is using JSON data, i also set the `Content-Type` header to JSON, as follows:

```shellsession
cybermaxxer@htb[/htb]$ curl -X POST http://<SERVER_IP>:<PORT>/api.php/city/ -d '{"city_name":"HTB_City", "country_name":"HTB"}' -H 'Content-Type: application/json'
```

now, i can read the content of the city i added (`HTB_City`), to see if it was successfully added:

```shellsession
cybermaxxer@htb[/htb]$ curl -s http://<SERVER_IP>:<PORT>/api.php/city/HTB_City | jq

[
  {
    "city_name": "HTB_City",
    "country_name": "HTB"
  }
]
```

a new city was created, which did not exist before.

> exercise for me: try adding a new city through the browser devtools, by using one of the Fetch POST requests i used in the previous section.

Q: which header must accompany JSON POST data? A: `Content-Type: application/json`.

#### Update

now that i know how to read and write entries through APIs, let me discuss two other HTTP methods i have not used so far: `PUT` and `DELETE`. as mentioned at the beginning of the section, `PUT` is used to update API entries and modify their details, while `DELETE` is used to remove a specific entity.

> note: the HTTP `PATCH` method may also be used to update API entries instead of `PUT`. to be precise, `PATCH` is used to partially update an entry (only modify some of its data, e.g. only `city_name`), while `PUT` is used to update the entire entry. i may also use the HTTP `OPTIONS` method to see which of the two is accepted by the server, and then use the appropriate method accordingly. in this section, i focus on the `PUT` method, though their usage is quite similar.

using `PUT` is quite similar to `POST` in this case, with the only difference being that i have to specify the name of the entity i want to edit in the URL, otherwise the API will not know which entity to edit. so, all i have to do is specify the `city name` in the URL, change the request method to `PUT`, and provide the JSON data like i did with POST, as follows:

```shellsession
cybermaxxer@htb[/htb]$ curl -X PUT http://<SERVER_IP>:<PORT>/api.php/city/london -d '{"city_name":"New_HTB_City", "country_name":"HTB"}' -H 'Content-Type: application/json'
```

in the example above i first specified `/city/london` as my city, and passed a JSON string that contained `"city_name":"New_HTB_City"` in the request data. so, the london city should no longer exist, and a new city with the name `New_HTB_City` should exist. let me try reading both to confirm:

```shellsession
cybermaxxer@htb[/htb]$ curl -s http://<SERVER_IP>:<PORT>/api.php/city/london | jq
```

```shellsession
cybermaxxer@htb[/htb]$ curl -s http://<SERVER_IP>:<PORT>/api.php/city/New_HTB_City | jq

[
  {
    "city_name": "New_HTB_City",
    "country_name": "HTB"
  }
]
```

indeed, i successfully replaced the old city name with the new city.

> note: in some APIs, the Update operation may be used to create new entries as well. basically, i send my data, and if it does not exist, it creates it. for example, in the above example, even if an entry with a `london` city did not exist, it would create a new entry with the details i passed. in my example, however, this is not the case. i should try to update a non existing city and see what i get.

Q: what does `PATCH` change that `PUT` doesn't? A: only part of the entry.

#### DELETE

finally, let me try to delete a city, which is as easy as reading a city. i simply specify the city name for the API and use the HTTP `DELETE` method, and it deletes the entry, as follows:

```shellsession
cybermaxxer@htb[/htb]$ curl -X DELETE http://<SERVER_IP>:<PORT>/api.php/city/New_HTB_City
```

```shellsession
cybermaxxer@htb[/htb]$ curl -s http://<SERVER_IP>:<PORT>/api.php/city/New_HTB_City | jq

[]
```

as i can see, after i deleted `New_HTB_City`, i get an empty array when i try reading it, meaning it no longer exists.

> exercise for me: try to delete any of the cities i added earlier through POST requests, and then read all entries to confirm that they were successfully deleted.

with this, i am able to perform all 4 CRUD operations through cURL. in a real web application, such actions may not be allowed for all users, or it would be considered a vulnerability if anyone can modify or delete any entry. each user would have certain privileges on what they can `read` or `write`, where `write` refers to adding, modifying, or deleting data. to authenticate my user to use the API, i would need to pass a cookie or an authorization header (e.g. JWT), as i did in an earlier section. other than that, the operations are similar to what i practiced in this section.

Q: an empty `[]` after DELETE means what? A: the entry is gone.

#### exercise questions

**question 1:** first, try to update any city's name to be 'flag'. then, delete any city. once done, search for a city named 'flag' to get the flag.

answer recorded in the module: `HTB{crud_4p!_m4n!pul4t0r}`

#### my personal actions

```bash
curl -X PUT http://url/london -d '{"city_name": "flag", "country_name": "flagland"}'
curl -X DELETE url/Glasgow
curl http://url/flag
```

---

## 2. Structured Breakdown (Cheat Sheet)

| Concept | Explanation | Practical Usage |
|---|---|---|
| URL anatomy | scheme + optional user info + host + port + path + query string + fragment, only scheme and host are mandatory | read any URL fast, spot credentials in `user:pass@host` and parameters after `?` |
| HTTP flow | DNS resolves the domain (after `/etc/hosts`), browser sends `GET /`, server returns `index.html` with `200 OK` | add test domains to `/etc/hosts`, explains why a request needs an IP first |
| HTTP vs HTTPS | HTTP is clear text, HTTPS encrypts after a hello, key exchange, certificate check | MiTM on public wifi reads HTTP logins, HTTPS hides them (but clear text DNS still leaks the domain) |
| `curl -k` | skips the SSL certificate check | testing practice targets with invalid or missing certificates |
| `curl -v` / `-i` / `-I` | `-v` shows request and response, `-i` headers plus body, `-I` sends `HEAD` and shows headers only | grab `Server: Apache/2.4.41` version or check what a server sends back |
| `curl -O` / `-o` / `-s` | save with remote name, save with my name, silence progress | download files in scripts without noise |
| request and response anatomy | first line (method path version or version code), then headers, blank line, optional body | craft or debug raw requests, know where each part sits |
| header families | general, entity, request, response, security (a learning model, RFC 9110 groups by function) | `Server` leaks versions, `Set-Cookie` flags protect sessions, `X-Forwarded-For` is spoofable |
| methods | GET read, POST send data in body, HEAD headers only, PUT create/replace, DELETE remove, OPTIONS list allowed, PATCH partial update | pick the right verb, and notice dangerous ones left open (PUT, DELETE) |
| status codes | 1xx info, 2xx success, 3xx redirect, 4xx client error, 5xx server error | `401` means auth needed, `403` forbidden, `404` missing, `301` is the http to https redirect |
| basic auth | `Authorization: Basic base64(user:pass)`, sent by `-u`, `user:pass@` in the URL, or `-H` | recognize `WWW-Authenticate: Basic`, decode base64 creds from captured traffic |
| GET parameters | parameters in the query string, visible in devtools Network tab | `Copy as cURL` from devtools to replay and edit a search request |
| POST with cookies and JSON | body holds data, `Set-Cookie` returns a session, `-b` replays it, `Content-Type: application/json` for JSON bodies | log in once, then script the backend directly without the front end |
| CRUD API | POST create, GET read, PUT update, DELETE remove on `/api.php/table/row` | read, edit and delete entries from the terminal, check which actions need auth |

---

## 3. Feynman Checks

1. **a page returns `401 Unauthorized` with `WWW-Authenticate: Basic realm="Access denied"`. name three ways to get in with curl, and explain why the manual header way works without ever typing the password.** → `curl -u admin:admin <url>`, `curl http://admin:admin@<url>`, or `curl -H 'Authorization: Basic YWRtaW46YWRtaW4=' <url>`. the header works because basic auth is just `base64(user:password)` attached to every request, so anyone who has the encoded string has the credentials. base64 is encoding, not encryption, which is also why this is only safe over HTTPS.

2. **a search box returns wrong results in the browser. how do you find the real request behind it, and how do you decide between a GET style replay and a JSON POST style replay?** → open devtools, go to the Network tab, clear it, run a search, and click the new request. if the URL carries `?search=le` it is a GET, so copy it as cURL and edit the parameter. if the `Request` tab shows `{"search":"london"}` and the headers show `Content-Type: application/json`, it is a POST, so send `-X POST -d '{"search":"flag"}'` with the `Content-Type` header and the session cookie (`-b`), because without those the server won't accept or recognize the request.

3. **on a CRUD API, you `PUT` to `/api.php/city/london` with a new `city_name`, then read `/city/london`. what do you get, and what would change if the API treated update as create too?** → an empty array `[]`, because `PUT` replaced the whole entry, so the old key `london` no longer exists and the new name does. if the API allowed update to create, a `PUT` to a non existing city would silently add a new entry instead of failing, which matters in testing because it widens what an unauthenticated or low privileged user could write.

---

## 4. Omitted Chuff

nothing from the module's theory, tables or examples was cut. what i left out is page furniture, not content:

* the academy pricing banner ("pricing is being updated on Oct 12th", "lock in annual rates", "learn more")
* "Go to Questions" links, the PDF timestamps, page URLs and page counters
* the "Spawn the target system", "Enable step by step solutions PRO" and "Hint" buttons
* the "N / 8 Sections Completed" progress counters
* the screenshots themselves (i point to each one in the text where it matters)

Q: did i drop any theory or command? A: no, only page furniture.
