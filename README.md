# lua-consul-balancer

Consul balancer for Openresty/Nginx.

* consul-balancer.lua - a lua file ready to copy/paste
* nginx.conf - the minimal Nginx config as an example

## Features

* Discover services from one or multiple Consul endpoints and stores their addresses for load balancing
* Periodically refresh addresses
* By default it picks up only non-critical services based on the Consul native healthcheck
* Optionally, you can enable an additional custom healthcheck against the service URL or allow services in warning state
* Traffic throttling for new instances of the services as configured by service_warmup_period
* Last address standing - in case no addresses are available from Consul you will still get the latest one
* Bonus: extra logic to sleep for 5s every 10 retries on error if `set $delayed_retry 1;` is defined on nginx location

## Description

You can configure service discovery refresh interval. There are also a couple of hardcoded settings in the lua file.

Traffic throttling means that a new service will receive less traffic proportionally within service_warmup_period.
For example, if service_warmup_period is 60s then a new service instance will receive 10% of the traffic on 10th second, 50% on 30th second and so on until full on 60th second. It will be visible from the status page
(see percentage in the end of this page as an example).

We are using this lua code at Quiq for 4 years or so and it is very fast with even 50 services in total
and multiple addresses per each one. It doesn't produce much load on Openresty/Nginx if any at all.

## Openresty required modules

* lua-resty-http
* lunajson

They can be installed via luarocks:

    /usr/local/openresty/luajit/bin/luarocks install lua-resty-http
    /usr/local/openresty/luajit/bin/luarocks install lunajson

## Test with docker

Assuming you have added certificates to the folder, updated nginx.conf according to your needs
you can run a test as follow:

    $ ls -la
    total 80
    -rw-r--r--  1 weber  staff   1830 Aug  3 16:38 bundle-crt.pem
    -rw-r--r--  1 weber  staff   3940 Aug  3 16:42 ca-certificates.crt
    -rw-r--r--  1 weber  staff  12459 Aug  4 15:20 consul-balancer.lua
    -rw-r--r--  1 weber  staff   3243 Aug  3 16:38 key.pem
    -rw-r--r--  1 weber  staff   1956 Aug  4 15:21 nginx.conf
    $
    $ docker run --rm -d -p 443:443 -v $PWD:/usr/local/openresty/nginx/conf:ro quiq/openresty:1.21.4.1-alpine

    $ curl -k https://localhost/status
    * Consul SD
      Last refresh time:        Thu Aug  4 12:21:51 2022
      foobar-service:           10.0.7.71:8081
      another-service:          10.0.7.74:8082 10.0.7.75:8082[50%]

`quiq/openresty` is our docker image built with everything required and available from Docker Hub.
