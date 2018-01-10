
### 以下是 information_schema 库里 RocksDB 表的具体描述:

#### ROCKSDB_CF_OPTIONS

| Field       | Type         | Null | Key | Default | Extra |
|-------------|--------------|------|-----|---------|-------|
| CF_NAME     | varchar(193) | NO   |     |         |       |
| OPTION_TYPE | varchar(193) | NO   |     |         |       |
| VALUE       | varchar(193) | NO   |     |         |  NULL |


#### ROCKSDB_CFSTATS

| Field     | Type         | Null | Key | Default | Extra |
|-----------|--------------|------|-----|---------|-------|
| CF_NAME   | varchar(193) | NO   |     |         |       |
| STAT_TYPE | varchar(193) | NO   |     |         |       |
| VALUE     | bigint(8)    | NO   |     | 0       |  NULL |


#### ROCKSDB_COMPACTION_STATS

| Field   | Type         | Null | Key | Default | Extra |
|---------|--------------|------|-----|---------|-------|
| CF_NAME | varchar(193) | NO   |     |         |       |
| LEVEL   | varchar(513) | NO   |     |         |       |
| TYPE    | varchar(513) | NO   |     |         |       |
| VALUE   | double       | NO   |     | 0       |  NULL |


#### ROCKSDB_DBSTATS

| Field     | Type         | Null | Key | Default | Extra |
|-----------|--------------|------|-----|---------|-------|
| STAT_TYPE | varchar(193) | NO   |     |         |       |
| VALUE     | bigint(8)    | NO   |     | 0       |  NULL |


#### ROCKSDB_DDL

| Field             | Type               | Null | Key | Default | Extra |
|-------------------|--------------------|------|-----|---------|-------|
| TABLE_SCHEMA      | varchar(193)       | NO   |     |         |       |
| TABLE_NAME        | varchar(193)       | NO   |     |         |       |
| PARTITION_NAME    | varchar(193)       | YES  |     | NULL    |       |
| INDEX_NAME        | varchar(193)       | NO   |     |         |       |
| COLUMN_FAMILY     | int(4)             | NO   |     | 0       |       |
| INDEX_NUMBER      | int(4)             | NO   |     | 0       |       |
| INDEX_TYPE        | smallint(2)        | NO   |     | 0       |       |
| KV_FORMAT_VERSION | smallint(2)        | NO   |     | 0       |       |
| TTL_DURATION      | bigint(8)          | NO   |     | 0       |       |
| INDEX_FLAGS       | bigint(8)          | NO   |     | 0       |       |
| CF                | varchar(193)       | NO   |     |         |       |
| AUTO_INCREMENT    | bigint(8) unsigned | YES  |     | NULL    |  NULL |


#### ROCKSDB_DEADLOCK

| Field          | Type         | Null | Key | Default | Extra |
|----------------|--------------|------|-----|---------|-------|
| DEADLOCK_ID    | bigint(8)    | NO   |     | 0       |       |
| TRANSACTION_ID | bigint(8)    | NO   |     | 0       |       |
| CF_NAME        | varchar(193) | NO   |     |         |       |
| WAITING_KEY    | varchar(513) | NO   |     |         |       |
| LOCK_TYPE      | varchar(193) | NO   |     |         |       |
| INDEX_NAME     | varchar(193) | NO   |     |         |       |
| TABLE_NAME     | varchar(193) | NO   |     |         |       |
| ROLLED_BACK    | bigint(8)    | NO   |     | 0       |  NULL |


#### ROCKSDB_GLOBAL_INFO

| Field | Type         | Null | Key | Default | Extra |
|-------|--------------|------|-----|---------|-------|
| TYPE  | varchar(513) | NO   |     |         |       |
| NAME  | varchar(513) | NO   |     |         |       |
| VALUE | varchar(513) | NO   |     |         |  NULL |


#### ROCKSDB_INDEX_FILE_MAP

