Оповещение Прометея Графана

mkdir /opt/GAP-1 nano /opt/GAP-1/prometheus.yml

docker run -d -p 9090:9090 -v /opt/GAP-1/prometheus.yml:/etc/prometheus/prometheus.yml
-v /opt/GAP-1/alert.rules:/etc/prometheus/alert. правила
средна/прометей
