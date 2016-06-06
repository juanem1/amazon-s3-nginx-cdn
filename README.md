# Amazon S3 Proxy
Serve amazon s3 public files from own server with ssl

```
server {
    listen 442 ssl;
    server_name cdn.domain.com;
    root /home/domain.com/public;
    
    include /etc/nginx/h5bp/basic.conf;

    ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

    index index.html index.htm index.php;

    charset utf-8;
    
    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log off;
    error_log  /var/log/nginx/domain.com-error.log error;
    error_page 404 /index.php;
    
    location / {
        set $s3_bucket        'your_bucket.s3.amazonaws.com';
        set $url_full         '$1';
        
        proxy_http_version     1.1;
        proxy_set_header       Host $s3_bucket;
        proxy_set_header       Authorization '';
        proxy_hide_header      x-amz-id-2;
        proxy_hide_header      x-amz-request-id;
        proxy_hide_header      Set-Cookie;
        proxy_ignore_headers   'Set-Cookie';
        proxy_buffering        off;
        proxy_intercept_errors on;
        
        resolver               172.16.0.23 valid=300s;
        resolver_timeout       10s;
        
        proxy_pass             http://$s3_bucket/$url_full;
    }
}
```

Example: https://cdn.domain.com/file-in-s3.txt



