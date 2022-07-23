>More a more users attempt to run micro-services for their applications due to the scalability, less overhead and portability. It is also provides a great efficiency

# The following will be covered here
- Install Docker
- Configure and use a docker name volume
- Building and running a state history container
- Building and running an atomic container service - <span style="color:red">**not yet complete**</span>
- Building and running a hyperion container service - <span style="color:red">**not yet complete**</span>

## Install docker-ce (community edition)
- uninstall all previous docker utilities
- Add Docker’s official GPG key
- Setup up the repository
- Install latest version of docker
- Optional (select version of docker to be deployed)

### Uninstall all previous docker utilities
```
sudo apt-get remove docker docker-engine docker.io containerd runc
```
### Add Docker’s official GPG key
```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
### Set up the repository
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
### Install latest version of docker
```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
### Optional (select version of docker to be deployed)
```
apt-cache madison docker-ce
sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io docker-compose-plugin
```

## Configure and use a docker name volume

>For your data driven application which require optimized disk configuration (i.e zfs which is raided and optimized), it is best to configure a named docker volume and have it mounted inside your micro-service opposed to bind a volume for use. With docker volumes, the storage is not coupled to the lifecycle of the container, but instead exists outside of it. You’ll also find that volumes don’t increase the size of the Docker container using them. It also provides more flexible backups and data are easier to migrate.

### Create a docker volume
- Issue command with Docker CLI to created a volume
- Link up the volume to an external mount point
- Start a micro-service in daemon mode with access to the external volume

### Issue command with Docker CLI to created a volume
> **_NOTE:_** In this example, I created a docker volume named testvolume to use
```
docker volume create testvolume
```
### Link up the volume to an external mount point
**_NOTE:_** By default all volume will be located in /var/lib/docker/volumes if you have not previously configured your graph driver to point elsewhere. 

**_NOTE:_** In this example your optimized disk will be mounted in linux for example in /apps/datavolume
```
sudo rm -rf /var/lib/docker/volumes/testvolume/_data/
mkdir /apps/datavolume/testvolume
sudo ln -s /apps/datavolume/testvolume /var/lib/docker/volumes/testvolume/_data/
```
### Start a micro-service in daemon mode with access to the external volume
In this example: 

- The container name will be: testvolumecontainerservice
- The container image used will be ubuntu:22.04
- The named docker volume being used will be testvolume
- The names docker volume will be accessible by the container through /data
```
docker run -d --name testvolumecontainerservice --mount source=testvolume,target=/data --tty ubuntu:22.04
```

## Building and running a state history container

