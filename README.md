 toolbox

## Redis

```bash
podman network create redis-network
podman run -d --name redis --network redis-network -p 127.0.0.1:6379:6379 -v redis-data:/data docker.io/library/redis:7 redis-server --appendonly yes --requirepass "T3mp0r4l*"
```

## Redis Client

```bash
podman run -d --name redisinsight --network redis-network -p 5540:5540 -v redisinsight-data:/data docker.io/redis/redisinsight:latest
```

## MongoBD

```bash
podman volume create mongodb-keyfile

podman run --rm \
  -v mongodb-keyfile:/key \
  docker.io/library/mongo:7.0.43 \
  bash -c '
    head -c 756 /dev/urandom | base64 > /key/keyfile &&
    chown mongodb:mongodb /key/keyfile &&
    chmod 400 /key/keyfile
  '

podman run -d \
  --name mongo1 \
  --network backend-network \
  -p 127.0.0.1:27017:27017 \
  -v mongo1-data:/data/db \
  -v mongodb-keyfile:/etc/mongo-keyfile:ro \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD='T3mp0r4l*' \
  docker.io/library/mongo:7.0.43 \
  --replSet rs0 \
  --bind_ip_all \
  --keyFile /etc/mongo-keyfile/keyfile

podman run -d \
  --name mongo2 \
  --network backend-network \
  -v mongo2-data:/data/db \
  -v mongodb-keyfile:/etc/mongo-keyfile:ro \
  docker.io/library/mongo:7.0.43 \
  --replSet rs0 \
  --bind_ip_all \
  --keyFile /etc/mongo-keyfile/keyfile

podman run -d \
  --name mongo3 \
  --network backend-network \
  -v mongo3-data:/data/db \
  -v mongodb-keyfile:/etc/mongo-keyfile:ro \
  docker.io/library/mongo:7.0.4 \
  --replSet rs0 \
  --bind_ip_all \
  --keyFile /etc/mongo-keyfile/keyfile

podman exec -it mongo1 mongosh \
  -u admin \
  -p 'T3mp0r4l*' \
  --authenticationDatabase admin

rs.initiate({
  _id: "rs0",
  members: [
    {
      _id: 0,
      host: "mongo1:27017"
    },
    {
      _id: 1,
      host: "mongo2:27017"
    },
    {
      _id: 2,
      host: "mongo3:27017"
    }
  ]
})

rs.status()

```
