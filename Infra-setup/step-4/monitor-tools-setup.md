

### Links to download Prometheus, Node_Exporter & black Box exporter https://prometheus.io/download/
### Links to download Grafana https://grafana.com/grafana/download
### Other link from video https://github.com/prometheus/blackbox_exporter


# prometheus setup

sudo apt update

sudo wget https://github.com/prometheus/prometheus/releases/download/v3.6.0-rc.1/prometheus-3.6.0-rc.1.linux-amd64.tar.gz

sudo tar -xvzf prometheus-3.6.0-rc.1.linux-amd64.tar.gz

sudo rm -rf prometheus-3.6.0-rc.1.linux-amd64.tar.gz

cd prometheus-3.6.0-rc.1.linux-amd64

./prometheus &

browse with your ip for prometheus Example http://ip:9090

# config the prometheus yaml to watch metrics on promethes
# prometheus.yml
---
static_configs:
      - targets: ["localhost:9090"]

  - job_name: 'node_exporter'
    static_configs:
      - targets: ["65.0.102.234:9100"]

  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ["65.0.102.234:8080"]


  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - http://prometheus.io    # Target to probe with http..
        - http://ip:31129 # Target to probe with http on port 8080.
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 15.207.84.111:9115  # The blackbox exporter's real hostname:port.
---


# grafana setup


sudo apt-get install -y adduser libfontconfig1 musl

wget https://dl.grafana.com/grafana-enterprise/release/12.1.1/grafana-enterprise_12.1.1_16903967602_linux_amd64.deb

sudo dpkg -i grafana-enterprise_12.1.1_16903967602_linux_amd64.deb

sudo /bin/systemctl daemon-reload

sudo /bin/systemctl enable grafana-server

sudo /bin/systemctl start grafana-server

browse with your ip for grafana Example http://ip:3000

# blackbox-exporter setup

sudo apt update

sudo wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.27.0/blackbox_exporter-0.27.0.linux-amd64.tar.gz

sudo tar -xvf blackbox_exporter-0.27.0.linux-amd64.tar.gz

sudo rm -rf blackbox_exporter-0.27.0.linux-amd64.tar.gz

cd blackbox_exporter-0.27.0.linux-amd64

./blackbox_exporter &

browse with your ip for black-box-exporter Example http://ip:9115

# node-exporter setup

# on jenkins server to check jenkins metrics

sudo apt udate

sudo wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz

sudo tar -xvf node_exporter-1.9.1.linux-amd64.tar.gz

sudo rm -rf node_exporter-1.9.1.linux-amd64.tar.gz

cd node_exporter-1.9.1.linux-amd64

./node_exporter &

browse with your ip for node-exporter Example http://ip:9100





