> Prometheus is a free software application used for event monitoring and alerting. It records real-time metrics in a time series database built using a HTTP pull model, with flexible queries and real-time alerting.

# *The following will be covered here*
- Manual Install prometheus on Ubuntu
- Add remote endpoint to allow prometheus to scrape proxy (haproxy) stats 
- Install and configure Grafana
- Import HA proxy dashboard to have visibility on query and history traffic
- Configure a telegram bot - <span style="color:red">**not yet complete**</span>
- Configure Alerting in Grafana to post alert messages to telegram - <span style="color:red">**not yet complete**</span>
- Automate deploy using Ansible playbook - <span style="color:red">**not yet complete**</span>

### *Manual Install prometheus on Ubuntu*
```
#------install Prometheus begin------
export RELEASE="2.2.1"
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
cd /opt/
wget https://github.com/prometheus/prometheus/releases/download/v2.26.0/prometheus-2.26.0.linux-amd64.tar.gz
sha256sum prometheus-2.26.0.linux-amd64.tar.gz
tar -xvf prometheus-2.26.0.linux-amd64.tar.gz
cd prometheus-2.26.0.linux-amd64
sudo cp /opt/prometheus-2.26.0.linux-amd64/prometheus /usr/local/bin/
sudo cp /opt/prometheus-2.26.0.linux-amd64/promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
sudo cp -r /opt/prometheus-2.26.0.linux-amd64/consoles /etc/prometheus
sudo cp -r /opt/prometheus-2.26.0.linux-amd64/console_libraries /etc/prometheus
sudo cp -r /opt/prometheus-2.26.0.linux-amd64/prometheus.yml /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
sudo chown -R prometheus:prometheus /etc/prometheus/prometheus.yml
echo '
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target' > /etc/systemd/system/prometheus.service
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
apt install firewalld
firewall-cmd --add-port=9090/tcp --permanent
service firewalld reload
#------------------------------------
```
### *Add remote endpoint to allow prometheus to scrape proxy stats*
>Example use here will be the following setup from which you would like to scrape.

```
frontend stats
        mode http
        bind 172.168.30.10:8404
        option http-use-htx
        http-request use-service prometheus-exporter if { path /metrics }
        stats enable
        stats uri /stats
        stats refresh 10s
```
What you will need to have is your monitor instance connecting to the instance yo scrape via a private like (check wireguard setup in this documentation)
To start scrape the metrics which is exposed via proxy, add the following to your prometheus.yml located in /etc/prometheus/
```
  - job_name: wax_hyperion_test_ha_proxy
    static_configs:
      - targets: ['172.168.30.10:8404']
```
### *Install and configure Grafana*
>Grafana will be configured here to use prometheus as a datasource for viewing the time scaled data (data over time for as long as your retention is set on the datasource db for prometheus)

```
#------install Grafana begin------
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get install grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server.service
firewall-cmd --add-port=3000/tcp --permanent
service firewalld reload
#---------------------------------
```
> Connect to the Grafana interface via your public/private ip, or a DNS address you have configured

<img src="/assets/Login Grafana.png"/>

> Select to configure a prometheus datasource for your monitoring data

<img src="/assets/datasource Grafana.png"/>

> Configure your prometheus datasource

<img src="/assets/datasource 1.png"/> <br>
<img src="/assets/datasource 2.png"/> <br>

### *Import HA proxy dashboard to have visibility on query and history traffic*

> Load you haproxy2 dashboard

Please find the dashboard json to import [here](./assets/haproxy2full.json)

<img src="/assets/import1 - Grafana.png"/> <br>
<img src="/assets/import2 - Grafana.png"/> <br>
Continue with the prompts until the dashboard is installed

> The dashboard should be visible here

<img src="/assets/haproxy2 full - Grafana.png"/> <br>
<img src="/assets/haproxy_options - Grafana.png"/> <br>
You can expand any statistic to get a full representation of how your ingress/egress is performing, as well as health statistics of the proxy service












