Reference: https://interviewnoodle.com/system-design-basics-proxy-vs-reverse-proxy-90d48da385be
- Web server 
- Reverse Proxy
- Load Balancer
### Proxy Server - Proxy server hides the identity of client from the server by sending requests on behalf of it.
- is a server that routes traffic between client(s) and another system, usually external to the network. 
- By doing so, it can regulate traffic according to preset policies, convert and mask client IP addresses, enforce security protocols, and block unknown traffic.

### Benefits
- Caching
- Anonymity
- Traffic control
- Logging
- Load Balancing
- Experimentations - Testing new features for a selected user base
- Router/Ingress - The reverse proxy can act as an ingress or a router in Kubernetes or micro-services architecture. It can map to actual services running for example if a client requests playlists it directs to YouTube server1 which has the service running for playlists.
- Encryption - enc/dec server responses
### Forward Proxy
- Proxy (also known as forward proxy) is a **server that makes “requests” on behalf of a client**, thus anonymizing the client from the server. With a proxy server, the server doesn’t know the client.
- is used to protect clients,
### Reverse Proxy - the reverse proxy server hides the final server that served the request from the client.
-  a reverse proxy is used to protect servers. 
- acts on behalf of the server
- server that accepts a request from a client, forwards the request to another one of many other servers, and returns the results from the server that actually processed the request to the client as if the proxy server had processed the request itself. 
- The client only communicates directly with the reverse proxy server and it does not know that some other server actually processed its request.
- Reverse proxies forward request to one or more ordinary servers that handle the request. The response from the reverse proxy server is returned as if it came directly from the original server, leaving the client with no knowledge of the original server.

### Terminologies
Directives  - key values pairs
Context - context in which the directives exist

### Keywords

types
mime.types
http
event
server
listen
root
location
alias
try_files
redirect using: return 307 /slug
rewrite using : ![[nginx rewrite directive.png]]

Load Balancing
```
// registering 4 different server for my backend service in nginx conf
upstream beServer{
 server 127.0.0.1:1111;
 server 127.0.0.1:2222;
 server 127.0.0.1:3333;
 server 127.0.0.1:4444;
}

// asking nginx to load 4 servers in round robin fashion when hitting the base path
location / {
 proxy_pass http://beServer/;
}

```