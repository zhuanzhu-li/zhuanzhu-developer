~~~

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;
    #gzip  on;
    underscores_in_headers on;



    # HTTPS server
    #
    server {
        listen       36180;
        server_name  xxx;
        ssl_certificate /home/dgisuser/nginx-1.4.7/ssl/server.crt;
        ssl_certificate_key /home/dgisuser/nginx-1.4.7/ssl/server.key;
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;
        location / {
            proxy_pass  https://135.32.117.214:26180;
            proxy_ssl_verify off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_redirect https://135.32.117.214:26180/ /;
        }
    }
    server {
        listen       30443;
        server_name  xxx;
        ssl_certificate /home/dgisuser/nginx-1.4.7/ssl/server.crt;
        ssl_certificate_key /home/dgisuser/nginx-1.4.7/ssl/server.key;
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;

        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;
        location / {
            proxy_pass  https://xxxx:30443;
            proxy_ssl_verify off;
            proxy_set_header Host xxxx;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_redirect https://xxxx:30443/ /;
        }
    }
    server {
        listen       30043 ;
        server_name  xxx;

        location / {
            proxy_pass  https://xxxx:30043;
            proxy_ssl_verify off;
            proxy_set_header Host xxxx;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_redirect https://xxxx:30043/ /;
        }
    }
    server {
        listen       28080 ;
        server_name  xxx;
        ssl_session_timeout  5m;
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;
        location / {
            proxy_pass  http://xxxx:28080;
        }
    }
    server {
        listen       18080 ;
        server_name  xxx;
        location / {
            proxy_pass  http://xxxx:18080;
        }
    }
    server {
        listen       35762 ;
        server_name  xxx;
        location / {
            proxy_pass  http://xxxx:15672;
        }
    }

}

~~~

* 生成证书
~~~
openssl genrsa -des3 -out server.key 2048
openssl req -new -key server.key -out server.csr
cp server.key server.key.org
openssl rsa -in server.key.org -out server.key
openssl x509 -req -days 3650 -in server.csr -signkey server.key -out server.crt
~~~