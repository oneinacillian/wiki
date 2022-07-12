>Locust is an open source load testing tool, which is usefull for load testing against your API's to understand throughput and mitigate API abuse

# The following will be covered here
- Stage the directory for locust
- Configure a basic test using the Locustfile.py
- Deploy Locust as a single container
- Run a load test using the Locust interface
- Deploy Locust Master as a microservices
- Deploy Locust Slaves as microservices
- Configure a load test against Atomic API
- Configure a load test against Hyperion API
- Configure a load test against History API

### Configure a basic test using the Locustfile.py
> You will stage the python module and import to Locust to define your test. In this example it will perform a single get method against your specific URL+(/v1/chain/get_info). Note that this file needs to be mounted when starting the container service
 
```
from locust import HttpUser, task, between

class QuickstartUser(HttpUser):
    wait_time = between(5, 9)
    @task
    def getAPI(self):
        self.client.get("/v1/chain/get_info")
        self.client.get("/v1/chain/get_table_rows")
        self.client.get("/v1/chain/get_info")
```
### Deploy Locust as a single container
```
docker run -p 8089:8089 -v $PWD:/mnt/locust locustio/locust -f /mnt/locust/locustfile.py
Response =>
[2022-07-12 18:59:48,317] 425d50cb7761/INFO/locust.main: Starting web interface at http://0.0.0.0:8089 (accepting connections from all network interfaces)
[2022-07-12 18:59:48,326] 425d50cb7761/INFO/locust.main: Starting Locust 2.10.1
```
- The application will bind to port 8089 which will automatically be exposed to the operating system
- If you would like to access the GUI for Locust remotely, be sure to enable it on your server firewall
  
```
firewall-cmd --add-port=8089/tcp --permanent
service firewalld reload
```
### Run a load test using the Locust interface
> You can access Locust by hitting your public/private ip on the exposed port above

<img src="/assets/locustfile.py.png"/>

> Run Locust

<img src="/assets/run locustfile.py.png"/>

> Change runtime parameters

<img src="/assets/runtime for locustfile.py.png"/>

> View results

<img src="/assets/results for locustfile.py.png"/>






