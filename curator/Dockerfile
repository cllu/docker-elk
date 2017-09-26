FROM ubuntu:14.04
RUN apt-get update -q && apt-get install -qy python-pip
RUN pip install elasticsearch-curator==5.2

ADD curator.yml /etc/curator/curator.yml
ADD actions.yml /etc/curator/actions.yml
ADD docker-entrypoint.sh /
RUN chmod +x /docker-entrypoint.sh

ADD tasks/purge-old-indices.sh /etc/periodic/


RUN printf "\n*\t*\t*\t*\t*\tbash /etc/periodic/purge-old-indices.sh" >> /var/spool/cron/crontabs/root
RUN printf "\n*\t*\t*\t*\t*\ttouch /tmp/cron-when-executed" >> /var/spool/cron/crontabs/root

ENTRYPOINT [ "/docker-entrypoint.sh"]
CMD ["cron", "-f"]
