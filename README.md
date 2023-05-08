# Nginx docker image with Headers more as dynamic module based on official stable Nginx docker image 
For more info about the "Headers more" module check: https://github.com/openresty/headers-more-nginx-module

## Examples

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




	
