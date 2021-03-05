https://stackoverflow.com/questions/39260477/nginx-response-time
https://www.cnblogs.com/kevingrace/p/5893499.html

We have a requirement to log the request time.

Add the following block in nginx.conf
log_format timed_combined '$remote_addr - $remote_user [$time_local] '
    '"$request" $status $body_bytes_sent '
    '"$http_referer" "$http_user_agent" '
    '$request_time $upstream_response_time $pipe';
    
    
