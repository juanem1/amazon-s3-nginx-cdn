# Amazon S3 Proxy
Serve amazon s3 public files from own server with ssl

```
server {
	listen 442;
	server_name cdn.domain.com;
	root /home/cdn.domain.com/public;

	ssl_certificate /etc/nginx/ssl/server.crt;
    ssl_certificate_key /etc/nginx/ssl/server.key;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

	charset utf-8;

	# Improve performance
	sendfile           on;
	sendfile_max_chunk 1m;
	tcp_nodelay        on;
	keepalive_timeout  65;


	access_log off;
	error_log  /var/log/nginx/cdn.domain.com-error.log error;
	error_page 403 /404.html;
	error_page 404 /403.html;
	error_page 500 /500.html;

	location = /favicon.ico { access_log off; log_not_found off; }
	location = /robots.txt  { access_log off; log_not_found off; }
	location = /403.html    { access_log off; log_not_found off; }
	location = /404.html    { access_log off; log_not_found off; }
	location = /500.html    { access_log off; log_not_found off; }

	# cache.appcache, your document html and data
	location ~* \.(?:manifest|appcache|html?|xml|json)$ {
		expires -1;
		try_files $uri @s3;
	}

	# Media: images, icons, video, audio, HTC
	location ~* \.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|svgz|mp4|ogg|ogv|webm|htc)$ {
		expires 1M;
		add_header Cache-Control "public";
		try_files $uri @s3;
	}

	# CSS and Javascript
	location ~* \.(?:css|js)$ {
		expires 1y;
		try_files $uri @s3;
	}

	# WebFonts
	location ~* \.(?:ttf|ttc|otf|eot|woff|woff2)$ {
		expires 1M;
		try_files $uri @s3;
	}

	# Main proxy config
	location @s3 {
		proxy_http_version     1.1;
		proxy_set_header       Host 'bucket.s3-website-sa-east-1.amazonaws.com';
		proxy_set_header       Authorization '';
		proxy_hide_header      x-amz-id-2;
		proxy_hide_header      x-amz-request-id;
		proxy_hide_header      Set-Cookie;
		proxy_ignore_headers   'Set-Cookie';
		proxy_buffering        off;
		proxy_intercept_errors on;
		proxy_method           GET;
		resolver               172.16.0.23 valid=300s;
		resolver_timeout       5s;
		proxy_pass             http://bucket.s3-website-sa-east-1.amazonaws.com;
	}

}

```

Example: https://cdn.domain.com/file-in-s3.txt