| Field                | Type         | Null | Key | Default | Extra |
|----------------------|--------------|------|-----|---------|-------|
| COLUMN_FAMILY        | int(4)       | NO   |     | 0       |       |
| INDEX_NUMBER         | int(4)       | NO   |     | 0       |       |
| SST_NAME             | varchar(193) | NO   |     |         |       |
| NUM_ROWS             | bigint(8)    | NO   |     | 0       |       |
| DATA_SIZE            | bigint(8)    | NO   |     | 0       |       |
| ENTRY_DELETES        | bigint(8)    | NO   |     | 0       |       |
| ENTRY_SINGLEDELETES  | bigint(8)    | NO   |     | 0       |       |
| ENTRY_MERGES         | bigint(8)    | NO   |     | 0       |       |
| ENTRY_OTHERS         | bigint(8)    | NO   |     | 0       |       |
| DISTINCT_KEYS_PREFIX | varchar(400) | NO   |     |         |  NULL |


#### ROCKSDB_LOCKS

| Field            | Type         | Null | Key | Default | Extra |
|------------------|--------------|------|-----|---------|-------|
| COLUMN_FAMILY_ID | int(4)       | NO   |     | 0       |       |
| TRANSACTION_ID   | int(4)       | NO   |     | 0       |       |
| KEY              | varchar(513) | NO   |     |         |       |
| MODE             | varchar(32)  | NO   |     |         |  NULL |


#### ROCKSDB_PERF_CONTEXT

| Field          | Type         | Null | Key | Default | Extra |
|----------------|--------------|------|-----|---------|-------|
| TABLE_SCHEMA   | varchar(193) | NO   |     |         |       |
| TABLE_NAME     | varchar(193) | NO   |     |         |       |
| PARTITION_NAME | varchar(193) | YES  |     | NULL    |       |
| STAT_TYPE      | varchar(193) | NO   |     |         |       |
| VALUE          | bigint(8)    | NO   |     | 0       |  NULL |


#### ROCKSDB_PERF_CONTEXT_GLOBAL

| Field     | Type         | Null | Key | Default | Extra |
|-----------|--------------|------|-----|---------|-------|
| STAT_TYPE | varchar(193) | NO   |     |         |       |
| VALUE     | bigint(8)    | NO   |     | 0       |  NULL |


#### ROCKSDB_TF_OPTIONS

| Field       | Type         | Null | Key | Default | Extra |
|-------------|--------------|------|-----|---------|-------|
| CF_NAME     | varchar(193) | NO   |     |         |       |
| OPTION_NAME | varchar(193) | NO   |     |         |       |
| VALUE       | varchar(193) | NO   |     |         |  NULL |


#### ROCKSDB_TRX

| Field                    | Type         | Null | Key | Default | Extra |
|--------------------------|--------------|------|-----|---------|-------|
| TRANSACTION_ID           | bigint(8)    | NO   |     | 0       |       |
| STATE                    | varchar(193) | NO   |     |         |       |
| NAME                     | varchar(193) | NO   |     |         |       |
| WRITE_COUNT              | bigint(8)    | NO   |     | 0       |       |
| LOCK_COUNT               | bigint(8)    | NO   |     | 0       |       |
| TIMEOUT_SEC              | int(4)       | NO   |     | 0       |       |
| WAITING_KEY              | varchar(513) | NO   |     |         |       |
| WAITING_COLUMN_FAMILY_ID | int(4)       | NO   |     | 0       |       |
| IS_REPLICATION           | int(4)       | NO   |     | 0       |       |
| SKIP_TRX_API             | int(4)       | NO   |     | 0       |       |
| READ_ONLY                | int(4)       | NO   |     | 0       |       |
| HAS_DEADLOCK_DETECTION   | int(4)       | NO   |     | 0       |       |
| NUM_ONGOING_BULKLOAD     | int(4)       | NO   |     | 0       |       |
| THREAD_ID                | int(8)       | NO   |     | 0       |       |
| QUERY                    | varchar(193) | NO   |     |         |  NULL |

