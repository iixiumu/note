The following load balancing mechanisms (or methods) are supported in nginx:

round-robin — requests to the application servers are distributed in a round-robin fashion,
least-connected — next request is assigned to the server with the least number of active connections,
ip-hash — a hash-function is used to determine what server should be selected for the next request (based on the client’s IP address).

Reverse proxy implementation in nginx includes load balancing for HTTP, HTTPS, FastCGI, uwsgi, SCGI, memcached, and gRPC.

To configure load balancing for HTTPS instead of HTTP, just use “https” as the protocol.

When setting up load balancing for FastCGI, uwsgi, SCGI, memcached, or gRPC, use fastcgi_pass, uwsgi_pass, scgi_pass, memcached_pass, and grpc_pass directives respectively.

When the load balancing method is not specifically configured, it defaults to round-robin.
```
http {
    upstream myapp1 {
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://myapp1;
        }
    }
}
```
Least connected
```
    upstream myapp1 {
        least_conn;
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }
```
Session persistence
```
upstream myapp1 {
    ip_hash;
    server srv1.example.com;
    server srv2.example.com;
    server srv3.example.com;
}
```
Weighted load balancing
```
    upstream myapp1 {
        server srv1.example.com weight=3;
        server srv2.example.com;
        server srv3.example.com;
    }
```