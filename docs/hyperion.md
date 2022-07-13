>Take note of ubuntu performance performance when configuring a Atomic instance

# The following will be covered here
- Create an indexing snapshot
- Restore a snapshot
- Optimize indexing operations for bulk processing
- Reset elastic credentials
- Upgrade Elasticsearch from 7.x to 8.x using upgrade assistant
- Recover Missing documents (failures during indexing operations)
- Recover Missing documents via a script
- pin up container on secondary host to participate in indexing operations
- Move the elastic data directory

### Create an indexing snapshot
- add path.repo: ["/data/snapshots"] to elasticsearch.yml where you would like to store your snapshots
- stop your hyperion indexer and api process
- restart elasticsearch
- log into kibana
- Using dev tools under management, create a snapshot repository
```
PUT /_snapshot/my_repository
{
  "type": "fs",
  "settings": {
    "location": "/data/snapshots"
  }
}
```
<img src="/assets/Configure snapshot repo - Elastic.png"/>
You should now see your repository being visible for snapshots
<img src="/assets/repo visible - Elastic.png"/>
- Create a daily snapshot policy
```
PUT /_slm/policy/daily_snapshot
{
  "schedule": "0 30 1 * * ?", 
  "name": "daily_snapshot", 
  "repository": "my_repository", 
  "config": { 
    "indices": ["wax-action-v1-000001"], 
    "ignore_unavailable": false,
    "include_global_state": false
  },
  "retention": { 
    "expire_after": "30d", 
    "min_count": 5, 
    "max_count": 50 
  }
}
```
You should now see the configure snapshot policy
<img src="/assets/snapshot policy - Elastic.png"/>
Trigger a run to make sure that the index/s you selected, backed up successfully
<img src="/assets/snapshot complete.png"/>

### Restore a snapshot
```
```
### Optimize indexing operations for bulk processing
```
```
### Reset elastic credentials
```
yes | /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto > credentials.file
```
You can then have a copy of the following stored in credentials.file

- apm_system
- kibana_system (configured in kibana.yml)
- kibana
- logstash_system
- beats_system
- remote_monitoring_user
- elastic (used to log into Kibana UI)
  
### Upgrade Elasticsearch from 7.x to 8.x
<img src="/assets/Upgrade Assistant - Elastic.png"/>
```
```
### Recover Missing documents manually (failures during indexing operations)
```
```
### Recover Missing documents via a script (failures during indexing operations)
```
```
### Spin up container on secondary host to participate in indexing operations
```
```

  
