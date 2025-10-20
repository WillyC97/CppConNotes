# HTTP and the Web

## HTTP

* **HTTP** stands for HyperText Transfer Protocol
* The **HTTP** is an **application layer protocol** that allows web-based applications to communicate and exchange data
* It is the messenger of the web
* It is as **TCP**/**IP based** protocol
* It used to deliver contents


* Client makes a request to the server
* Server has a response


#### Basic steps
1. Client makes a request
2. Server says "*Do I have the files the client is requesting*"
3. Server will send a status code back
  * **1xx**: Informational
  * **2xx**: Success
  * **3xx**: Redirection
  * **4xx**: Client error
  * **5xx**: Server error


#### GET Header
* A request gets sent through a header:

```
            Method       Target         Version
              |            |               |
              v            v               v
Start Line:  GET/    background.png      HTTP/1.0

Headers:     User-Agent: Mozilla/5.0

Body:        None For GET
```

* The server gives the response

```
              Version  Status code Status Text
                |         |            |
                v         v            v
Start Line:  HTTP/1.0    404        Not Found

Headers:     Content-Type: Image/Png

Body:        The file/resource requested
```

* HTTP 1.0 is Stateless
  * When the client sends a request, and the server sends a response, the connection is broken

* Based on TCP/IP Address


## IP Address

IPV4: 32 bits

IPv6: 128 bits


### Domain names

* TLD - Top level Domain
  * .com
  * .net
  * .org

* CCTLD - Country code TLD
  * .uk
  * .mx
  * .cn


### DNS (Domain Name Server)

* maps a domain name to an IP Address

Client -> router -> DNS Server -> router -> server
