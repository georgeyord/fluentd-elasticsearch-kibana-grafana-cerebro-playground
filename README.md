
> **Purpose:** Create logs through Fluentd that will be stored in ElastiSearch and visible in Kibana/Grafana

# Usage

## How to run

```
docker-compose up -d
```

Check that everything is up:
```
docker-compose ps
```

Check the logs:
```
docker-compose logs -f
```

Check the logs of a specific container:
```
docker-compose logs -f cerebro
```

## How to access the services

### Available urls

 - Elasticsearch: http://localhost:9200
 - Kibana: http://localhost:5601
 - Grafana: http://localhost:3000
 - Fluentd HTTP: http://localhost:9880
 - Fluentd TCP: http://localhost:24224
 - Cerebro: http://localhost:9000 (to connect to the ElasticSearch above use the following url: http://elasticsearch:9200)

## How to generate logs

### Through HTTP

```
curl -X POST -d \
  'json={"message":{"foo":"bar","num":2}}' \
  http://127.0.0.1:9880/centaur.logs
```

In a loop:

```
while true; do \
  curl -X POST -d \
    'json={"message":{"foo":"bar","num":3}}' \
    http://127.0.0.1:9880/centaur.logs
    sleep 2
done
```

### Through TCP

```
echo '{"message":{"foo":"bar","num":1}}' | \
  docker run --rm -i fluent/fluentd fluent-cat \
    --json debug.forward \
    --host 172.17.0.1 \
    --port 24224
```
In a loop:

```
while true; do \
  echo '{"message":{"foo":"bar","num":1}}' | \
    docker run --rm -i fluent/fluentd fluent-cat \
      --json debug.forward \
      --host 172.17.0.1 \
      --port 24224
    sleep 2
done
```
