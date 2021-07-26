# How to fix Elasticbeanstalk large body

Error: client intended to send too large body

https://www.tecmint.com/limit-file-upload-size-in-nginx/

It's caused by Nginx upload file limit.

The default file size is 1 MB.

For Nginx itself, setting a new maximum limit is fine.

It sets the maximum allowed size of the client request body, specified in the “Content-Length” request header field. Here’s an example of increasing the limit to 100MB in /etc/nginx/nginx.conf file.

Set in http block which affects all server blocks (virtual hosts).

```
http {
    ...
    client_max_body_size 100M;
}  
```

Set in server block, which affects a particular site/app.

```
server {
    ...
    client_max_body_size 100M;
}
```

But for Elasticbeanstalk it's another story.

Here is how to extend the EB nginx configuration.

## For Amazon linux 1 

https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/nodejs-platform-proxy.html

```
.ebextensions/
  nginx/
    conf.d/
      proxy.conf
  01_files.config
```


## For Amazon linux 2

https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/platforms-linux-extend.html

```
.platform/
  nginx/
    conf.d/
      proxy.conf
  01_files.config
```
  
## File Content

proxy.conf
```
client_max_body_size 50M;
```
  
01_files.config
```
files:
  "/etc/nginx/conf.d/proxy.conf" :
      mode: "000755"
      owner: root
      group: root
      content: |
         client_max_body_size 50M;
```
