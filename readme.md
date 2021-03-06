# **Docker Nginx Auto Proxy**
This docker image automatic proxies requests to your docker containers

## Usage

## Configuration

First, pull the image from docker hub

    docker pull stephenafamo/docker-nginx-auto-proxy:2.1.2

Run a container

    docker run --name nginx -v /path/to/my/config:/docker/config/config -p 80:80 -p 443:443 stephenafamo/docker-nginx-auto-proxy:2.1.0

The container reads a configuration file `/docker/config/config`
To easily manage all proxies, you should mount your own configuration file.
`-v /path/to/my/config.txt:/docker/config/config`

### HTTP Proxy & Load balancing

The syntax is as follows(showing all possible fields).

        "myblog"
        "UPSTREAM"main.stephenafamo.com | 1st.stephenafamo.com weight=3 | 2nd.stephenafamo.com max_fails=3 fail_timeout=30s"
        "UPSTREAM_OPTIONS"ip_hash | keepalive 32"
        "DOMAIN"stephenafamo.com"
        "DIRECTORY"blog"
        "SSL"1"
        "SSL_SOURCE"letsencrypt"
        "HTTPS_ONLY"1"
        "TYPE"HTTP"
        "myblog"


1. The only required fields are `UPSTREAM` and `DOMAIN`
1. A block of configuration should be started and ended by the configuration name. This name should be unique. In the example above, the configuration name is `myblog`
2. Neither the domain or upstream address should include the scheme `http://`
3. `UPSTREAM` must be reachable or the config will not be generated.
4. For load balancing, you can add multiple `UPSTREAM` addresses. Separate them with pipes.
5. You can add any extra parameters at the end of a single upstream server. [Read this](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#server).
6. The `UPSTREAM_OPTIONS` are not required. Use only if you need to add extra directives to the upstream block for fine tuning. Separate directives with pipes. [Read this](http://nginx.org/en/docs/http/ngx_http_upstream_module.html).
7. `DOMAIN` can be multiple, but should be seperated by spaces
8. `DIRECTORY` is the optional path to to be proxies. For example, if you'd like to proxy only `example.com/blog`, the `blog` will be the directory
9. `SSL` can be enable by setting the parameter to `1`
10. `SSL_SOURCE` for now, only letsencrypt is supported. Certificates will be generated automatically. Soon, mannual configuration will be supported. To be able to re-use the generated certificates, you should mount your `/etc/letsencrypt` folder into the container `-v /etc/letsencrypt:/etc/letsencrypt`. THis only works if `SSL` is `1`.
11. `HTTPS_ONLY` If this is set to `1`, then all `http` requests will be redirected to `https`.
12. `TYPE`: For a `http` proxy, you can omit this field or set it to `HTTP`.

### TCP & UDP Load balancing

The syntax is as follows(showing all possible fields).

        "myblog"
        "UPSTREAM"main.stephenafamo.com | 1st.stephenafamo.com weight=3 | 2nd.stephenafamo.com max_fails=3 fail_timeout=30s"
        "UPSTREAM_OPTIONS"ip_hash | keepalive 32"
        "PORT"12345"
        "SERVER_OPTIONS"blog"
        "SSL"1"
        "SSL_SOURCE"letsencrypt"
        "HTTPS_ONLY"1"
        "myblog"


1. The only required fields are `UPSTREAM`, `PORT` and `TYPE`.
1. A block of configuration should be started and ended by the configuration name. This name should be unique. In the example above, the configuration name is `myblog`
2. Neither the domain or upstream address should include the scheme `http://`
3. `UPSTREAM` must be reachable or the config will not be generated.
4. For load balancing, you can add multiple `UPSTREAM` addresses. Separate them with pipes.
5. You can add any extra parameters at the end of a single upstream server. [Read this](http://nginx.org/en/docs/http/ngx_http_upstream_module.html#server).
6. The `UPSTREAM_OPTIONS` are not required. Use only if you need to add extra directives to the **upstream** block for fine tuning. Separate directives with pipes. [Read this](http://nginx.org/en/docs/http/ngx_http_upstream_module.html). Anything with context `upstream` can be used.
7. The `SERVER_OPTIONS` are not required. Use only if you need to add extra directives to the **server** block for fine tuning. Separate directives with pipes. [Read this](http://nginx.org/en/docs/stream/ngx_stream_proxy_module.html). Anything with context `stream, server` can be used.
8. `TYPE`: Can be set to either `TCP` or `UDP` depending on the type of traffic to load balance.
9. `PORT` is the port which nginx should listen on for the `TCP` or `UDP` traffic.

## Additional commands 

The following commands are available through the contianer.

1. **_active_domains_**: Will list out the domains that have been configured
3. **_load_config_**: Will re-generate configuration files and reload nginx

## Let's Encrypt

If set up correctly, the container will attempt to get a new certificate if there was none, or renew the certificate.
The normal `letsencrypt renew` command may fail. Instead, to renew certificates, run the `load_config` command and certificate renewal will be attempted during the process. 
You should set up a cron to do this automatically e.g `30 2 * * 1 docker exec nginx load_config >> /var/log/nginx-reload.log`

## Roadmap

* ~~Load balancing with multiple containers~~ **DONE**
* ~~Automatic SSL support with let's encrypt~~ **DONE**
* Allow custom ssl certificate configuration.

Please inform me of any issues, Pull requests are appreciated.
