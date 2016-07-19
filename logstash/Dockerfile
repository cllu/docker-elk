FROM logstash:2.3.4

RUN logstash-plugin install logstash-input-beats

COPY config /etc/logstash/conf.d

CMD ['logstash', '-f', '/etc/logstash/conf.d']