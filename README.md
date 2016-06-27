# Configure Nginx as CDN with Amazon S3 in your own server and ssl


```
server {
	listen 442;
	server_name cdn.domain.com;
	root /home/cdn.domain.com/public;

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
	ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
	ssl_prefer_server_ciphers on;
	ssl_dhparam /etc/nginx/dhparams.pem;

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



