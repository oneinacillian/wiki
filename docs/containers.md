>More a more users attempt to run micro-services for their applications due to the scalability, less overhead and portability. It is also provides a great efficiency

# The following will be covered here
- Install Docker
- Configure and use a docker name volume
- Building and running a state history container
- Building and running an atomic container service
- Building and running a hyperion container service

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
sudo ln -s /apps/datavolume/testvolume /var/lib/docker/volumes/testvolume/_data
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

> Create your Dockerfile

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

> Build your docker images

```
docker build -t atomicimage:atomicimage -f ./Dockerfile .
```

> Map an external volume for your container persistent data

```
docker volume create atomictest
rm -rf /var/lib/docker/volumes/atomictest/_data/
mkdir /data/containers/atomictest
sudo ln -s /data/containers/atomictest /var/lib/docker/volumes/atomictest/_data
```

> Start up atomic container, publishing the ports necessary to expose the API to the community

```
docker run -d --name atomictest --publish 9000:9000 --publish 9001:9001 --mount source=atomictest,target=/data --tty atomictest:atomictest
```

> Exec into the container and start postgresql. Get your current data directory

```
postgres@52c2bc761f47:~$ psql
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=# SHOW data_directory;
       data_directory        
-----------------------------
 /var/lib/postgresql/14/main
(1 row)
```

In the above instance, you can see that the data directory is mapped inside the container in /var/lib
Ideally you want your data to persistent on a faster volume, which is not bound to the container

> Move the data directory and update postgresql config

```
service postgresql stop
cp -var /var/lib/postgresql/ /data/
nano /etc/postgresql/14/main/postgresql.conf
****
update data directory
data_directory = '/data/postgresql/14/main'             # use data in another directory
****
```

> Confirm that your postgresl directory is on the newly specified volume

```
root@36bbb3a2788d:/etc/postgresql/14/main# service postgresql start
 * Starting PostgreSQL 14 database server                                                                                                                                                                                                                                                                                                 [ OK ] 
root@36bbb3a2788d:/etc/postgresql/14/main# su - postgres
postgres@36bbb3a2788d:~$ psql
psql (14.4 (Ubuntu 14.4-1.pgdg20.04+1))
Type "help" for help.

postgres=# SHOW data_directory;
      data_directory      
--------------------------
 /data/postgresql/14/main
(1 row)
```

> Restore n atomic database

create the database which you would like to restore and attempt the restore

```
postgres=# CREATE DATABASE "api-wax-mainnet-atomic-1";
pg_restore --verbose -Fc -d api-wax-mainnet-atomic-1 -1 /data/backups/atomictest.dump
```

> **_NOTE:_** The backup was create by a guild member that used a specific role as the owner of the database

If you encounter issues with the restore process as such:
```

pg_restore: error: could not execute query: ERROR:  role "wecan_user" does not exist
```

Create a role with the same name the backup was created

```
postgres=# CREATE ROLE wecan_user;
```

Reattempt the restore and it should complete, unless you encounter issue with the backup

> View the database once restored by using psql

```
postgres=# \l+
                                                                       List of databases
           Name           |  Owner   | Encoding | Collate |  Ctype  |   Access privileges   |  Size   | Tablespace |                Description                 
--------------------------+----------+----------+---------+---------+-----------------------+---------+------------+--------------------------------------------
 api-wax-mainnet-atomic-1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 |                       | 20 GB   | pg_default | 
 postgres                 | postgres | UTF8     | C.UTF-8 | C.UTF-8 |                       | 8545 kB | pg_default | default administrative connection database
 template0                | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +| 8377 kB | pg_default | unmodifiable empty database
                          |          |          |         |         | postgres=CTc/postgres |         |            | 
 template1                | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +| 8529 kB | pg_default | default template for new databases
                          |          |          |         |         | postgres=CTc/postgres |         |            | 
(4 rows)
```

> Configure your eosio contract configuration files accordingly. 

- readers.config.json
- server.config.json
- connections.config.json (confirm that your postgresql authentication is configured properly as well as your ship settings)

