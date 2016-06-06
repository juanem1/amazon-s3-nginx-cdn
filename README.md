# Amazon S3 Proxy
Serve amazon s3 public files from own server and ssl

## Public Files

```
server {

    include /etc/nginx/h5bp/basic.conf;

    include amazon-s3-proxy.conf;
}
```

amazon-s3-proxy.conf
```
location ~* ^/cdn/(.*) {
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
```

Example: https://cdn.your_server/readme.txt



