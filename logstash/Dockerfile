FROM logstash:5.5.2

RUN logstash-plugin install logstash-input-beats

COPY config /etc/logstash/conf.d
COPY patterns /opt/logstash/patterns

CMD ["logstash", "-f", "/etc/logstash/conf.d"]
