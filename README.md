
https://aws.plainenglish.io/visualizing-log-data-with-grafana-loki-and-promtail-f82179ed1215


# Lightweight lightweight centralized logs server


On docker swarm, To watch for docker containers on cluster:

```
version: "3.8"

services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:v0.49.1
    ports:
      - "8080:8080"
    networks:
      - proxy
    deploy:
      mode: global
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
networks:
  proxy:
    external: true

.................................................................

```

# To deploy loki
 
 Download loki-config:
```
wget https://raw.githubusercontent.com/grafana/loki/v2.8.0/cmd/loki/loki-local-config.yaml -O loki-config.yaml

```
Docker run loki container:
```
docker run -dit --name loki -v $(pwd):/mnt/config -p 3100:3100 grafana/loki:2.8.0 --config.file=/mnt/config/loki-config.yaml
```

# To deploy promtail

Download promtail-config:
```
wget  https://raw.githubusercontent.com/grafana/loki/v2.8.0/clients/cmd/promtail/promtail-docker-config.yaml -O promtail-config.yaml
```

Create mount point:
```
/opt/swarm-data/PLG/monitoring-logs
```
Docker run promtail container:
```
docker run -dit --name promtail -v $(pwd):/mnt/config -p 9080:9080 -v /opt/swarm-data/path/logs:/opt/swarm-data/path/logs -v /var/log:/var/log --link loki grafana/promtail:2.8.0 --config.file=/mnt/config/promtail-config.yaml

```

Deploy container for Grafana:
```
docker run -dit \
  -p 3000:3000 \
  --name=grafana \
  -e GF_SECURITY_ADMIN_USER=admin \
  -e GF_SECURITY_ADMIN_PASSWORD=changeme \
  -e TZ=Asia/Yangon \
  grafana/grafana-oss

``` 
  
  
Deploy container for Prometheus:

```
  docker run -dit \
  --name=prometheus \
  -p 9090:9090 \
  -v /opt/swarm-data/PLG/monitoring-logs/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```
Deploy container for Node-Exporter:

```

docker run -dit \
  --name=node-exporter \
  -p 9100:9100 \
  --pid="host" \
  -v /:/host:ro,rslave \
  prom/node-exporter \
  --path.rootfs=/host

```

To edit prometheus.yml file:
prometheus.yml:

```

global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['10.10.99.40:9090']

  - job_name: 'uat'
    static_configs:
      - targets: ['10.10.99.40:9100']
      - targets: ['10.10.99.41:9100']
      - targets: ['10.100.64.39:9100']

  - job_name: 'sit'
    static_configs:
      - targets: ['10.10.99.42:9100']
      - targets: ['10.10.99.43:9100']

  - job_name: 'prod'
    static_configs:
      - targets: ['10.100.64.37:9100']
      - targets: ['10.100.64.38:9100']
      - targets: ['10.100.64.50:9100']
      - targets: ['10.100.64.51:9100']

  # cAdvisor for per-container metrics
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['10.10.99.40:8080', '10.10.99.41:8080']

  - job_name: 'keycloak'
    metrics_path: /metrics
    static_configs:
      - targets: ['10.100.64.39:8050']

  - job_name: 'promtail'
    static_configs:
      - targets: ['10.10.99.40:9080']


```


