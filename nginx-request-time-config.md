# Log ip and response time in ElasticBeanstalk with application load balancer

#ElasticBeanstalk #Application Load Balancer #Nginx #Cloudwatch

Introduction:

- Log ip and response time in Nginx logs
- Docker running on 64bit Amazon Linux/2.16.7
- ElasticBeanstalk with application load balancer
- Store the logs in cloudwatch

We have a requirement to log the ip and request time.

Go to the solution part if you don't care about the steps.

This article is motivated by [this article](https://alfianeffendy.medium.com/measuring-http-response-time-of-ecs-docker-container-with-cloudwatch-insight-b3dfb45b2cbf).

But this link doesn't say much on how to change the log format. And morgan way to create a log will get you a log file application.****.log. And EB won't push that log into cloudwatch. So I focus on how to change the access.log (Nginx main log file).

# Configure Nginx

## Log the client IP

https://aws.amazon.com/premiumsupport/knowledge-center/elb-capture-client-ip-addresses/
According to the AWS doc, For Application Load Balancers and Classic Load Balancers with HTTP/HTTPS listeners, the X-Forwarded-For HTTP header captures client IP addresses. You can then configure your web server access logs to record these IP addresses.

## Log the response time

https://stackoverflow.com/questions/39260477/nginx-response-time

https://www.cnblogs.com/kevingrace/p/5893499.html

This is how we add ip and requestTime and responseTime in Nginx

request_time vs upstream_response_time

request_time: request processing time in seconds with a milliseconds resolution; time elapsed between the first bytes were read from the client and the log write after the last bytes were sent to the client.

upstream_response_time: keeps time spent on receiving the response from the upstream server; the time is kept in seconds with millisecond resolution. Times of several responses are separated by commas and colons like addresses in the [$upstream_addr](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#var_upstream_addr) variable.

client ->1 nginx ->2 upsteam server ->3 nginx ->4 client

request_time: 1 2 3 4

upstream_response_time: 2 3

```
log_format timed_combined '"$http_x_forwarded_for"
	 $remote_addr - $remote_user [$time_local] '
	            '"$request" $status $body_bytes_sent '
	            '"$http_referer" "$http_user_agent" '
	            '$request_time $upstream_response_time $pipe';
    
server {
	listen 80;
	access_log /var/log/nginx/access.log timed_combined;
}

```

# Configure Nginx in elasticbeanstalk

We use docker platform.

https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ebextensions.html

This is how you do environment customization with configuration files.

Add .ebextensions/00_my_config.config in the build folder.

But adding these line will only get you a empty access-new.log.

```nginx
$remote_addr - $remote_user [$time_local] '
	            '"$request" $status $body_bytes_sent '
	            '"$http_referer" "$http_user_agent" '
	            '$request_time $upstream_response_time $pipe';
access_log /var/log/nginx/access-new.log timed_combined;
```

So I searched online and get this

https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/nodejs-platform-proxy.html

You need to delete the existing config and replace with you new one.

At first I didn't understand this page until I go to next step. Why delete a file? I just want to add a new log config.

Then I created a new EB instance and connected to the instance with ssh pem key. 

Here is what I found:

```
// Config path
/etc
	/nginx
		nginx.conf
		/conf.d
			proxy.conf
			elasticbeanstalk-nginx-docker-upstream.conf
    /sites-available
    	elasticbeanstalk-nginx-docker-proxy.conf
    /sites-enabled
    	elasticbeanstalk-nginx-docker-proxy.conf -> sites-available/elasticbeanstalk-nginx-docker-proxy.conf


// Log path
/var/log/nginx/access.log
```

Nginx config entry
/etc/nginx/nginx.conf

```nginx
# Elastic Beanstalk Nginx Configuration File

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log;

pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    access_log    /var/log/nginx/access.log;

    log_format  healthd '$msec"$uri"$status"$request_time"$upstream_response_time"$http_x_forwarded_for';

    include       /etc/nginx/conf.d/*.conf;
    include       /etc/nginx/sites-enabled/*;
}


```

/etc/nginx/conf.d/elasticbeanstalk-nginx-docker-upstream.conf 

```nginx
upstream docker {
    server 172.17.0.2:3000;
    keepalive 256;
}
```

/etc/nginx/conf.d/proxy.conf

```nginx
log_format timed_combined '"$http_x_forwarded_for"
 $remote_addr - $remote_user [$time_local] '
            '"$request" $status $body_bytes_sent '
            '"$http_referer" "$http_user_agent" '
            '$request_time $upstream_response_time $pipe';

access_log /var/log/nginx/access-mew.log timed_combined;
```

/etc/nginx/sites-enabled/elasticbeanstalk-nginx-docker-proxy.conf

->/etc/nginx/sites-available/elasticbeanstalk-nginx-docker-proxy.conf

```nginx
map $http_upgrade $connection_upgrade {
        default        "upgrade";
        ""            "";
    }
    
    server {
        listen 80;

    	gzip on;
	    gzip_comp_level 4;
	    gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

        if ($time_iso8601 ~ "^(\d{4})-(\d{2})-(\d{2})T(\d{2})") {
            set $year $1;
            set $month $2;
            set $day $3;
            set $hour $4;
        }
        access_log /var/log/nginx/healthd/application.log.$year-$month-$day-$hour healthd;

        access_log    /var/log/nginx/access.log;
    
        location / {
            proxy_pass            http://docker;
            proxy_http_version    1.1;
    
            proxy_set_header    Connection            $connection_upgrade;
            proxy_set_header    Upgrade                $http_upgrade;
            proxy_set_header    Host                $host;
            proxy_set_header    X-Real-IP            $remote_addr;
            proxy_set_header    X-Forwarded-For        $proxy_add_x_forwarded_for;
        }
    }
```

So  it's clear why the new log is empty. 

access_log /var/log/nginx/access-new.log timed_combined;

This line should be in the server block to get logs.

Then I followed the instruction as the doc.

- Combine the elasticbeanstalk-nginx-docker-proxy.conf and proxy.conf
- Delete existing elasticbeanstalk-nginx-docker-proxy.conf

# Here is the Solution

**You should find the default config for your environment. This is for docker only.**

Add .ebextensions/nginx/00-my-proxy.config to your build folder

```
files:
  /etc/nginx/conf.d/proxy.conf:
    mode: "000644"
    owner: root
    group: root
    content: |
      map $http_upgrade $connection_upgrade {
        default        "upgrade";
        ""            "";
      }

      log_format timed_combined '"$http_x_forwarded_for" $remote_addr - $remote_user [$time_local] '
                       '"$request" $status $body_bytes_sent '
                       '"$http_referer" "$http_user_agent" '
                       '$request_time $upstream_response_time $pipe';

      server {
          listen 80;

          gzip on;
          gzip_comp_level 4;
          gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

            if ($time_iso8601 ~ "^(\d{4})-(\d{2})-(\d{2})T(\d{2})") {
                set $year $1;
                set $month $2;
                set $day $3;
                set $hour $4;
            }
            access_log /var/log/nginx/healthd/application.log.$year-$month-$day-$hour healthd;

            access_log /var/log/nginx/access.log timed_combined;

            location / {
                proxy_pass            http://docker;
                proxy_http_version    1.1;

                proxy_set_header    Connection          $connection_upgrade;
                proxy_set_header    Upgrade             $http_upgrade;
                proxy_set_header    Host                $host;
                proxy_set_header    X-Real-IP           $remote_addr;
                proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
            }
      }

  /opt/elasticbeanstalk/hooks/configdeploy/post/99_kill_default_nginx.sh:
    mode: "000755"
    owner: root
    group: root
    content: |
      #!/bin/bash -xe
      rm -f /etc/nginx/sites-enabled/elasticbeanstalk-nginx-docker-proxy.conf
      service nginx stop
      service nginx start

container_commands:
  removeconfig:
    command: "rm -f /tmp/deployment/config/#etc#nginx#sites-enabled#elasticbeanstalk-nginx-docker-proxy.conf /etc/nginx/sites-enabled/elasticbeanstalk-nginx-docker-proxy.conf"

```

# Cloudwatch

Now I get the new log file.

But I need to store the log somewhere.

Here is the instructions.

Turn on the **S3 log stroage** and **Instance log streaming to CloudWatch Logs** in EB configuration.

https://docs.amazonaws.cn/en_us/elasticbeanstalk/latest/dg/environments-cfg-logging.html

Then you get several log groups in cloudwatch.

Notice:

access.log is the only one I need.  I deleted the others. Then the EB can't restart. Because other log groups are not found in cloudwatch. The solution is that disable the Instance log streaming to CloudWatch Logs and enable it again.

These are the log groups automatically created by EB.

```
/aws/elasticbeanstalk/logs-test/var/log/docker
/aws/elasticbeanstalk/logs-test/var/log/docker-events.log
/aws/elasticbeanstalk/logs-test/var/log/eb-activity.log
/aws/elasticbeanstalk/logs-test/var/log/eb-docker/containers/eb-current-app/stdouterr.log
/aws/elasticbeanstalk/logs-test/var/log/nginx/access.log
/aws/elasticbeanstalk/logs-test/var/log/nginx/error.log
```

In cloudwatch logs/insights

Select the /aws/elasticbeanstalk/logs-test/var/log/nginx/access.log

```
parse @message '"*" * - * [*] "*" * * "*" "*" * * *' as ip, remoteIp, user, time, request, statusCode, bytes, referer, userAgent, requestTime, responseTime, pipe
| filter statusCode = 200
| filter ip != '-'
| stats avg(responseTime) by bin(2h)
```

Run query

The list of logs is here.

You may add the query result to dashboard or add widget in dashboard.

I also tried restart and rebuild the EB. The log format remains the same.

Rebuilding will create a new log stream. The old logs are not lost.



At last, I don't know what the effect of enabling S3 log storage/ Rotate logs is. Learn more link will lead you to [this page](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-cfg-softwaresettings.html?icmpid=docs_elasticbeanstalk_console). But I didn't figure out how to upload logs to S3 or find the logs somewhere in the bucket.

I saw one solution saying you can override the /etc/nginx/nginx.conf

I tried the .ebextensions/nginx/nginx.conf. EB seems to ignore it.

I tried to override nginx.conf. It works as well.

.ebextensions/nginx/00-my-proxy.config
```
files:
  /etc/nginx/nginx.conf:
    mode: "000644"
    owner: root
    group: root
    content: |
        # Elastic Beanstalk Nginx Configuration File

        user  nginx;
        worker_processes  auto;

        error_log  /var/log/nginx/error.log;

        pid /var/run/nginx.pid;

        events {
            worker_connections  1024;
        }

        http {
          include /etc/nginx/mime.types;
          default_type application/octet-stream;

          access_log /var/log/nginx/access.log;

          log_format healthd '$msec"$uri"$status"$request_time"$upstream_response_time"$http_x_forwarded_for';

          upstream docker {
              server 172.17.0.2:3000;
              keepalive 256;
          }
          log_format timed_combined '"$http_x_forwarded_for"'
          						'$remote_addr - $remote_user [$time_local] '
                      '"$request" $status $body_bytes_sent '
                      '"$http_referer" "$http_user_agent" '
                      '$request_time $upstream_response_time $pipe';


          map $http_upgrade $connection_upgrade {
                  default        "upgrade";
                  ""            "";
          }

          server {
              listen 80;

                gzip on;
              gzip_comp_level 4;
              gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

                if ($time_iso8601 ~ "^(\d{4})-(\d{2})-(\d{2})T(\d{2})") {
                    set $year $1;
                    set $month $2;
                    set $day $3;
                    set $hour $4;
                }
                access_log /var/log/nginx/healthd/application.log.$year-$month-$day-$hour healthd;

                access_log /var/log/nginx/access.log timed_combined;

                location / {
                    proxy_pass            http://docker;
                    proxy_http_version    1.1;

                    proxy_set_header    Connection            $connection_upgrade;
                    proxy_set_header    Upgrade                $http_upgrade;
                    proxy_set_header    Host                $host;
                    proxy_set_header    X-Real-IP            $remote_addr;
                    proxy_set_header    X-Forwarded-For        $proxy_add_x_forwarded_for;
                }
          }
        }

```



