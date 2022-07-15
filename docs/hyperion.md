>Take note of ubuntu performance performance when configuring a Atomic instance

# The following will be covered here
- Create an indexing snapshot
- Restore a snapshot - <span style="color:red">**not yet complete**</span>
- Optimize indexing operations for bulk processing - <span style="color:red">**not yet complete**</span>
- Reset elastic credentials
- Upgrade Elasticsearch from 7.x to 8.x using upgrade assistant - <span style="color:red">**not yet complete**</span>
- Recover Missing documents via a script
- Recover Missing documents manually (failures during indexing operations) - <span style="color:red">**not yet complete**</span>
- pin up container on secondary host to participate in indexing operations - <span style="color:red">**not yet complete**</span>

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

### Recover Missing documents via a script which was provided by one of the Guild members. 
> It often can happen that during the indexing operation you encountered a component failure which causes the indexing operation to miss certain blocks during the indexing.

One of you valued community members has provided a python based utility to automate the recovery of documents which were lost during the indexing operations.

[Please follow this link as a first attempt to resolve all missing documents](https://github.com/eosrio/hyperion-history-api/tree/v3.3.5/scripts/fix_missing_blocks) 

To confirm that no block data is missing, connect to Kibana and open **dev tools** under management <br>
<img src="/assets/dev tools - Elastic.png"/> <br>
Run the following POST command to view the block histogram. **Note** that each bucket contains 10,000,000 documents <br>
This way you can also easily verify which bucket has missing data <br>
<img src="/assets/missing_block_1  - Elastic.png"/> <br>
```
POST wax-block-*/_search
{
  "aggs": {
    "block_histogram": {
      "histogram": {
        "field": "block_num",
        "interval": 10000000,
        "min_doc_count": 1
      },
      "aggs": {
        "max_block": {
          "max": {
            "field": "block_num"
          }
        }
      }
    }
  },
  "size": 0,
  "query": {
    "match_all": {}
  }
}
```
Check the block histogram on the results pane to see if any documents are missing. <br>
The expectation here is that each bucket prior to the bucket in which documents are indexed, are fully populated with 10,000,000 documents <br>
Run the wax indexer utility until you catch up with headblock the run the following to query hyperion health
```
http://hyperion.oiac.io/v2/health
```
The expected respone will be to have your **head_block_num**, **last_indexed_block** and **total_indexed_blocks** in sync <br>
<img src="/assets/elastic block status.png"/> <br>
Should this not be the case, start the manual recovery process as explained in the below section

### Recover Missing documents manually (failures during indexing operations)
 In this section, I will explain the manual process of finding the blocks and recovering the manually via the wax-indexer operations.

Determine the amount of documents stored in each bucket, which should contain 10000000 items
```
POST wax-block-*/_search
{
  "aggs": {
    "block_histogram": {
      "histogram": {
        "field": "block_num",
        "interval": 10000000,
        "min_doc_count": 1
      },
      "aggs": {
        "max_block": {
          "max": {
            "field": "block_num"
          }
        }
      }
    }
  },
  "size": 0,
  "query": {
    "match_all": {}
  }
} 
```
Result will look like follows =>
```
"aggregations" : {
    "block_histogram" : {
      "buckets" : [
        {
          "key" : 0.0,
          "doc_count" : 9999998,
          "max_block" : {
            "value" : 9999999.0
          }
        },
        {
          "key" : 1.0E7,
          "doc_count" : 10000000,
          "max_block" : {
            "value" : 1.9999999E7
          }
        },
        {
          "key" : 2.0E7,
          "doc_count" : 10000000,
          "max_block" : {
            "value" : 2.9999999E7
          }
        },
        {
          "key" : 3.0E7,
          "doc_count" : 10000000,
          "max_block" : {
            "value" : 3.9999999E7
          }
        },
        {
          "key" : 4.0E7,
          "doc_count" : 10000000,
          "max_block" : {
            "value" : 4.9999999E7
          }
        },
        {
          "key" : 5.0E7,
          "doc_count" : 10000000,
          "max_block" : {
            "value" : 5.9999999E7
          }
        },
        {
          "key" : 6.0E7,
          "doc_count" : 10000000,
          "max_block" : {
            "value" : 6.9999999E7
          }
        },
        {
          "key" : 7.0E7,
          "doc_count" : 10000000,
          "max_block" : {
            "value" : 7.9999999E7
          }
        },
        {
          "key" : 8.0E7,
          "doc_count" : 10000000,
          "max_block" : {
            "value" : 8.9999999E7
          }
        },
        {
          "key" : 9.0E7,
          "doc_count" : 10000000,
          "max_block" : {
            "value" : 9.9999999E7
          }
        },
        {
          "key" : 1.0E8,
          "doc_count" : 10000000,
          "max_block" : {
            "value" : 1.09999999E8
          }
        },
        {
          "key" : 1.1E8,
          "doc_count" : 10000000,
          "max_block" : {
            "value" : 1.19999999E8
          }
        },
        {
          "key" : 1.2E8,
          "doc_count" : 10000000,
          "max_block" : {
            "value" : 1.29999999E8
          }
        },
        {
          "key" : 1.3E8,
          "doc_count" : 10000000,
          "max_block" : {
            "value" : 1.39999999E8
          }
        },
        {
          "key" : 1.4E8,
          "doc_count" : 10000000,
          "max_block" : {
            "value" : 1.49999999E8
          }
        },
        {
          "key" : 1.5E8,
          "doc_count" : 10000000,
          "max_block" : {
            "value" : 1.59999999E8
          }
        },
        {
          "key" : 1.6E8,
          "doc_count" : 3904020,
          "max_block" : {
            "value" : 1.639114E8
          }
        }
      ]
    }
  }
```

### Spin up container on secondary host to participate in indexing operations

```
```

  
