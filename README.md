# Nginx docker image with Headers more as dynamic module based on official Nginx docker image  

## Examples

### More Set Headers

```
http {
    ...
    more_set_headers 'Server: 1337-server';
    ...
}
```

