>Take note of ubuntu performance performance when configuring a Atomic instance.
>Take note of postgresql documentation for database specific operations

# The following will be covered here
- All own identified issues experienced running Atomic API

## Troubleshooting rollback issues (These are issues we have encountered due to various fork and rollback events)
> We have experienced an incident where the rollback kept on failing and retrying, causing the Atomic Indexer to run out of sync<br>

### Example_1
Typical error in the .pm2/logs/eosio-contract-api-filler-out-0.log would be:

```
2023-07-21T07:31:39.226Z [PID:4145502] [info] : Executed rollback query 29 / 274 
2023-07-21T07:31:39.226Z [PID:4145502] [warn] : Fork rollback taking longer than expected. Executing query... {"operation":"delete","table":"contract_traces","values":null,"condition":{"str":"\"global_sequence\" = $1 AND \"account\" = $2","values":["82233095234","atomicassets"]}}
2023-07-21T07:40:45.070Z [PID:406251] [info] : Executed rollback query 1 / 274 
2023-07-21T07:40:45.070Z [PID:406251] [warn] : Fork rollback taking longer than expected. Executing query... {"operation":"delete","table":"contract_traces","values":null,"condition":{"str":"\"global_sequence\" = $1 AND \"account\" = $2","values":["82233095412","atomicassets"]}}
2023-07-21T07:45:50.141Z [PID:406251] [info] : Executed rollback query 2 / 274
```

I queried the database for public.contract_traces and found that I could not retrieve information for account 'atomicassets', ordered by 'global_sequence' and limit the return to 100 records as it would timeout. This indicated a performance issue on this particular table and thought could lead to the rollback failures

```
SELECT * FROM public.contract_traces 
WHERE "account" = 'atomicassets'
ORDER BY "global_sequence" DESC
LIMIT 100;
```
I then created an index named **idx_contract_traces_account_global_sequence** on public.contract_traces account and global_sequence fields, with the global_sequence fields being indexed in a descending order.
```
CREATE INDEX idx_contract_traces_account_global_sequence
ON public.contract_traces ("account", "global_sequence" DESC);
```
The index took approximately 45 minutes to create and the API filler immedaitely resumed blocks processing

