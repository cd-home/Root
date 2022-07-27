[TOC]

### Module

#### HTTP Load Balancing

~~~nginx
http {
    # Maybe Include From conf.d http
    upstream backend {
        server xxip:port;
    } 
    server {
        location / {
            proxy_pass http://backend;
        }
    }
}
~~~

#### HTTP UDP Load Balancing

~~~nginx
stream {
    upstream stream_backend {
        
    }
    
    server {
        listen ;
        proxy_pass ;
    }
}
~~~

