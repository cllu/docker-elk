# Docker ELK stack

Run the ELK (Elasticseach, Logstash, Kibana) stack with Docker Compose.

Based on the official images:

* [elasticsearch](https://registry.hub.docker.com/_/elasticsearch/), v2.3.4
* [logstash](https://registry.hub.docker.com/_/logstash/), v2.3.4
* [kibana](https://registry.hub.docker.com/_/kibana/), v4.5.3

## Features

The Logstash supports the following input:

* 5044: Filebeat input
* 12201/udp: GELF input

GELF is used to accept logs via Docker containers. The following log filters are supported:

* Apache: if the Docker container is running `apache-foreground` command
* nginx: if the Filebeats log contains `nginx` in the file path


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

## Testing

### By input type

To simulate a GELF log (with $ELK_IP pointing to the ELK server ip, such as `ELK_IP=$(docker-machine ip default)`):

```bash
echo '{"version": "1.1","host":"example.org","message":"Backtrace heremore stuff"}' | tee /dev/tty | gzip --stdout | nc -u -w 1 $ELK_IP 12201
```

To simulate a Filebeat input:

```bash
```

### By log type

Apache log are identified by existence of `command==apache2-foreground`:


```bash
echo '{"version": "1.1","host":"example.org","command": "apache2-foreground", "message":"173.245.62.180 - - [22/Jul/2016:13:55:29 +0000] \"GET /ELK HTTP/1.1\" 200 5899 \"https://wiki.initiumlab.com/w/index.php?title=ELK&action=edit\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36\""}' | tee /dev/tty | gzip --stdout | nc -u -w 1 $ELK_IP 12201
```

Nginx log are identified by `type==nginx-access`:

```bash
echo '{"version": "1.1","host":"example.org","type": "nginx-access", "message":"192.241.205.129 - - [19/Jul/2016:23:59:56 +0800] \"GET /article/20160719-dailynews-germany-train-axe/ HTTP/1.1\" 200 13840 \"-\" \"Mozilla/5.0 (Linux; Android 5.1; 8681-A01 Build/LMY47D) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/39.0.0.0 Mobile Safari/537.36\""}' | tee /dev/tty | gzip --stdout | nc -u -w 1 $ELK_IP 12201
```