> Start your atomic filler until is has caught up to the main block, then your API as well (run from within your /apps/eosio-contract-api folder).

```
pm2 start ecosystems.config.json --only eosio-contract-api-filler
pm2 start ecosystems.config.json --only eosio-contract-api-server
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
&& apt-get update && apt-get install kibana \
&& echo "server.host: 0.0.0.0" >> /etc/kibana/kibana.yml \
&& cred=$(cat credentials.file | awk '/PASSWORD kibana_system = /{print}' | cut -d '=' -f 2 | sed 's/ //') \
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

> build your docker image

```
docker build -t hyperiontest:latest -f ./Dockerfile .
```

> Map an external volume for your container persistent data

```
docker volume create hyperiontest
rm -rf /var/lib/docker/volumes/hyperiontest/_data/
mkdir /data/containers/hyperiontest
sudo ln -s /data/containers/hyperiontest /var/lib/docker/volumes/hyperiontest/_data
```

> Start up atomic container, publishing the ports necessary to expose the API to the community

```
docker run -d --name hyperiontest --publish 7000:7000 --publish 5601:5601 --mount source=hyperiontest,target=/data --tty hyperiontest:latest
```

> Execute to a terminal in the hyperion container

```
docker exec -ti hyperiontest /bin/bash
```

> Move the data and log volumes for elasticsearch to your named volume

```
chown -R elasticsearch:elasticsearch /data
mkdir -p /data/es-data
mkdir -p /data/es-logs
cp -var /var/lib/elasticsearch/* /data/es-data
cp -var /var/log/elasticsearch/* /data/es-logs
```

> Modify the configuration (elasticsearch) to reference the new volumes for data and log management (elasticsearch.yml)

```
path.data: /data/es-data
path.logs: /data/es-logs
```

> Startup elasticsearch and all dependant services. You will have a stop and startup.sh file located in /apps

```
bash /apps/startup.sh
```

Once all the application has been started, proceed to kibana through your web console on the port you expose (in this example, ::5601)

**_NOTE:_** Should you have trouble starting Kibana, reset the elastic passwords (yes | /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto > credentials.file, take not of the password and update your password reference in /etc/kibana/kibana.yml)

> Restore snapshot (if not indexing from the 1st block). In this example, I exposed a snapshot via http "http://23.88.71.224:8080/downloads/" and named the repository "gipi-repo". Perform this using DEV tools under management

Create repository
```
PUT _snapshot/gipi-repo
{
   "type": "url",
   "settings": {
       "url": "http://23.88.71.224:8080/downloads/",
       "max_restore_bytes_per_sec": "1gb",
       "max_snapshot_bytes_per_sec": "1gb"
   }
}
```

List all snapshots
```
GET _snapshot/gipi-repo/_all
```

Set the repository url as trusted by elasticsearch (modification in elasticsearch.yml) and restart elasticsearch
```
repositories.url.allowed_urls: "http://23.88.71.224:8080/downloads/"
``` 

Start a restore of a snapshot (DEV tools)
```
POST _snapshot/gipi-repo/daily_snapshot-qu4_uyc0tuotoaj1x3lsjq/_restore
{
  "indices": "*,-.*"
}
```

> Wait for the restore to complete, then configure your hyperion contract information

- /apps/hyperion-history-api/connections.json **NOTE** RabbitMQ password will be what you have defined in the Dockerfile build 
```
{
  "amqp": {
    "host": "127.0.0.1:5672",
    "api": "127.0.0.1:15672",
    "protocol": "http",
    "user": "<from dockerfile>",
    "pass": "<from dockerfile>",
    "vhost": "hyperion",
    "frameMax": "0x10000"
  },
  "elasticsearch": {
    "protocol": "http",
    "host": "127.0.0.1:9200",
    "ingest_nodes": [
      "127.0.0.1:9200"
    ],
    "user": "elastic",
    "pass": "<from credentials file in /apps>"
  },
  "redis": {
    "host": "127.0.0.1",
    "port": "6379"
  },
  "chains": {
    "wax": {
      "name": "Wax",
      "ship": "ws://172.168.40.201:10876",
      "http": "http://172.168.40.201:9888",
      "chain_id": "f16b1833c747c43682f4386fca9cbb327929334a762755ebec17f6f23c9b8a12",
      "WS_ROUTER_HOST": "127.0.0.1",
      "WS_ROUTER_PORT": 7001
    }
  }
}
```
- /apps/hyperion-history-api/chains/wax.config.json
```
{
  "api": {
    "enabled": true,
    "pm2_scaling": 1,
    "node_max_old_space_size": 1024,
    "chain_name": "wax",
    "server_addr": "172.168.40.100",
    "server_port": 7000,
    "server_name": "hyperion-test.oiac.io",
    "provider_name": "Oneinacillian",
    "provider_url": "https://oiac.io",
    "chain_api": "",
    "push_api": "",
    "chain_logo_url": "",
    "enable_caching": true,
    "cache_life": 1,
    "limits": {
      "get_actions": 1000,
      "get_voters": 100,
      "get_links": 1000,
      "get_deltas": 1000,
      "get_trx_actions": 200
    },
    "access_log": false,
    "chain_api_error_log": false,
    "custom_core_token": "",
    "enable_export_action": false,
    "disable_rate_limit": false,
    "rate_limit_rpm": 1000,
    "rate_limit_allow": [],
    "disable_tx_cache": false,
    "tx_cache_expiration_sec": 3600,
    "v1_chain_cache": [
      {
        "path": "get_block",
        "ttl": 3000
      },
      {
        "path": "get_info",
        "ttl": 500
      }
    ]
  },
  "indexer": {
    "enabled": true,
    "node_max_old_space_size": 4096,
    "start_on": 0,
    "stop_on": 0,
    "rewrite": false,
    "purge_queues": false,
    "live_reader": true,
    "live_only_mode": false,
    "abi_scan_mode": false,
    "fetch_block": true,
    "fetch_traces": true,
    "disable_reading": false,
    "disable_indexing": false,
    "process_deltas": true,
    "disable_delta_rm": true
  },
  "settings": {
    "preview": false,
    "chain": "wax",
    "eosio_alias": "eosio",
    "parser": "1.8",
    "auto_stop": 0,
    "index_version": "v1",
    "debug": false,
    "bp_logs": false,
    "bp_monitoring": false,
    "ipc_debug_rate": 60000,
    "allow_custom_abi": false,
    "rate_monitoring": true,
    "max_ws_payload_mb": 1024,
    "ds_profiling": false,
    "auto_mode_switch": false,
    "hot_warm_policy": false,
    "custom_policy": "",
    "bypass_index_map": true,
    "index_partition_size": 10000000,
    "es_replicas": 0
  },
  "blacklists": {
    "actions": [],
    "deltas": []
  },
  "whitelists": {
    "actions": [],
    "deltas": [],
    "max_depth": 10,
    "root_only": false
  },
  "scaling": {
    "readers": 1,
    "ds_queues": 1,
    "ds_threads": 1,
    "ds_pool_size": 1,
    "indexing_queues": 1,
    "ad_idx_queues": 1,
    "dyn_idx_queues": 1,
    "max_autoscale": 4,
    "batch_size": 5000,
    "resume_trigger": 5000,
    "auto_scale_trigger": 20000,
    "block_queue_limit": 10000,
    "max_queue_limit": 100000,
    "routing_mode": "round_robin",
    "polling_interval": 10000
  },
  "features": {
    "streaming": {
      "enable": true,
      "traces": true,
      "deltas": true
    },
    "tables": {
      "proposals": true,
      "accounts": true,
      "voters": true
    },
    "index_deltas": true,
    "index_transfer_memo": true,
    "index_all_deltas": true,
    "deferred_trx": false,
    "failed_trx": false,
    "resource_limits": false,
    "resource_usage": false
  },
  "prefetch": {
    "read": 50,
    "block": 100,
    "index": 500
  },
  "plugins": {}
}
```

> Start your api indexer to allow your data to index from the SHIP node

```
./run.sh wax-indexer
```

> Once it has indexed all the outstanding blocks, you can expose the api

```
./run.sh wax-api
``` 






