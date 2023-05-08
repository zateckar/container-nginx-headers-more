# Nginx container image with Headers more as dynamic module based on official stable Nginx docker image 
For more info about the "Headers more" module check: https://github.com/openresty/headers-more-nginx-module


### Get the latest stable image based on Debian
```
docker pull ghcr.io/zateckar/container-nginx-headers-more:stable
```


## Nginx configuration examples

### Load module

```
load_module /usr/lib/nginx/modules/ngx_http_headers_more_filter_module.so;

http {
    ...
}
```


### Change server header

```
http {
    ...
    more_set_headers 'Server: 1337-server';
    ...
}
```

### Remove server header

```
http {
    ...
    more_clear_headers	Server;
    ...
}
```



### Full example of a Nginx serving as a reverse proxy to one upstream

```
user root;
worker_processes auto;
pid /tmp/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
load_module /usr/lib/nginx/modules/ngx_http_headers_more_filter_module.so;

events {
    worker_connections 1024;
}

http {
	server_tokens			off;
	more_clear_headers		Server;
	underscores_in_headers	on;

	limit_req_zone $binary_remote_addr zone=ext:10m rate=500r/s;
	limit_req_zone $binary_remote_addr zone=dummy:10m rate=1r/m;
	limit_req_status 429;

	proxy_buffering 			off;
	proxy_request_buffering			off;
	proxy_headers_hash_max_size		1024;
	proxy_headers_hash_bucket_size		128;
  	proxy_read_timeout			130s;
	client_max_body_size			40m;

	access_log   off;
	error_log    /var/log/nginx/error.log	warn;

	include              /etc/nginx/mime.types;
	default_type         application/octet-stream;

	ssl_verify_client        optional_no_ca;
	ssl_protocols            TLSv1.2 TLSv1.3;
	ssl_ciphers              ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
	ssl_session_cache        shared:SSL:80m;
	ssl_session_timeout      1h;
	ssl_dhparam              /etc/ssl/dhparam.pem;
	ssl_trusted_certificate  /etc/ssl/certs/ca-certificates.crt;

	map $ssl_client_s_dn $ssl_client_s_dn_cn {
        default "";
        ~(^|,)CN=(?<CN>[^,]+) $CN;
	}

	upstream gw {
		server			127.0.0.1:8443;
		keepalive 2;
	}
	
	
	server {
		listen       443 ssl http2  default_server;
		server_name  _;
	
		ssl_certificate			/etc/nginx/conf/dummy.pem;
		ssl_certificate_key		/etc/nginx/conf/dummy.pem;
		limit_req zone=dummy;

		error_page 401 403 404 = @400;
		location @400 { return 400 '\n'; }
		
		return       404;
	}

	server {
		listen 443 ssl http2 reuseport;
		server_name myserver.example.com;

		ssl_certificate				/etc/nginx/conf/myserver.example.com.pem;
		ssl_certificate_key			/etc/nginx/conf/myserver.example.com.key;

		error_page 401 403 404 = @400;
		location @400 { return 400 '\n'; }
		
		error_page 500 502 503 504 = @500;
		location @500 { return 500 '\n'; }

		location / {
			limit_req zone=ext burst=500 nodelay;
			proxy_pass		https://gw;
			proxy_http_version	1.1;

			proxy_set_header Connection "";
			proxy_set_header iv-user "";
			proxy_set_header X-SSL-Client-CN "";
			proxy_set_header X-SSL-Client-Verified "";
			proxy_set_header X-SSL-Client-Issuer "";

			proxy_set_header Host $host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_set_header X-SSL-Client-Issuer $ssl_client_i_dn;
			proxy_set_header X-SSL-Client-CN $ssl_client_s_dn_cn;
			proxy_set_header X-SSL-Client-Verified $ssl_client_verify;
			}
	}
}
```
	
