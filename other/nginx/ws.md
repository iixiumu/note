To turn a connection between a client and server from HTTP/1.1 into WebSocket, the protocol switch mechanism available in HTTP/1.1 is used.
By default, the connection will be closed if the proxied server does not transmit any data within 60 seconds. This timeout can be increased with the proxy_read_timeout directive. Alternatively, the proxied server can be configured to periodically send WebSocket ping frames to reset the timeout and check if the connection is still alive.
```
http {
    map $http_upgrade $connection_upgrade {
        default upgrade;
        ''      close;
    }

    server {
        ...

        location /chat/ {
            proxy_pass http://backend;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
        }
    }
```
