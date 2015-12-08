# dokku mysql (beta) [![Build Status](https://img.shields.io/travis/dokku/dokku-mysql.svg?branch=master "Build Status")](https://travis-ci.org/dokku/dokku-mysql) [![IRC Network](https://img.shields.io/badge/irc-freenode-blue.svg "IRC Freenode")](https://webchat.freenode.net/?channels=dokku)

Official mysql plugin for dokku. Currently defaults to installing [mysql 5.6.26](https://hub.docker.com/_/mysql/).

## requirements

- dokku 0.4.0+
- docker 1.8.x

## installation

```shell
# on 0.3.x
cd /var/lib/dokku/plugins
git clone https://github.com/dokku/dokku-mysql.git mysql
dokku plugins-install

# on 0.4.x
dokku plugin:install https://github.com/dokku/dokku-mysql.git mysql
```

## commands

```
ib:clone <name> <new-name>  Create container <new-name> then copy data from <name> into <new-name>
ib:connect <name>           Connect via mysql to a mysql service
ib:create <name>            Create a mysql service with environment variables
ib:destroy <name>           Delete the service and stop its container if there are no links left
ib:export <name> > <file>   Export a dump of the mysql service database
ib:expose <name> [port]     Expose a mysql service on custom port if provided (random port otherwise)
ib:import <name> < <file>   Import a dump into the mysql service database
ib:info <name>              Print the connection information
ib:link <name> <app>        Link the mysql service to the app
ib:list                     List all mysql services
ib:logs <name> [-t]         Print the most recent log(s) for this service
ib:promote <name> <app>     Promote service <name> as DATABASE_URL in <app>
ib:restart <name>           Graceful shutdown and restart of the mysql service container
ib:start <name>             Start a previously stopped mysql service
ib:stop <name>              Stop a running mysql service
ib:unexpose <name>          Unexpose a previously exposed mysql service
ib:unlink <name> <app>      Unlink the mysql service from the app
```

## usage

```shell
# create a mysql service named lolipop
dokku ib:create lolipop

# create a mysql service
dokku ib:create lolipop

# get connection information as follows
dokku ib:info lolipop

# a mysql service can be linked to a
# container this will use native docker
# links via the docker-options plugin
# here we link it to our 'playground' app
# NOTE: this will restart your app
dokku ib:link lolipop playground

# the following environment variables will be set automatically by docker (not
# on the app itself, so they won’t be listed when calling dokku config)
#
#   DOKKU_MYSQL_LOLIPOP_NAME=/lolipop/DATABASE
#   DOKKU_MYSQL_LOLIPOP_PORT=tcp://172.17.0.1:3306
#   DOKKU_MYSQL_LOLIPOP_PORT_3306_TCP=tcp://172.17.0.1:3306
#   DOKKU_MYSQL_LOLIPOP_PORT_3306_TCP_PROTO=tcp
#   DOKKU_MYSQL_LOLIPOP_PORT_3306_TCP_PORT=3306
#   DOKKU_MYSQL_LOLIPOP_PORT_3306_TCP_ADDR=172.17.0.1
#
# and the following will be set on the linked application by default
#
#   DATABASE_URL=mysql://root:SOME_PASSWORD@dokku-mysql-lolipop:3306/lolipop
#
# NOTE: the host exposed here only works internally in docker containers. If
# you want your container to be reachable from outside, you should use `expose`.

# another service can be linked to your app
dokku mysql:link other_service playground

# since DATABASE_URL is already in use, another environment variable will be
# generated automatically
#
#   DOKKU_MYSQL_BLUE_URL=mysql://root:ANOTHER_PASSWORD@dokku-mysql-other-service:3306/other_service

# you can then promote the new service to be the primary one
# NOTE: this will restart your app
dokku mysql:promote other_service playground

# this will replace DATABASE_URL with the url from other_service and generate
# another environment variable to hold the previous value if necessary.
# you could end up with the following for example:
#
#   DATABASE_URL=mysql://root:ANOTHER_PASSWORD@dokku-mysql-other_service:3306/other_service
#   DOKKU_MYSQL_BLUE_URL=mysql://root:ANOTHER_PASSWORD@dokku-mysql-other-service:3306/other_service
#   DOKKU_MYSQL_SILVER_URL=mysql://root:SOME_PASSWORD@dokku-mysql-lolipop:3306/lolipop

# you can also unlink a mysql service
# NOTE: this will restart your app and unset related environment variables
dokku ib:unlink lolipop playground

# you can tail logs for a particular service
dokku ib:logs lolipop
dokku ib:logs lolipop -t # to tail

# you can dump the database
dokku ib:export lolipop > lolipop.sql

# you can import a dump
dokku ib:import lolipop < database.sql

# you can clone an existing database to a new one
dokku ib:clone lolipop new_database

# finally, you can destroy the container
dokku ib:destroy lolipop
```
