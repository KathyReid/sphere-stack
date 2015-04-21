# sphere-stack

This setup is used to launch a standalone copy of the sphere platform either on your local machine or on a cloud host.

# prerequisites 

To start you need a host running ubuntu 14.04.02 with docker installed, this can be setup using docker-machine.

* [docker](http://docker.io)
* [docker-compose](https://docs.docker.com/compose/)
* [docker-machine](https://docs.docker.com/machine/) [optional]

# overview

I have broken the system up into two compose configuration files, being resources and services.

## resources

This are the resource services, consisting of mysql, rabbitmq, couchdb and redis.

These services keep their data in the following folders on the docker host:

* /data/couchdb
* /data/mysql
* /data/rabbitmq

## services

These are the services which ninjablocks developed.

* [douitsu](https://github.com/ninjablocks/douitsu)
* [activation](https://github.com/ninjablocks/sphere-activation-service)
* [apiservice](https://github.com/ninjablocks/sphere-api-service)
* [rpcservice](https://github.com/ninjablocks/sphere-cloud-rpc-service)
* [modelservice](https://github.com/ninjablocks/sphere-cloud-modelstore-service)
* [sphere-go-state-service](https://github.com/ninjablocks/sphere-go-state-service)
* [mqtt-proxy](https://github.com/ninjablocks/mqtt-proxy)

# running

```
docker-compose -f resources-docker-compose.yml -d
```

## databases

Then to if this is the first time you have run it you need to import the SQL database.

```
cat test_data.sql | docker exec -i spherestack_spheremysql_1 mysql -uroot
```

Then ensure the couchdb database is created and create the secondary index.

```
docker exec -i spherestack_spherecouch_1 curl -X PUT http://127.0.0.1:5984/sphere_modelstore
curl -X PUT http://IPOFDOCKERHOST.local:5984/sphere_modelstore/_design/manifest -d @manifest.json
```

## openssl 

```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout haproxy/ssl/sphere.key -out haproxy/ssl/sphere.crt
```

When you hit "ENTER", you will be asked a number of questions.

```
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:New York
Locality Name (eg, city) []:New York City
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Your Company
Organizational Unit Name (eg, section) []:Department of Kittens
Common Name (e.g. server FQDN or YOUR name) []:*.your_domain.com
Email Address []:your_email@domain.com
```
Combine these files.

```
cat haproxy/ssl/sphere.key haproxy/ssl/sphere.crt >> haproxy/ssl/wildcard.pem
```

## services

Then start the services.

```
docker-compose -f resources-docker-compose.yml
```

# Licensing

sphere-stack is licensed under the MIT License. See LICENSE for the full license text.
