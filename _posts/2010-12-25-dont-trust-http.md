---
layout: post
title: Don't Trust HTTP
---

Spurred by the the number of times I have been burned by buggy HTTP stack, I give you the following advice:

  Do not trust HTTP stacks

When you encounter an application that implements its own HTTP stack, assume it is extremely buggy and is missing support for features mandated by RFC 2616, such as chunked transfer encoding and compressible content encoding. Assume it is prone to trival DoS attacks, such as streaming header content at 1 byte/s.

When you encounter popular libraries, still do not trust these. They are likely better, but also contain many bugs.

# Examples

## .NET Ignores Expect: 100-Continue

When performing POST requests with System.Net.HttpWebRequest, the HTTP client will issue a HTTP request with a *Expect: 100-continue* header. This effectively says, *server: tell me via a 100 response code that you are willing to process the data I'm about to send.* Good idea, especially if it saves the client from transferring something the server won't accept. However, no matter what the response from the server, the client will send the body. e.g.

<pre>
POST /upload HTTP/1.1
Host: foo.com
Content-Length: 105546
Expect: 100-continue

HTTP/1.1 404 Not Found
Date: Thu, 22 Jul 2010 23:36:39 GMT
Vary: Accept-Encoding
Content-Length: 212
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL /upload was not found on this server.</p>
</body></html>

{"ver":"0.1", ...
</pre>

Since the server just told the client, "I don't want your data, go away" it thinks it is done with the request. When the client sends the data, the server is looking for a HTTP request line (e.g. "GET / HTTP/1.1"), which the data likely doesn't resemble, so the server will likely error with a 400 Bad Request response. NOT COOL.
