СТЕК МОНИТОРИНГА
<pre>
# Create dir and edit config
mkdir /GAP-1/prometheus_stack/prometheus/prometheus.yml 
nano /GAP-1/prometheus_stack/prometheus/prometheus.yml
</pre>

<pre>
# start docker-compose
 <code>
docker-compose up -d 
 </code>
</pre>

<pre>
# Config docker-compose.yml

version: '3.9'
 <code>
services:

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus:/etc/prometheus/
    container_name: prometheus
    hostname: prometheus
    command:
      - --config.file=/etc/prometheus/prometheus.yml
    ports:
      - 9090:9090
    restart: unless-stopped
    environment:
      TZ: "Europe/Moscow"
    networks:
      - default   
# exporters
  node-exporter:
    image: prom/node-exporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    container_name: exporter
    hostname: exporter
    command:
      - --path.procfs=/host/proc
      - --path.sysfs=/host/sys
      - --collector.filesystem.ignored-mount-points
      - ^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)
    ports:
      - 9100:9100
    restart: unless-stopped
    environment:
      TZ: "Europe/Moscow"
    networks:
      - default
      
# Grafana      
  grafana:
    image: grafana/grafana
    user: root
    depends_on:
      - prometheus
    ports:
      - 3000:3000
    volumes:
      - ./grafana:/var/lib/grafana
      - ./grafana/provisioning/:/etc/grafana/provisioning/
    container_name: grafana
    hostname: grafana
    restart: unless-stopped
    environment:
      TZ: "Europe/Moscow"
    networks:
      - default
      
# Alertmanager bot
  alertmanager-bot:
    command:
      - --alertmanager.url=http://alertmanager:9093
      - --log.level=info
      - --store=bolt
      - --bolt.path=/data/bot.db
      - --telegram.admin=11******
      - --telegram.token=65********:******************************************
    image: metalmatze/alertmanager-bot:0.4.3
    user: root
    ports:
      - 8080:8080
    container_name: alertmanager-bot
    hostname: alertmanager-bot
    environment:
      TZ: "Europe/Moscow"
    restart: unless-stopped
    volumes:
      - ./data:/data
    networks:
      - default
      
# Alermanager
  alertmanager:
    image: prom/alertmanager:v0.21.0
    user: root
    ports:
    - 127.0.0.1:9093:9093
    volumes:
      - ./alertmanager/:/etc/alertmanager/
    container_name: alertmanager
    hostname: alertmanager
    environment:
      TZ: "Europe/Moscow"
    restart: unless-stopped
    command:
      - '--config.file=/etc/alertmanager/config.yml'
      - '--storage.path=/etc/alertmanager/data'
    networks:
      - default
      
# Blackbox
  blackbox:
    image: prom/blackbox-exporter
    container_name: blackbox
    hostname: blackbox
    ports:
      - 9115:9115
    restart: unless-stopped
    command:
      - "--config.file=/etc/blackbox/blackbox.yml"
    volumes:
      - ./blackbox:/etc/blackbox
    environment:
      TZ: "Europe/Moscow"
    networks:
      - default
      
# Victoria
  victoria:
    image: prom/victoria-metrics
    container_name: victoria-metrics
    hostname: victoria-data
    ports:
      - 8428:8428
    restart: unless-stopped
    command:
      - "--config.file=/etc/victoria-data/vicrotia-data.yml"
    volumes:
      - ./victoria-metrics:/etc/victoria
    environment:
      TZ: "Europe/Moscow"
    networks:
      - default
      
# subnet docker
networks:
  default:
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16
 </code>
</pre> 


# Send metrics Victoria
<pre>
remote_write:
  - url: http://192.168.56.106:8428/api/v1/write
    queue_config:
      max_samples_per_send: 10000
      capacity: 20000
      max_shards: 30
<code>

# Victoria Create Container (192.168.56.106)
<pre>
 docker volume create victoria-metrics-data
<code>
 # Victoria UP Container (192.168.56.106)
<pre>
docker run -d \
  -v victoria-metrics-data:/victoria-metrics-data \
  -p 8428:8428 \
  --name victoria-metrics \
  victoriametrics/victoria-metrics:latest 

</code>
