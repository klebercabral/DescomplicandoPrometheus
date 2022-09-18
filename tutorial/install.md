Baixar binários:

```
wget https://github.com/prometheus/prometheus/releases/download/v2.38.0/prometheus-2.38.0.linux-arm64.tar.gz
```

Descompactar binários:

```
tar xf prometheus-2.38.0.linux-arm64.tar.gz
```

Mover binários:

```
sudo mv prometheus-2.38.0.linux-arm64/prometheus /usr/local/bin/prometheus
sudo mv prometheus-2.38.0.linux-arm64/promtool /usr/local/bin/promtool
```

Criar path da conf:

```
sudo mkdir /etc/prometheus
```

Mover consoles, console_libraries e o arquivo prometheus.yml:

```
sudo mv prometheus-2.38.0.linux-arm64/prometheus.yml /etc/prometheus/prometheus.yml
sudo mv prometheus-2.38.0.linux-arm64/consoles /etc/prometheus
sudo mv prometheus-2.38.0.linux-arm64/console_libraries /etc/prometheus
```

Editar o arquivo de configuração do Prometheus:

```
sudo vim /etc/prometheus/prometheus.yml
````

Deixar assim:


```
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  scrape_timeout: 10s
rule_files:
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "Meu Primeiro Exporter"
    static_configs:
      - targets: ["localhost:8899"]
```

Criar path do TSDB:

```
sudo mkdir /var/lib/prometheus
```

Criar grupo e user para o serviço:

```
sudo groupadd -r prometheus
sudo adduser -s /sbin/nologin -r -G prometheus prometheus
```

Criar arquivo do serviço:

```
sudo vim /etc/systemd/system/prometheus.service
```

Deixar assim:

```
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.external-url=

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
```

Configurar permissões nos path's:

```
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus
sudo chown -R prometheus:prometheus /usr/local/bin/prometheus
sudo chown -R prometheus:prometheus /usr/local/bin/promtool
```

Habilitar e iniciar serviço:

```
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus
```