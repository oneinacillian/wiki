>More a more users attempt to run micro-services for their applications due to the scalability, less overhead and portability. It is also provides a great efficiency

# Install docker-ce (community edition)
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

# Create a docker volume
> **_NOTE:_** In this example, I created a docker volume named testvolume to use
- Issue command with Docker CLI to created a volume
- Link up the volume to an external mount point

### Issue command with Docker CLI to created a volume
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





