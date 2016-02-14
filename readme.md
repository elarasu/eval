# eval server

eval server

## development


## init
```
psql -h postgres -U docker postgres
DROP DATABASE todos;
CREATE DATABASE todos;
psql -h postgres -U docker todos
CREATE EXTENSION postgis;
CREATE EXTENSION postgis_topology;

python manage.py collectstatic
python manage.py loaddata fixtures/sites.json
python manage.py loaddata fixtures/oauth2_provider.json

```

## docker-compose
sudo route -n add 172.17.0.0/16 192.168.99.100


## Setup
```
virtualenv -p python3 env
source env/bin/activate
pip install -r requirements.txt
./manage.py makemigrations
./manage.py migrate
./manage.py loaddata sites
./manage.py runserver
```

## Instructions
```
source env/bin/activate
```

   * go to /oauth/applications/, Client Type: confidential, Authorization Grant Type: Resource owner password-based
   * python manage.py loaddata oauth2_provider.json

### oauth token

   * curl -X POST -d 'client_id=client&client_secret=secret&grant_type=password&username=demo1&password=test123' 'http://localhost:4080/oauth/token/'
   * export AUTH_HDR='Authorization: Bearer <access-token-here>'

### save oauth fixture

   * python manage.py dumpdata oauth2_provider.application > fixtures/oauth2_provider.application

### run
To run the server
```
export MQTT_HOST='192.168.1.13'
./manage.py runserver 0.0.0.0:4080
```

## APIs

### User Management

### Todo List Management

   * curl -X GET     -H "$AUTH_HDR" 'http://localhost:4080/api/todos'
   * curl -X POST    -H "$AUTH_HDR" 'http://localhost:4080/api/todos' --data 'task=where'
   * curl -X PUT     -H "$AUTH_HDR" 'http://localhost:4080/api/todos/2' --data 'task=changed+here'
   * curl -X DELETE  -H "$AUTH_HDR" 'http://localhost:4080/api/todos/2'

## Observer Protocol

### JsonData Protocol
{jsondata:, id:#entity-id, event:"add|set|remove", data: <json-serialized-data>}

### JsonRPC Protocol
{jsonrpc:, method:"get|create|update|delete", params:}

### Client

   * use mqtt topic '/data/:model'
   * subscribe to changes on the model '/data/:model/#'
   * an event on this topic would carry jsondata events - check for jsondata key and then use it
   * To initiate changes (one can use REST endpoint or mqtt endpoint):
      * use jsonrpc protocol on same topic '/data/:model'
      * method could be one of 'get' | 'create' | 'update' | 'delete'
      * params - would carry params, in case of update/create it goes in body
         * topic: /data/todos {method:'get', params:{a:"hello"}} => GET /api/todos?a=hello
         * topic: /data/todos/1 {method:'delete'} => DELETE /api/todos/1
         * topic: /data/todos {method:'create', params:{task:'my todo'}} => POST /data/todos,  body will carry task='my todo'
      * do we use 'jsondata' as another param to identify that this is a jsondata command?
   * client should first syncup data that it requires before using the above mechanism
      * sync all (could use REST or jsonData)
         *
   * authentication

### Server
   * Notification server
      * support jsonrpc for request-response

   * REST server
      * REST api support
         * create - POST on /api/:model
         * update - PUT with /api/model/:id
         * get - GET on /api/:model - list of items
         * get item - GET on /api/:model/:id - returns item
         * delete - DELETE on /api/:model/:id
      * any changes to data model, publish an event to topic /data/:model
      * event should be of jsondata protocol spec

## Todo

    * circusweb to be fixed
    * upgrade to postgrest 9.5
    * upgrade to python3
    * set password and db stuff from environment variables
    * load test
