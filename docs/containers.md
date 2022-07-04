>More a more users attempt to run micro-services for their applications due to the scalability, less overhead and portability. It is also provides a great efficiency

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

>For your data driven application which require optimized disk configuration (i.e zfs which is raided and optimized), it is best to configure a named docker volume and have it mounted inside your micro-service opposed to bind a volume for use. With docker volumes, the storage is not coupled to the lifecycle of the container, but instead exists outside of it. You’ll also find that volumes don’t increase the size of the Docker container using them. It also provides more flexible backups and data are easier to migrate.

## Create a docker volume
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

## Here are some examples of docker files we created to test node functionality in containers
- Running State History in containers
- Running Atomic-API in containers

### SHIP build artifact
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
- Create a startup script: start.sh
```
#!/bin/bash

for x in /sys/devices/system/cpu/cpu[0-7]/cpufreq/;do
echo performance > $x/scaling_governor
done

DATADIR="/apps/waxdata"
NODEOSBINDIR="/apps/eosio/2.0/bin"
$DATADIR/stop.sh
echo -e "Starting Nodeos \n";


$NODEOSBINDIR/nodeos --disable-replay-opts --data-dir $DATADIR --config-dir $DATADIR --genesis-json=/apps/waxdata/genesis.json "$@" > $DATADIR/stdout.txt 2> $DATADIR/stderr.txt &  echo $! > $DATADIR/nodeos.pid
```
- Create a shutdown script: stop.sh
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
- Create your Dockerfile
```
FROM ubuntu:18.04
#COPY 01norecommend /etc/apt/apt.conf.d
RUN apt-get update && apt -y install git && git clone https://github.com/worldwide-asset-exchange/wax-blockchain.git && cd wax-blockchain \
&& git checkout v2.0.12wax02 && git submodule update --init --recursive && cd scripts && yes | ./eosio_build.sh -P && ./eosio_install.sh && apt -y install vim \
&& mkdir -p /apps/waxdata && mv /wax-* /apps && mv /root/eosio* /apps
COPY config.ini genesis.json start.sh stop.sh /apps/waxdata/
RUN chmod +x /apps/waxdata/start.sh && chmod +x /apps/waxdata/stop.sh
```
- Execute your build command
```
docker built -t yourrepo:yourtag -f ./yourdockerfile .
```





