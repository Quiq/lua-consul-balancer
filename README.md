# lua-consul-balancer

Consul balancer for Openresty/Nginx.

* consul-balancer.lua - a lua file ready to copy/paste
* nginx.conf - the minimal Nginx config as an example

## Features

* Discovers services from one or multiple Consul endpoints and stores their addresses for load balancing
* Periodically refresh addresses
* Traffic throttling for new instances of the services as configured by service_warmup_period
* Last address standing - in case no addresses are available from Consul you will still get the latest one
* Bonus: extra logic to sleep for 5s every 10 retries if "set $delayed_retry 1;" is defined

You can configure service discovery refresh interval. There are also a couple of hardcoded settings in the lua file.

Traffic throttling means that a new service will receive less traffic proportionally within service_warmup_period.
For example, if service_warmup_period is 60s then a new service instance will receive 10% of the traffic at 10th second, 50% on 30th second and so on until full on 60th second. It will be visible from the status page
(see below as an example).

We are using this lua code at Quiq for 4 years or so and it is very fast with even 50 services in total
and multiple addresses per each one. It doesn't produce much load on Openresty/Nginx if any at all.

## Status page output

    * Consul SD
      Last refresh time:        Thu Aug  4 12:21:51 2022
      foobar-service:           10.0.7.71:8081
      another-service:          10.0.7.74:8082 10.0.7.75:8082[50%]

## Openresty required modules

* lua-resty-http
* lunajson

They can be installed as follow:

    /usr/local/openresty/luajit/bin/luarocks install lua-resty-http
    /usr/local/openresty/luajit/bin/luarocks install lunajson
