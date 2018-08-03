# Spring Boot 2.0 with HTTP/2

## Prerequisites
* Java 8
* Spring Boot > 2.0
* Servlet Container which supports HTTP/2
    * Current starters do not support HTTP/2 out of the box (except undertow)
    * You need to upgrade/enhance the configuration yourself as follows  
* HTTPS is a must for HTTP/2

## Spring Boot restrictions
Spring Boot only supports the ciphertext transmission part of HTTP2 (using HTTPS) and does not support the plaintext transmission part of HTTP2 (h2c, HTTP/2 cleartext version).
Therefore, your Spring Boot application must support HTTPS and your Web container must be configured to support SSL.

## Configure the web container
### Undertow
Undertow 1.4.0+, if you use Java 8, supports HTTP2 without any additional configuration.

### Jetty
Jetty 9.4.11, if you use Java 8, needs the conscrypt library, with two following Jetty modules.

```
        <dependency>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-alpn-conscrypt-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jetty.http2</groupId>
            <artifactId>http2-server</artifactId>
        </dependency>
```

### Tomcat
Tomcat 9.0.x, if you use Java 8, needs the libtcnative class library separately 
(using the APR connector and setting the upgrade protocol to Http2Protocol), and the startup parameters 
for adding the JVM when starting Tomcat.

```bash
-Djava.library.path=/usr/local/opt/tomcat-native/lib
```
Tomcat 9.0.x, if you use Java 9, doesn't need any additional configuration to support HTTP2.

## Validating HTTP/2

```bash

$ curl -vk --http2 https://localhost:8443/hello                                                                     
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* Cipher selection: ALL:!EXPORT:!EXPORT40:!EXPORT56:!aNULL:!LOW:!RC4:@STRENGTH
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/cert.pem
  CApath: none
* TLSv1.2 (OUT), TLS handshake, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Server hello (2):
* TLSv1.2 (IN), TLS handshake, Certificate (11):
* TLSv1.2 (IN), TLS handshake, Server key exchange (12):
* TLSv1.2 (IN), TLS handshake, Server finished (14):
* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):
* TLSv1.2 (OUT), TLS change cipher, Client hello (1):
* TLSv1.2 (OUT), TLS handshake, Finished (20):
* TLSv1.2 (IN), TLS change cipher, Client hello (1):
* TLSv1.2 (IN), TLS handshake, Finished (20):
* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256
* ALPN, server accepted to use h2
* Server certificate:
*  subject: C=CH; ST=ZH; L=Zurich; O=42talents GmbH; OU=Spring Boot; CN=Patrick Baumgartner
*  start date: Aug  3 17:39:10 2018 GMT
*  expire date: Jul 29 17:39:10 2019 GMT
*  issuer: C=CH; ST=ZH; L=Zurich; O=42talents GmbH; OU=Spring Boot; CN=Patrick Baumgartner
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x7fa568802600)
> GET /hello HTTP/2
> Host: localhost:8443
> User-Agent: curl/7.54.0
> Accept: */*
> 
* Connection state changed (MAX_CONCURRENT_STREAMS updated)!
< HTTP/2 200 
< date: Fri, 03 Aug 2018 17:41:08 GMT
< content-type: text/plain;charset=utf-8
< content-length: 6
< 
* Connection #0 to host localhost left intact
Hello!%  

```