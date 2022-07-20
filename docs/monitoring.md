> Prometheus is a free software application used for event monitoring and alerting. It records real-time metrics in a time series database built using a HTTP pull model, with flexible queries and real-time alerting.

# *The following will be covered here*
- Manual Install prometheus on Ubuntu
- Add remote endpoint to allow prometheus to scrape proxy (haproxy) stats 
- Install and configure Grafana
- Import HA proxy dashboard to have visibility on query and history traffic
- Configure a telegram bot
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
apt-get install -y firewalld
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
sudo apt-get install -y grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server.service
firewall-cmd --add-port=3000/tcp --permanent
service firewalld reload
service grafana-server start
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

### *Configure a telegram bot*

1. Open Telegram, and create a new Bot by searching for @BotFather. Select the certified one as per the identification badge <br>

<img src="/assets/telegram - search botfather.png"/> <br>

2. Click on start <br>

<img src="/assets/telegram - click on start.png"/> <br>

3. Create new bot <br>

<img src="/assets/telegram - create a new bot.png"/> <br>

4. Choose a name for the bot <br>

<img src="/assets/telegram - select name for bot.png"/> <br>

5. Create username for the bot <br>

<img src="/assets/telegram - create username for bot.png"/> <br>

6. Take note of API token and bot link <br>

<img src="/assets/telegram - API token.png"/> <br>

7. Start the bot <br>

<img src="/assets/telegram - start bot.png"/> <br>
<img src="/assets/telegram - start bot2.png"/> <br> 

8. Create a new group in telegram <br>

<img src="/assets/telegram - create new group.png"/> <br> 

9. Add the bot to the group (find by bot link) <br>

<img src="/assets/telegram - add bot to group.png"/> <br> 

10. Select a group name of your choice (where you want to receive your alerts) <br>

<img src="/assets/telegram - name group.png"/> <br> 

11. Verify that the newly created group exists <br>

<img src="/assets/telegram - verify that group exists.png"/> <br> 

### *Configure Alerting in Grafana to post alert messages to telegram*

1. Proceed to alert setup <br>

<img src="/assets/telegram 1 - Grafana.png"/> <br>

1. Create a new contact point <br>

<img src="/assets/telegram 2 - Grafana.png"/> <br>

3. Select a name and type for telegram alert spec <br>

<img src="/assets/telegram 3 - Grafana.png"/> <br>

4. Enter you bot API token, as per step 6 of configuring your telegram bot (data should still be in your telegram)
5. Enter your chat ID for your group (can be gathered by viewing the group info) <span style="color:red">**will only work if you bot has been started. Step 7 on setting up a telegram bot**</span>
```
https://api.telegram.org/bot<YOUR_API_TOKEN_KEY>/getUpdates 
```
> Your chat ID with the numeric digits prefixed by the '-'

Alternatively, you can open telegram web, proceed to the group and retrieve your chat ID which is indicated by your url after the # sign

6. Configure your Message and notification settings (for auto resolving) as you please and save contact point

<img src="/assets/telegram 4 - Grafana.png"/> <br>










