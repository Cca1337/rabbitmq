# RabbitMQ

### run a standalone instance
```
docker network create rabbits
docker run -d --rm --net rabbits --hostname rabbit-1 --name rabbit-1 rabbitmq

# grab existing erlang cookie
docker exec -it rabbit-1 cat /var/lib/rabbitmq/.erlang.cookie

# clean up
docker rm -f rabbit-1

```
### Note

Remember we will need the Erlang Cookie to allow instances to authenticate with each other.

## Automated Clustering rabbitmq.conf

different config file for each instance

```
docker run -d --rm --net rabbits `
-v ${PWD}/config/rabbit-1/:/config/ `
-e RABBITMQ_CONFIG_FILE=/config/rabbitmq `
-e RABBITMQ_ERLANG_COOKIE=WIWVHCDTCIUAWANLMQAW `
--hostname rabbit-1 `
--name rabbit-1 `
-p 8081:15672 `
rabbitmq

docker run -d --rm --net rabbits `
-v ${PWD}/config/rabbit-2/:/config/ `
-e RABBITMQ_CONFIG_FILE=/config/rabbitmq `
-e RABBITMQ_ERLANG_COOKIE=WIWVHCDTCIUAWANLMQAW `
--hostname rabbit-2 `
--name rabbit-2 `
-p 8082:15672 `
rabbitmq

docker run -d --rm --net rabbits `
-v ${PWD}/config/rabbit-3/:/config/ `
-e RABBITMQ_CONFIG_FILE=/config/rabbitmq `
-e RABBITMQ_ERLANG_COOKIE=WIWVHCDTCIUAWANLMQAW `
--hostname rabbit-3 `
--name rabbit-3 `
-p 8083:15672 `
rabbitmq

```
## enable federation plugin
```
docker exec -it rabbit-1 rabbitmq-plugins enable rabbitmq_federation 
docker exec -it rabbit-2 rabbitmq-plugins enable rabbitmq_federation
docker exec -it rabbit-3 rabbitmq-plugins enable rabbitmq_federation

```
## Enable Statistics
```
docker exec -it rabbit-1 rabbitmq-plugins enable rabbitmq_management
docker exec -it rabbit-2 rabbitmq-plugins enable rabbitmq_management
docker exec -it rabbit-3 rabbitmq-plugins enable rabbitmq_management
```
## Management
guest:guest

```
#NODE 1 : MANAGEMENT http://localhost:8081
#NODE 2 : MANAGEMENT http://localhost:8082
#NODE 3 : MANAGEMENT http://localhost:8083

```
## Automatic Synchronization and mirroring

```
rabbitmqctl set_policy ha-fed \
    ".*" '{"federation-upstream-set":"all", "ha-sync-mode":"automatic", "ha-mode":"nodes", "ha-params":["rabbit@rabbit-1","rabbit@rabbit-2","rabbit@rabbit-3"]}' \
    --priority 1 \
    --apply-to queues
    
```