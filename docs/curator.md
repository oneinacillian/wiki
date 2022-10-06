## Elasticsearch Curator

```
root@191b27da5fae:/apps# pip install elasticsearch-curator
Collecting elasticsearch-curator
  Downloading elasticsearch_curator-5.8.4-py2.py3-none-any.whl (106 kB)
     |████████████████████████████████| 106 kB 9.9 MB/s 
Collecting elasticsearch<8.0.0,>=7.12.0
  Downloading elasticsearch-7.17.6-py2.py3-none-any.whl (385 kB)
     |████████████████████████████████| 385 kB 70.5 MB/s 
Collecting requests>=2.25.1
  Downloading requests-2.28.1-py3-none-any.whl (62 kB)
     |████████████████████████████████| 62 kB 13.8 MB/s 
Collecting click<8.0,>=7.0
  Downloading click-7.1.2-py2.py3-none-any.whl (82 kB)
     |████████████████████████████████| 82 kB 11.1 MB/s 
Collecting urllib3==1.26.4
  Downloading urllib3-1.26.4-py2.py3-none-any.whl (153 kB)
     |████████████████████████████████| 153 kB 149.5 MB/s 
Collecting certifi>=2020.12.5
  Downloading certifi-2022.9.14-py3-none-any.whl (162 kB)
     |████████████████████████████████| 162 kB 147.8 MB/s 
Collecting requests-aws4auth>=1.0.1
  Downloading requests_aws4auth-1.1.2-py2.py3-none-any.whl (24 kB)
Collecting boto3>=1.17.57
  Downloading boto3-1.24.75-py3-none-any.whl (132 kB)
     |████████████████████████████████| 132 kB 137.8 MB/s 
Collecting six>=1.15.0
  Downloading six-1.16.0-py2.py3-none-any.whl (11 kB)
Collecting pyyaml==5.4.1
  Downloading PyYAML-5.4.1-cp38-cp38-manylinux1_x86_64.whl (662 kB)
     |████████████████████████████████| 662 kB 129.7 MB/s 
Collecting voluptuous>=0.12.1
  Downloading voluptuous-0.13.1-py3-none-any.whl (29 kB)
Collecting idna<4,>=2.5
  Downloading idna-3.4-py3-none-any.whl (61 kB)
     |████████████████████████████████| 61 kB 1.2 MB/s 
Collecting charset-normalizer<3,>=2
  Downloading charset_normalizer-2.1.1-py3-none-any.whl (39 kB)
Collecting jmespath<2.0.0,>=0.7.1
  Downloading jmespath-1.0.1-py3-none-any.whl (20 kB)
Collecting s3transfer<0.7.0,>=0.6.0
  Downloading s3transfer-0.6.0-py3-none-any.whl (79 kB)
     |████████████████████████████████| 79 kB 58.7 MB/s 
Collecting botocore<1.28.0,>=1.27.75
  Downloading botocore-1.27.75-py3-none-any.whl (9.1 MB)
     |████████████████████████████████| 9.1 MB 149.6 MB/s 
Collecting python-dateutil<3.0.0,>=2.1
  Downloading python_dateutil-2.8.2-py2.py3-none-any.whl (247 kB)
     |████████████████████████████████| 247 kB 143.9 MB/s 
Installing collected packages: certifi, urllib3, elasticsearch, idna, charset-normalizer, requests, click, six, requests-aws4auth, jmespath, python-dateutil, botocore, s3transfer, boto3, pyyaml, voluptuous, elasticsearch-curator
Successfully installed boto3-1.24.75 botocore-1.27.75 certifi-2022.9.14 charset-normalizer-2.1.1 click-7.1.2 elasticsearch-7.17.6 elasticsearch-curator-5.8.4 idna-3.4 jmespath-1.0.1 python-dateutil-2.8.2 pyyaml-5.4.1 requests-2.28.1 requests-aws4auth-1.1.2 s3transfer-0.6.0 six-1.16.0 urllib3-1.26.4 voluptuous-0.13.1
```

> Prepare your curator config and actions file

```
touch /root/.curator/curator.yml
touch /root/.curator/actions.yml
```

> Configure your curator for elastic interaction

```
client:
  hosts:
    - 127.0.0.1
  port: 9200
  url_prefix:
  use_ssl: false
  certificate:
  client_cert:
  client_key:
  ssl_no_validate: False
  username: ""
  password: ""
  timeout: 30
  master_only: False
logging:
  loglevel: INFO
  logfile:
  logformat: default
  blacklist: ['elasticsearch', 'urllib3']
```

> Configure your actions file to determine actions against indexes

```
actions:
  1:
    action: delete_indices
    description: >-
      Delete indices older than 4 hours (based on index name)
    options:
      ignore_empty_list: True
      disable_action: False
    filters:
    - filtertype: pattern
      kind: regex
      value: '^(wax-action-|wax-delta-).*$'
    - filtertype: age
      source: creation_date
      direction: older
      timestring: '%Y.%m.%d.%h'
      unit: hours
      unit_count: 5
```

> Perform a dry run to test which indexes will receive an action

```
curator /root/.curator/actions.yml --dry-run
```

-------------------------------------------- Test Setup -------------------------------------------- <br>
Recently did a full hyperion load from SHIP in the past 2 days to simulator the use of elastic curator <br>
<span style="color:red">NOTE</span>: Any actions performed against the index will be revelant on the age of the index. Which means <br>
if you restore all indexes using a snapshot or copy of metadata, age will be recent <br>
Test was performed using a container, on elastic 7.17.6 <br>
I am deleting indexes older than 2 days => <br>
-------------------------------------------- Test Setup -------------------------------------------- <br>

```
actions:
  1:
    action: delete_indices
    description: >-
      Delete indices older than 4 hours (based on index name)
    options:
      ignore_empty_list: True
      disable_action: False
    filters:
    - filtertype: pattern
      kind: regex
      value: '^(wax-action-|wax-delta-).*$'
    - filtertype: age
      source: creation_date
      direction: older
      timestring: '%Y.%m.%d.%h'
      unit: days
      unit_count: 2 
```

The result of this would be (in my case as I started the hyperion load approximately 2 days ago)

```
2022-09-18 17:45:46,210 INFO      DRY-RUN MODE.  No changes will be made.
2022-09-18 17:45:46,210 INFO      (CLOSED) indices may be shown that may not be acted on by action "delete_indices".
2022-09-18 17:45:46,210 INFO      DRY-RUN: delete_indices: wax-action-v1-000001 with arguments: {}
2022-09-18 17:45:46,210 INFO      DRY-RUN: delete_indices: wax-action-v1-000002 with arguments: {}
2022-09-18 17:45:46,210 INFO      DRY-RUN: delete_indices: wax-action-v1-000003 with arguments: {}
2022-09-18 17:45:46,210 INFO      DRY-RUN: delete_indices: wax-delta-v1-000001 with arguments: {}
2022-09-18 17:45:46,210 INFO      DRY-RUN: delete_indices: wax-delta-v1-000002 with arguments: {}
2022-09-18 17:45:46,210 INFO      DRY-RUN: delete_indices: wax-delta-v1-000003 with arguments: {}
2022-09-18 17:45:46,211 INFO      Action ID: 1, "delete_indices" completed.
2022-09-18 17:45:46,211 INFO      Job completed.
```

