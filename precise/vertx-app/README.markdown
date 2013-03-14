
# juju charm to deploy a user-defined Vert.x app

This is an example 
[juju](http://juju.ubuntu.com)
charm to deploy a user-defined Vert.x app
directly from artifact repository.

This charm will be maintained with a general set of hooks
for various services that can be used with node apps
(like redis).


# Using this charm

First, edit `config.yaml` to add info about your app.

Then deploy some basic services

    juju deploy vertx-app myapp
    juju deploy redis-master
    juju deploy haproxy

relate them

    juju add-relation redis-master:db myapp
    juju add-relation myapp haproxy

scale up your app (to 10 nodes for example)

    juju add-unit -n 10 myapp

open it up to the outside world

    juju expose haproxy

Find the haproxy instance's public URL from 

    juju status

(or attach it to an elastic IP via the aws console)
and open it up in a browser.


## Application configuration

The formula looks for `${app_name}.conf` in your app which
starts off looking something like this

    {
      "name" : "@@APP_NAME@@",
      "listen_port" : @@APP_PORT@@,
      "redis_host" : "@@REDIS_HOST@@",
      "mongo_port" : @@REDIS_PORT@@
    }


and gets modified with contextually correct configuration information during
either deployment (via the `install` hook) or relation to another service 
(`relation-changed` hook).

## Network access

This charm does not open any public ports itself.
The intention is to relate it to a proxy service like
`haproxy`, which will in turn open port 80 to the outside world.
This allows for instant horizontal scalability.

If your Vert.x app is itself a proxy and you want it directly exposed,
this can easily be done by adding 

    open-port $app_port

to the bottom of the `install` hook, and then once your stack
is started, you expose

    juju expose myapp

it to the outside world.

By default, juju services within the same environment
can talk to each other on any port over
internal network interfaces.
