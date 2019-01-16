
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

### Initiate ES pipeline used in Fluentd

```
curl -X PUT \
  http://localhost:9200/_ingest/pipeline/debug-pipeline \
  -H 'Content-Type: application/json' \
  -d '{
    "description": "debug pipeline",
    "processors": [
        {
            "rename": {
                "field": "location.coordinates.latitude",
                "target_field": "location.coordinates.lat",
                "tag": "geo_renamed_lat",
                "ignore_missing": true
            }
        },
        {
            "rename": {
                "field": "location.coordinates.longitude",
                "target_field": "location.coordinates.lon",
                "tag": "geo_renamed_lon",
                "ignore_missing": true
            }
        },
        {
            "remove": {
                "field": "login"
            }
        }
    ]
}'
```

### Initiate Kibana dashboards used

- Go to: http://localhost:5601/app/kibana#/management/kibana/objects
- Import: `files/kibana/export.json`

## How to access the services

### Available urls

 - Elasticsearch: http://localhost:9200
 - Kibana: http://localhost:5601
 - Grafana: http://localhost:3000
 - Fluentd HTTP: http://localhost:9880
 - Fluentd TCP: http://localhost:24224
 - Cerebro: http://localhost:9000 (to connect to the ElasticSearch above use the following url: http://elasticsearch:9200)

## How to generate logs

### Through TCP

```
curl https://randomuser.me/api/ | jq -c .results[0] | \
  docker run --rm -i fluent/fluentd fluent-cat \
    --json debug.forward \
    --host 172.17.0.1 \
    --port 24224
```
In a loop:

```
while true; do \
  curl https://randomuser.me/api/ | jq -c .results[0] | \
    docker run --rm -i fluent/fluentd fluent-cat \
      --json debug.forward \
      --host 172.17.0.1 \
      --port 24224
    sleep 2
done
```

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
