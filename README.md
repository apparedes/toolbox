# toolbox

## Redis

```bash
podman network create redis-network
podman run -d --name redis --network redis-network -p 127.0.0.1:6379:6379 -v redis-data:/data docker.io/library/redis:7 redis-server --appendonly yes --requirepass "T3mp0r4l*"
```

## Redis Client
```bash
podman run -d --name redisinsight --network redis-network -p 5540:5540 -v redisinsight-data:/data redis/redisinsight:latest
```
