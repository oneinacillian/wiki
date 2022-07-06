>Take note of ubuntu performance performance when configuring a Atomic instance

## The following will be covered here
- Create an indexing snapshot
- Restore a snapshot
- Optimize indexing operations for bulk processing
- Reset elastic credentials
- Upgrade Elasticsearch from 7.x to 8.x
- Recover Missing documents (failures during indexing operations)
- Recover Missing documents via a script

### Create an indexing snapshot
```
```
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
You can then have a copy of the following:
- apm_system
- kibana_system (configured in kibana.yml)
- kibana
- logstash_system
- beats_system
- remote_monitoring_user
- elastic (used to log into Kibana UI)
### Upgrade Elasticsearch from 7.x to 8.x
```
```
### Recover Missing documents manually (failures during indexing operations)
```
```
### Recover Missing documents via a script (failures during indexing operations)
```
```

  
