# Docker ELK stack

Run the ELK (Elasticseach, Logstash, Kibana) stack with Docker Compose.

Based on the official images:

* [elasticsearch](https://registry.hub.docker.com/_/elasticsearch/), v2.3.4
* [logstash](https://registry.hub.docker.com/_/logstash/), v2.3.4
* [kibana](https://registry.hub.docker.com/_/kibana/), v4.5.3

## Features

The Logstash supports the following input:

* 5044: Filebeat input
* 12201/udp: GELF input, which is used to accept logs via Docker containers.

The Elasticsearch index can be specified with `[@metadata][index]`, `[@metadata][beat]` or `logstash_index` field.
The following log filters are supported, which should be specified via the `[@metadata][type]` or `logstash_type` field.

* `apache_access`: Apache access log
* `nginx_access`: nginx access log

## Setup

Start the ELK stack using *docker-compose*:

```bash
$ docker-compose up -d
```

To use Filebeats input, we need to set the template in Elasticsearch:

```bash
curl -XPUT 'http://192.168.99.100:9200/_template/filebeat?pretty' -d@filebeat/index-template.json
```

After that, we can access Kibana UI by hitting [http://localhost:5601](http://localhost:5601).

By default, the stack exposes the following ports:
* 9200: Elasticsearch HTTP
* 9300: Elasticsearch TCP transport
* 5601: Kibana


### Setup Docker containers

A `docker-compose.yml` should contain configurations like:

```yaml
version: '2'
services:
  web:
    image: nginx:1.11.1
    labels:
      logstash_index: theinitium.com
      logstash_type: nginx_access
    logging:
      driver: gelf
      options:
        gelf-address: udp://$LOGSTASH_IP:12201
        gelf-tag: mediawiki
        labels: logstash_index,logstash_type
```

### Setup Filebeat

The `/etc/filebeat/filebeat.yml` should contain configurations like:

```yaml
filebeat:
  prospectors:
    -
      paths:
        - /var/log/nginx/*.log
      document_type: nginx_access

output:
  logstash:
    hosts: ["$LOGSTASH_IP:5044"]
    index: theinitium.com
```

## Testing

For debugging purpose, the `./logstash/31-output-stdout.conf` will be mounted at the logstash config directory and enable
`stdout` output. The file is not built into the Docker image.

### By input type

To simulate a GELF log (with $ELK_IP pointing to the ELK server ip, such as `ELK_IP=$(docker-machine ip default)`):

```bash
echo '{"version": "1.1","host":"example.org","message":"Backtrace heremore stuff"}' | tee /dev/tty | gzip --stdout | nc -u -w 1 $ELK_IP 12201
```

To simulate a Filebeat input:

```bash
```

### By log type

Specify the log type by `logstash_type` (`apache_access` or `nginx_access`):

```bash
echo '{"version": "1.1","host":"example.org","logstash_type": "nginx_access", "message":"192.241.205.129 - - [19/Jul/2016:23:59:56 +0800] \"GET /article/20160719-dailynews-germany-train-axe/ HTTP/1.1\" 200 13840 \"-\" \"Mozilla/5.0 (Linux; Android 5.1; 8681-A01 Build/LMY47D) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/39.0.0.0 Mobile Safari/537.36\""}' | tee /dev/tty | gzip --stdout | nc -u -w 1 $ELK_IP 12201
```