### Running State History in containers
- Proceed to a directory from which you would like to build your SHIP images
- Create the genesis file (testnet's one for example)
```
{
 "initial_timestamp": "2019-12-06T06:06:06.000",
 "initial_key": "EOS7PmWAXLBaqCzSgbq8cyr2HFztQpwBpXk3djBJA8fyoyUnYM37q",
 "initial_configuration": {
 "max_block_net_usage": 1048576,
 "target_block_net_usage_pct": 1000,
 "max_transaction_net_usage": 524288,
 "base_per_transaction_net_usage": 12,
 "net_usage_leeway": 500,
 "context_free_discount_net_usage_num": 20,
 "context_free_discount_net_usage_den": 100,
 "max_block_cpu_usage": 200000,
 "target_block_cpu_usage_pct": 2500,
 "max_transaction_cpu_usage": 150000,
 "min_transaction_cpu_usage": 100,
 "max_transaction_lifetime": 3600,
 "deferred_trx_expiration_window": 600,
 "max_transaction_delay": 3888000,
 "max_inline_action_size": 4096,
 "max_inline_action_depth": 6,
 "max_authority_depth": 6
 }
}
```
### Create a startup script: start.sh
```
#!/bin/bash

for x in /sys/devices/system/cpu/cpu*/cpufreq/;do
echo performance > $x/scaling_governor
done

DATADIR="/apps/waxdata"
NODEOSBINDIR="/apps/eosio/2.0/bin"
$DATADIR/stop.sh
echo -e "Starting Nodeos \n";


$NODEOSBINDIR/nodeos --disable-replay-opts --data-dir $DATADIR --config-dir $DATADIR --genesis-json=/apps/waxdata/genesis.json "$@" > $DATADIR/stdout.txt 2> $DATADIR/stderr.txt &  echo $! > $DATADIR/nodeos.pid
```
### Create a shutdown script: stop.sh
```
#!/bin/bash
DIR="/apps/waxdata"
 if [ -f $DIR"/nodeos.pid" ]; then
        pid=`cat $DIR"/nodeos.pid"`
        echo $pid
        kill $pid
        echo -ne "Stoping Nodeos"
        while true; do
            [ ! -d "/proc/$pid/fd" ] && break
            echo -ne "."
            sleep 1
        done
        rm -r $DIR"/nodeos.pid"
        DATE=$(date -d "now" +'%Y_%m_%d-%H_%M')
        if [ ! -d $DIR/logs ]; then
            mkdir $DIR/logs
        fi
        tar -pcvzf $DIR/logs/stderr-$DATE.txt.tar.gz stderr.txt stdout.txt
        echo -ne "\rNodeos Stopped.  \n"
    fi
```
### To make use of the Named Docker volume (testvolume as explained previously), create a config.ini file in your build directory and configure your blocks and state dir on the mounted volume within the container
```
blocks-dir = /data/blocks-2
state-history-dir = /data/state-history
chain-threads = 8
wasm-runtime = eos-vm-jit
eos-vm-oc-compile-threads = 4
...
...
...
```
### Create your Dockerfile
```
FROM ubuntu:18.04
#COPY 01norecommend /etc/apt/apt.conf.d
RUN apt-get update && apt -y install git && git clone https://github.com/worldwide-asset-exchange/wax-blockchain.git && cd wax-blockchain \
&& git checkout v2.0.12wax02 && git submodule update --init --recursive && cd scripts && yes | ./eosio_build.sh -P && ./eosio_install.sh && apt -y install vim \
&& mkdir -p /apps/waxdata && mv /wax-* /apps && mv /root/eosio* /apps
COPY config.ini genesis.json start.sh stop.sh /apps/waxdata/
RUN chmod +x /apps/waxdata/start.sh && chmod +x /apps/waxdata/stop.sh
```
Execute your build command
```
docker built -t yourrepo:yourtag -f ./yourdockerfile .
```
### To start your container and expose WS and HTTP port (example)
- name of container = wax2012wax02
- expose state history in config.ini on port 8888
- expose http port in config.ini on port 9876
- named docker volume named testvolume (follow instructions in previous steps)
- named volume mounted in container @ /data
- WAX nodeos image built and tagged as 2.0.12wax02:latest
```
docker run -d --name wax2012wax02 --publish 8888:8888 --publish 9876:9876 --mount source=testvolume,target=/data --tty 2.0.12wax02:latest
```
You can now start the nodeos process in the container
```
cd /apps/waxdata
./start.sh
```

## Building and running an atomic container service

### Create your Dockerfile
```
FROM ubuntu:20.04
WORKDIR /apps
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get -y install curl && curl -o file.sh https://deb.nodesource.com/setup_16.x && chmod +x file.sh && ./file.sh && apt-get install -y nodejs \
&& npm install pm2 -g && apt-get -y install wget ca-certificates && apt-get -y install wget && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - \
&& sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && apt-get update && apt-get -y install postgresql postgresql-contrib \
&& apt-get -y install vim && curl -fsSL https://packages.redis.io/gpg | gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg && echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/redis.list && apt-get update && apt-get -y install redis \
&& npm install --global yarn && apt-get -y install git && git clone https://github.com/pinknetworkx/eosio-contract-api.git && cd /apps/eosio-contract-api && yarn install
```

## Building and running an hyperion container service

### Create your Dockerfile
```
FROM ubuntu:20.04
WORKDIR /apps
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update \
&& apt-get -y upgrade \
&& apt -y install npm git wget curl vim htop systemctl aptitude git lxc-utils netfilter-persistent sysstat ntp gpg \
&& wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add - \
&& apt-get install apt-transport-https \
&& echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list \
&& apt-get update \
&& apt-get install elasticsearch \
&& git clone https://github.com/eosrio/hyperion-history-api.git --branch v3.3.5 \
&& echo "-Xms31g" >> /etc/elasticsearch/jvm.options \
&& echo "-Xmx31g" >> /etc/elasticsearch/jvm.options \
#&& sed -i 's/var\/lib\/elasticsearch/data\/es-data/g' /etc/elasticsearch/elasticsearch.yml \
#&& sed -i 's/var\/log\/elasticsearch/data\/es-logs/g' /etc/elasticsearch/elasticsearch.yml \
&& service elasticsearch start \
&& echo "xpack.security.enabled: true" >> /etc/elasticsearch/elasticsearch.yml \
&& service elasticsearch restart && sleep 20 \
&& yes | /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto > credentials.file \
&& wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add - \
&& apt-get install apt-transport-https \
&& echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee -a /etc/apt/sources.list.d/elastic-7.x.list \
&& apt-get update && apt-get install kibana \
&& echo "server.host: 0.0.0.0" >> /etc/kibana/kibana.yml \
&& cred=$(cat credentials.file | awk '/PASSWORD elastic = /{print}' | cut -d '=' -f 2 | sed 's/ //') \
&& echo 'elasticsearch.username: "kibana_system"' >> /etc/kibana/kibana.yml \
&& echo "elasticsearch.password: \"$cred\"" >> /etc/kibana/kibana.yml >> /etc/kibana/kibana.yml \
&& service kibana start \
&& curl -o nodefile.sh https://deb.nodesource.com/setup_16.x && chmod +x nodefile.sh \
&& apt-get install -y nodejs && node -v \
&& curl -fsSL https://packages.redis.io/gpg | gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg  \
&& echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb focal main" | tee /etc/apt/sources.list.d/redis.list \
&& apt-get update && apt-get -y install redis \
&& sed -i 's/supervised auto/supervised systemd/g' /etc/redis/redis.conf \
&& service redis-server start \
&& npm install pm2@latest -g \
&& pm2 startup \
&& apt-get install curl gnupg apt-transport-https -y \
&& curl -1sLf "https://keys.openpgp.org/vks/v1/by-fingerprint/0A9AF2115F4687BD29803A206B73A36E6026DFCA" | gpg --dearmor | tee /usr/share/keyrings/com.rabbitmq.team.gpg > /dev/null \
&& curl -1sLf "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0xf77f1eda57ebb1cc" | gpg --dearmor | tee /usr/share/keyrings/net.launchpad.ppa.rabbitmq.erlang.gpg > /dev/null \
&& curl -1sLf "https://packagecloud.io/rabbitmq/rabbitmq-server/gpgkey" | gpg --dearmor | tee /usr/share/keyrings/io.packagecloud.rabbitmq.gpg > /dev/null \
&& echo "deb [signed-by=/usr/share/keyrings/net.launchpad.ppa.rabbitmq.erlang.gpg] http://ppa.launchpad.net/rabbitmq/rabbitmq-erlang/ubuntu bionic main" >> /etc/apt/sources.list.d/rabbitmq.list \
&& echo "deb-src [signed-by=/usr/share/keyrings/net.launchpad.ppa.rabbitmq.erlang.gpg] http://ppa.launchpad.net/rabbitmq/rabbitmq-erlang/ubuntu bionic main" >> /etc/apt/sources.list.d/rabbitmq.list \
&& echo "deb [signed-by=/usr/share/keyrings/io.packagecloud.rabbitmq.gpg] https://packagecloud.io/rabbitmq/rabbitmq-server/ubuntu/ bionic main" >> /etc/apt/sources.list.d/rabbitmq.list \
&& echo "deb-src [signed-by=/usr/share/keyrings/io.packagecloud.rabbitmq.gpg] https://packagecloud.io/rabbitmq/rabbitmq-server/ubuntu/ bionic main" >> /etc/apt/sources.list.d/rabbitmq.list \
&& apt-get update -y \
&& apt-get install -y erlang-base erlang-asn1 erlang-crypto erlang-eldap erlang-ftp erlang-inets erlang-mnesia erlang-os-mon erlang-parsetools erlang-public-key erlang-runtime-tools erlang-snmp erlang-ssl erlang-syntax-tools erlang-tftp erlang-tools erlang-xmerl \
&& apt-get install rabbitmq-server -y --fix-missing \
&& service rabbitmq-server start \
&& rabbitmq-plugins enable rabbitmq_management \
&& rabbitmqctl add_vhost hyperion \
&& rabbitmqctl add_user hyper 123456 \
&& rabbitmqctl set_user_tags hyper administrator \
&& rabbitmqctl set_permissions -p hyperion hyper ".*" ".*" ".*" \
&& rabbitmqctl add_vhost /hyperion \
&& rabbitmqctl set_permissions -p /hyperion hyper ".*" ".*" ".*" \
&& cd /apps/hyperion-history-api && npm install \
&& echo "service elasticsearch start" >> /apps/startup.sh \
&& echo "service kibana start" >> /apps/startup.sh \
&& echo "service rabbitmq-server start" >> /apps/startup.sh \
&& echo "service redis-server start" >> /apps/startup.sh \
&& echo "service elasticsearch stop" >> /apps/stop.sh \
&& echo "service kibana stop" >> /apps/stop.sh \
&& echo "service rabbitmq-server stop" >> /apps/stop.sh \
&& echo "service redis-server stop" >> /apps/stop.sh \
&& chmod +x /apps/startup.sh \
&& chmod +x /apps/stop.sh
RUN apt-get install -y systemd
```
