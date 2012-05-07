---
layout: post
title: "ParseError - Premature EOF - Java WebServices Client"
date: 2012-05-06 20:12
comments: true
categories: java webservices
author: Alex Moffat <alex.moffat@gmail.com>
---

Quick post to document the solution, without the underlying root cause, to an exception. 
The setup is a client implemented with the JAX-WS reference implementation in Java 6 (JAX-WS RI 2.1.6 in JDK 6) calling a server also using JAX-WS hosted by Jetty using the [jetty-jaxws2spi package](http://jectbd.com/?p=1624).
Setup as normal I get an exception in the client when it tries to parse the response from the server.

```
com.sun.xml.internal.ws.streaming.XMLStreamReaderException: XML reader error: javax.xml.stream.XMLStreamException: ParseError at [row,col]:[1,507]
Message: Premature EOF
```

<!-- more -->

I used the [TCPProxy from The Grinder](http://grinder.sourceforge.net/g3/tcpproxy.html) to look at the request and response from the communication.
In the output below I've obscured some of the namespaces and element names but kept the content length the same.
You can see that the response is being sent back chunked, in two chunks.

```
--- localhost:49970->127.0.0.1:8070 opened --
--- 127.0.0.1:8070->localhost:49970 opened --
------ localhost:49970->127.0.0.1:8070 ------
POST /serviceAddress HTTP/1.1
Content-type: text/xml;charset="utf-8"
Soapaction: "sendAuthorizationEmail"
Accept: text/xml, multipart/related, text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2
User-Agent: JAX-WS RI 2.1.6 in JDK 6
Host: localhost:8071
Connection: keep-alive
Content-Length: 523


------ localhost:49970->127.0.0.1:8070 ------
<?xml version="1.0" ?><S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/"><S:Body><ns2:sendAuthorizationEmailRequest xmlns:ns2="http://___________________________________________________/types" xmlns:ns3="http://___________________________________________________/faults"><sendAuthorizationEmailRequest><partyId>mem001</partyId><logicalID>NOT REQUIRED</logicalID><dynamicContent>Some content for an email</dynamicContent></sendAuthorizationEmailRequest></ns2:sendAuthorizationEmailRequest></S:Body></S:Envelope>
------ 127.0.0.1:8070->localhost:49970 ------
HTTP/1.1 200 OK
Content-Type: text/xml;charset=UTF-8
Transfer-Encoding: chunked
Server: Jetty(8.1.3.v20120416)

5E
<?xml version="1.0" ?><S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/"><S:Body>

------ 127.0.0.1:8070->localhost:49970 ------
19C
<ns2:sendAuthorizationEmailResponse xmlns:ns2="http://___________________________________________________/types" xmlns:ns3="http://___________________________________________________/faults"><sendAuthorizationEmailResponse><XXXXMessageType><messageCode>OK</messageCode><messageReason>OK</messageReason></XXXXMessageType></sendAuthorizationEmailResponse></ns2:sendAuthorizationEmailResponse></S:Body></S:Envelope>

--- 127.0.0.1:8070->localhost:49970 closed --
--- localhost:49970->127.0.0.1:8070 closed --
```

Eventually, after much poking around, made more difficult by lack of source code, I decided it might be that the client code could not handle the chunked response from the server. 
To tell the server that the client didn't want / couldn't handle chunked I needed to add the "Connection: close" header to the request. 
As with all things in the Java WebServices world this was not easy, involving poorly documented options that all seem to be controlled by passing around untyped Maps.
Here the magic incantation was

```
RequestContext ctx = ((BindingProvider) service).getRequestContext();
ctx.put(MessageContext.HTTP_REQUEST_HEADERS,
    Collections.singletonMap("Connection", Collections.singletonList("close")));
```

Setting this changed the response from the server, as shown below, and the client was able to successfully parse the result.

```
--- localhost:51386->127.0.0.1:8070 opened --
--- 127.0.0.1:8070->localhost:51386 opened --
------ localhost:51386->127.0.0.1:8070 ------
POST /serviceAddress HTTP/1.1
Content-type: text/xml;charset="utf-8"
Connection: close
Soapaction: "sendAuthorizationEmail"
Accept: text/xml, multipart/related, text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2
User-Agent: JAX-WS RI 2.1.6 in JDK 6
Host: localhost:8071
Content-Length: 523

<?xml version="1.0" ?><S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/"><S:Body><ns2:sendAuthorizationEmailRequest xmlns:ns2="http://___________________________________________________/types" xmlns:ns3="http://___________________________________________________/faults"><sendAuthorizationEmailRequest><partyId>mem001</partyId><logicalID>NOT REQUIRED</logicalID><dynamicContent>Some content for an email</dynamicContent></sendAuthorizationEmailRequest></ns2:sendAuthorizationEmailRequest></S:Body></S:Envelope>
------ 127.0.0.1:8070->localhost:51386 ------
HTTP/1.1 200 OK
Content-Type: text/xml;charset=UTF-8
Connection: close
Server: Jetty(8.1.3.v20120416)

<?xml version="1.0" ?><S:Envelope xmlns:S="http://schemas.xmlsoap.org/soap/envelope/"><S:Body>
------ 127.0.0.1:8070->localhost:51386 ------
<ns2:sendAuthorizationEmailResponse xmlns:ns2="http://___________________________________________________/types" xmlns:ns3="http://___________________________________________________/faults"><sendAuthorizationEmailResponse><XXXXMessageType><messageCode>OK</messageCode><messageReason>OK</messageReason></XXXXMessageType></sendAuthorizationEmailResponse></ns2:sendAuthorizationEmailResponse></S:Body></S:Envelope>
--- 127.0.0.1:8070->localhost:51386 closed --
--- localhost:51386->127.0.0.1:8070 closed --

```