
> **Purpose:** Create logs through Fluentd that will be stored in ElastiSearch and visible in Kibana/Grafana

# Usage

## How to run

```
docker-compose up -d
```

Check that everything is up *(re-run the command above if any service failed)*:
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

### Initiate ES pipeline used to process payloads from Fluentd

```
curl -X PUT \
  http://localhost:9200/_ingest/pipeline/users-pipeline \
  -H 'Content-Type: application/json' \
  --data-binary "@files/pipelines/users.pipeline.json"
```

### Initiate ES index template used to index payloads from Fluentd

```
curl -X PUT \
  "http://localhost:9200/_template/users" \
  -H 'Content-Type: application/json' \
  --data-binary "@files/index_templates/index_users.json"
```

### Initiate Kibana dashboards used for Users data

- Go to: http://localhost:5601/app/kibana#/management/kibana/objects
- Import: `files/kibana/export.json`

## Generate logs

### Through TCP

```
curl -s https://randomuser.me/api/ | jq -c .results[0] | \
  docker run --rm -i fluent/fluentd fluent-cat \
    --json users.forward \
    --host 172.17.0.1 \
    --port 24224
```
In a loop:

```
while true; do \
  curl -s https://randomuser.me/api/ | jq -c .results[0] | \
    docker run --rm -i fluent/fluentd fluent-cat \
      --json users.forward \
      --host 172.17.0.1 \
      --port 24224
    sleep 2
done
```

### Through HTTP

```
echo "json=$(curl -s https://randomuser.me/api/ | jq -c .results[0])" |
curl -X POST -d \
  @- \
  http://127.0.0.1:9880/users.http
```

In a loop:

```
while true; do \
  echo "json=$(curl -s https://randomuser.me/api/ | jq -c .results[0])" |
  curl -X POST -d \
    @- \
    http://127.0.0.1:9880/users.http
    sleep 2
done
```

## Access the services - available urls

 - Elasticsearch: http://localhost:9200
 - Kibana: http://localhost:5601
 - Grafana: http://localhost:3000
 - Fluentd HTTP: http://localhost:9880
 - Fluentd TCP: http://localhost:24224
 - Cerebro: http://localhost:9000 (to connect to the ElasticSearch above use the following url: http://elasticsearch:9200)
