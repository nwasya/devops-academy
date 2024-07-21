## Time to monitor

```
https://ourcodeworld.com/articles/read/1686/how-to-install-prometheus-node-exporter-on-ubuntu-2004
```

```yaml
version: '3.4'


services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: always
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/:/etc/prometheus/
      - prometheus:/prometheus
      - /etc/hosts:/etc/hosts:ro
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--log.level=debug'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=90d'
      - '--storage.tsdb.retention.size=50GB'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
      - '--web.enable-lifecycle'
    networks:
      - proxy
      - internal

  database-prometheus:
    image: prom/prometheus:v2.37.8
    restart: always
    volumes:
      - ./database-prometheus/:/etc/prometheus/
      - database-prometheus:/prometheus
      - /etc/hosts:/etc/hosts:ro
    ports:
      - "127.0.0.1:10901:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--log.level=debug'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=90d'
      - '--storage.tsdb.retention.size=50GB'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
      - '--web.enable-lifecycle'
    networks:
      - internal

  grafana:
    image: grafana/grafana:9.3.0
    container_name: grafana
    restart: always
    environment:
      GF_RENDERING_SERVER_URL: http://renderer:8081/render
#      GF_RENDERING_CALLBACK_URL: http://grafana:3000/
      GF_RENDERING_CALLBACK_URL: https://aysan.girt.ir
      GF_RENDERING_RENDERER_TOKEN: wk3zuejreRQG1hwn
      GF_LOG_FILTERS: "rendering:debug"
      RENDERING_VERBOSE_LOGGING: "true"
      RENDERING_DUMPIO: "true"
      GF_SERVER_ROOT_URL: https://aysan.girt.ir
    depends_on:
      - prometheus
    ports:
      - 3121:3000
    volumes:
      - grafana:/var/lib/grafana
#      - "./grafana_defaults.ini:/usr/share/grafana/conf/defaults.ini"
    networks:
      - proxy
      - internal
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=proxy"
      - "traefik.http.routers.grafana.rule=Host(`academy.girt.ir`)"
      - "traefik.http.services.grafana.loadbalancer.server.port=3000"
      # - "traefik.http.routers.grafana.middlewares=user-auth@file"
      - "traefik.http.routers.grafana.entrypoints=web"
      # - "traefik.http.routers.grafana.tls=true"
      # - "traefik.http.routers.grafana.tls.certresolver=cloudflare"
  renderer:
    image: grafana/grafana-image-renderer:3.6.2
#    ports:
#      - "8081:8081"
    container_name: renderer
    restart: always
    depends_on:
      - prometheus
    environment:
      AUTH_TOKEN: wk3zuejreRQG1hwn
      RENDERING_VERBOSE_LOGGING: "true"
      LOG_LEVEL: "debug"
      RENDERING_MODE: default
    networks:
      - internal

  alertmanager:
    image: prom/alertmanager
    container_name: alertmanager
    restart: always
    # ports:
    #   - "9093:9093"
    volumes:
      - ./alertmanager/:/etc/alertmanager/
    command:
      - '--config.file=/etc/alertmanager/config.yml'
      - '--storage.path=/alertmanager'
    networks:
      - proxy
      - internal


  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: always
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - --path.procfs=/host/proc
      - --path.sysfs=/host/sys
      - --collector.filesystem.ignored-mount-points
      - ^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)
    networks:
      - internal


networks:
  proxy:
    external: true
  internal:
    external: false

volumes:
  grafana:
    name: grafana
  prometheus:
    name: prometheus
  database-prometheus:
  ```


prometheus/prometheus.yml

  ```yaml
  
global:
  scrape_interval:     30s
  evaluation_interval: 30s

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'prom'

# Load and evaluate rules in this file every 'evaluation_interval' seconds.
rule_files:
  - 'alerts/*.rules'

# alert
alerting:
  alertmanagers:
  - scheme: http
    static_configs:
    - targets:
        - "alertmanager:9093"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']


  - job_name: 'alertmanager'
    static_configs:
      - targets: ['alertmanager:9093']

  - job_name: 'grafana'
    static_configs:
      - targets: ['grafana:3000']

  - job_name: 'node-exporter'
    static_configs:
      - targets: [
        # beta
        'node-exporter:9100' , 'myapp:9100']
  ```


database-prometheus/prometheus.yml
```yaml
global:
  scrape_interval:     30s
  evaluation_interval: 30s
  external_labels:
      monitor: 'prom'
rule_files:
  - 'alerts/*.rules'
alerting:
  alertmanagers:
  - scheme: http
    static_configs:
    - targets:
        - "alertmanager:9093"
```



alertmanager/config.yml
```yaml
global:
  # The smarthost and SMTP sender used for mail notifications.
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'eshraghi.aysan@gmail.com'
  smtp_auth_username: 'eshraghi.aysan@gmail.com'
  smtp_auth_password: 'asdasdasd'
  # smtp_auth_identity: 'USER-SENDER-MAILBOX'

route:
  group_by: ['instance', 'severity']
  group_wait: 30s
  group_interval: 30s
  repeat_interval: 30m
  receiver: 'production'

receivers:
- name: 'production'
  email_configs:
    - send_resolved: true
      to: 'eshraghi.aysan@gmail.com'

```

```bash
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle", instance="myapp:9100"}[5m])) * 100)

100 - ((node_memory_MemAvailable_bytes{instance="myapp:9100"} * 100) / node_memory_MemTotal_bytes{instance="myapp:9100"})
```