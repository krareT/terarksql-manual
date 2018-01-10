

### 以下是 information_schema 库里 InnoDB 表的具体描述:

#### INNODB_BUFFER_PAGE

| Field               | Type                | Null | Key | Default | Extra |
|---------------------|---------------------|------|-----|---------|-------|
| POOL_ID             | bigint(21) unsigned | NO   |     | 0       |       |
| BLOCK_ID            | bigint(21) unsigned | NO   |     | 0       |       |
| SPACE               | bigint(21) unsigned | NO   |     | 0       |       |
| PAGE_NUMBER         | bigint(21) unsigned | NO   |     | 0       |       |
| PAGE_TYPE           | varchar(64)         | YES  |     | NULL    |       |
| FLUSH_TYPE          | bigint(21) unsigned | NO   |     | 0       |       |
| FIX_COUNT           | bigint(21) unsigned | NO   |     | 0       |       |
| IS_HASHED           | varchar(3)          | YES  |     | NULL    |       |
| NEWEST_MODIFICATION | bigint(21) unsigned | NO   |     | 0       |       |
| OLDEST_MODIFICATION | bigint(21) unsigned | NO   |     | 0       |       |
| ACCESS_TIME         | bigint(21) unsigned | NO   |     | 0       |       |
| TABLE_NAME          | varchar(1024)       | YES  |     | NULL    |       |
| INDEX_NAME          | varchar(1024)       | YES  |     | NULL    |       |
| NUMBER_RECORDS      | bigint(21) unsigned | NO   |     | 0       |       |
| DATA_SIZE           | bigint(21) unsigned | NO   |     | 0       |       |
| COMPRESSED_SIZE     | bigint(21) unsigned | NO   |     | 0       |       |
| PAGE_STATE          | varchar(64)         | YES  |     | NULL    |       |
| IO_FIX              | varchar(64)         | YES  |     | NULL    |       |
| IS_OLD              | varchar(3)          | YES  |     | NULL    |       |
| FREE_PAGE_CLOCK     | bigint(21) unsigned | NO   |     | 0       | NULL  |


#### INNODB_BUFFER_PAGE_LRU

| Field               | Type                | Null | Key | Default | Extra |
|---------------------|---------------------|------|-----|---------|-------|
| POOL_ID             | bigint(21) unsigned | NO   |     | 0       |       |
| LRU_POSITION        | bigint(21) unsigned | NO   |     | 0       |       |
| SPACE               | bigint(21) unsigned | NO   |     | 0       |       |
| PAGE_NUMBER         | bigint(21) unsigned | NO   |     | 0       |       |
| PAGE_TYPE           | varchar(64)         | YES  |     | NULL    |       |
| FLUSH_TYPE          | bigint(21) unsigned | NO   |     | 0       |       |
| FIX_COUNT           | bigint(21) unsigned | NO   |     | 0       |       |
| IS_HASHED           | varchar(3)          | YES  |     | NULL    |       |
| NEWEST_MODIFICATION | bigint(21) unsigned | NO   |     | 0       |       |
| OLDEST_MODIFICATION | bigint(21) unsigned | NO   |     | 0       |       |
| ACCESS_TIME         | bigint(21) unsigned | NO   |     | 0       |       |
| TABLE_NAME          | varchar(1024)       | YES  |     | NULL    |       |
| INDEX_NAME          | varchar(1024)       | YES  |     | NULL    |       |
| NUMBER_RECORDS      | bigint(21) unsigned | NO   |     | 0       |       |
| DATA_SIZE           | bigint(21) unsigned | NO   |     | 0       |       |
| COMPRESSED_SIZE     | bigint(21) unsigned | NO   |     | 0       |       |
| COMPRESSED          | varchar(3)          | YES  |     | NULL    |       |
| IO_FIX              | varchar(64)         | YES  |     | NULL    |       |
| IS_OLD              | varchar(3)          | YES  |     | NULL    |       |
| FREE_PAGE_CLOCK     | bigint(21) unsigned | NO   |     | 0       | NULL  |


#### INNODB_BUFFER_POOL_STATS

| Field                            | Type                | Null | Key | Default | Extra |
|----------------------------------|---------------------|------|-----|---------|-------|
| POOL_ID                          | bigint(21) unsigned | NO   |     | 0       |       |
| POOL_SIZE                        | bigint(21) unsigned | NO   |     | 0       |       |
| FREE_BUFFERS                     | bigint(21) unsigned | NO   |     | 0       |       |
| DATABASE_PAGES                   | bigint(21) unsigned | NO   |     | 0       |       |
| OLD_DATABASE_PAGES               | bigint(21) unsigned | NO   |     | 0       |       |
| MODIFIED_DATABASE_PAGES          | bigint(21) unsigned | NO   |     | 0       |       |
| PENDING_DECOMPRESS               | bigint(21) unsigned | NO   |     | 0       |       |
| PENDING_READS                    | bigint(21) unsigned | NO   |     | 0       |       |
| PENDING_FLUSH_LRU                | bigint(21) unsigned | NO   |     | 0       |       |
| PENDING_FLUSH_LIST               | bigint(21) unsigned | NO   |     | 0       |       |
| PAGES_MADE_YOUNG                 | bigint(21) unsigned | NO   |     | 0       |       |
| PAGES_NOT_MADE_YOUNG             | bigint(21) unsigned | NO   |     | 0       |       |
| PAGES_MADE_YOUNG_RATE            | double              | NO   |     | 0       |       |
| PAGES_MADE_NOT_YOUNG_RATE        | double              | NO   |     | 0       |       |
| NUMBER_PAGES_READ                | bigint(21) unsigned | NO   |     | 0       |       |
| NUMBER_PAGES_CREATED             | bigint(21) unsigned | NO   |     | 0       |       |
| NUMBER_PAGES_WRITTEN             | bigint(21) unsigned | NO   |     | 0       |       |
| PAGES_READ_RATE                  | double              | NO   |     | 0       |       |
| PAGES_CREATE_RATE                | double              | NO   |     | 0       |       |
| PAGES_WRITTEN_RATE               | double              | NO   |     | 0       |       |
| NUMBER_PAGES_GET                 | bigint(21) unsigned | NO   |     | 0       |       |
| HIT_RATE                         | bigint(21) unsigned | NO   |     | 0       |       |
| YOUNG_MAKE_PER_THOUSAND_GETS     | bigint(21) unsigned | NO   |     | 0       |       |
| NOT_YOUNG_MAKE_PER_THOUSAND_GETS | bigint(21) unsigned | NO   |     | 0       |       |
| NUMBER_PAGES_READ_AHEAD          | bigint(21) unsigned | NO   |     | 0       |       |
| NUMBER_READ_AHEAD_EVICTED        | bigint(21) unsigned | NO   |     | 0       |       |
| READ_AHEAD_RATE                  | double              | NO   |     | 0       |       |
| READ_AHEAD_EVICTED_RATE          | double              | NO   |     | 0       |       |
| LRU_IO_TOTAL                     | bigint(21) unsigned | NO   |     | 0       |       |
| LRU_IO_CURRENT                   | bigint(21) unsigned | NO   |     | 0       |       |
| UNCOMPRESS_TOTAL                 | bigint(21) unsigned | NO   |     | 0       |       |
| UNCOMPRESS_CURRENT               | bigint(21) unsigned | NO   |     | 0       |       |


#### INNODB_CMP

| Field                      | Type    | Null | Key | Default | Extra |
|----------------------------|---------|------|-----|---------|-------|
| page_size                  | int(5)  | NO   |     | 0       |       |
| compress_ops               | int(11) | NO   |     | 0       |       |
| compress_ops_ok            | int(11) | NO   |     | 0       |       |
| compress_time              | int(11) | NO   |     | 0       |       |
| compress_ok_time           | int(11) | NO   |     | 0       |       |
| compress_primary_ops       | int(11) | NO   |     | 0       |       |
| compress_primary_ops_ok    | int(11) | NO   |     | 0       |       |
| compress_primary_time      | int(11) | NO   |     | 0       |       |
| compress_primary_ok_time   | int(11) | NO   |     | 0       |       |
| compress_secondary_ops     | int(11) | NO   |     | 0       |       |
| compress_secondary_ops_ok  | int(11) | NO   |     | 0       |       |
| compress_secondary_time    | int(11) | NO   |     | 0       |       |
| compress_secondary_ok_time | int(11) | NO   |     | 0       |       |
| uncompress_ops             | int(11) | NO   |     | 0       |       |
| uncompress_time            | int(11) | NO   |     | 0       |       |
| uncompress_primary_ops     | int(11) | NO   |     | 0       |       |
| uncompress_primary_time    | int(11) | NO   |     | 0       |       |
| uncompress_secondary_ops   | int(11) | NO   |     | 0       |       |
| uncompress_secondary_time  | int(11) | NO   |     | 0       |       |


#### INNODB_CMPMEM

| Field                | Type       | Null | Key | Default | Extra |
|----------------------|------------|------|-----|---------|-------|
| page_size            | int(5)     | NO   |     | 0       |       |
| buffer_pool_instance | int(11)    | NO   |     | 0       |       |
| pages_used           | int(11)    | NO   |     | 0       |       |
| pages_free           | int(11)    | NO   |     | 0       |       |
| relocation_ops       | bigint(21) | NO   |     | 0       |       |
| relocation_time      | int(11)    | NO   |     | 0       |       |


#### INNODB_CMPMEM_RESET

| Field                | Type       | Null | Key | Default | Extra |
|----------------------|------------|------|-----|---------|-------|
| page_size            | int(5)     | NO   |     | 0       |       |
| buffer_pool_instance | int(11)    | NO   |     | 0       |       |
| pages_used           | int(11)    | NO   |     | 0       |       |
| pages_free           | int(11)    | NO   |     | 0       |       |
| relocation_ops       | bigint(21) | NO   |     | 0       |       |
| relocation_time      | int(11)    | NO   |     | 0       |       |


#### INNODB_CMP_PER_INDEX

| Field           | Type         | Null | Key | Default | Extra |
|-----------------|--------------|------|-----|---------|-------|
| database_name   | varchar(192) | NO   |     |         |       |
| table_name      | varchar(192) | NO   |     |         |       |
| index_name      | varchar(192) | NO   |     |         |       |
| compress_ops    | int(11)      | NO   |     | 0       |       |
| compress_ops_ok | int(11)      | NO   |     | 0       |       |
| compress_time   | int(11)      | NO   |     | 0       |       |
| uncompress_ops  | int(11)      | NO   |     | 0       |       |
| uncompress_time | int(11)      | NO   |     | 0       |       |


#### INNODB_CMP_PER_INDEX_RESET

| Field           | Type         | Null | Key | Default | Extra |
|-----------------|--------------|------|-----|---------|-------|
| database_name   | varchar(192) | NO   |     |         |       |
| table_name      | varchar(192) | NO   |     |         |       |
| index_name      | varchar(192) | NO   |     |         |       |
| compress_ops    | int(11)      | NO   |     | 0       |       |
| compress_ops_ok | int(11)      | NO   |     | 0       |       |
| compress_time   | int(11)      | NO   |     | 0       |       |
| uncompress_ops  | int(11)      | NO   |     | 0       |       |
| uncompress_time | int(11)      | NO   |     | 0       |       |


#### INNODB_CMP_RESET

| Field                      | Type    | Null | Key | Default | Extra |
|----------------------------|---------|------|-----|---------|-------|
| page_size                  | int(5)  | NO   |     | 0       |       |
| compress_ops               | int(11) | NO   |     | 0       |       |
| compress_ops_ok            | int(11) | NO   |     | 0       |       |
| compress_time              | int(11) | NO   |     | 0       |       |
| compress_ok_time           | int(11) | NO   |     | 0       |       |
| compress_primary_ops       | int(11) | NO   |     | 0       |       |
| compress_primary_ops_ok    | int(11) | NO   |     | 0       |       |
| compress_primary_time      | int(11) | NO   |     | 0       |       |
| compress_primary_ok_time   | int(11) | NO   |     | 0       |       |
| compress_secondary_ops     | int(11) | NO   |     | 0       |       |
| compress_secondary_ops_ok  | int(11) | NO   |     | 0       |       |
| compress_secondary_time    | int(11) | NO   |     | 0       |       |
| compress_secondary_ok_time | int(11) | NO   |     | 0       |       |
| uncompress_ops             | int(11) | NO   |     | 0       |       |
| uncompress_time            | int(11) | NO   |     | 0       |       |
| uncompress_primary_ops     | int(11) | NO   |     | 0       |       |
| uncompress_primary_time    | int(11) | NO   |     | 0       |       |
| uncompress_secondary_ops   | int(11) | NO   |     | 0       |       |
| uncompress_secondary_time  | int(11) | NO   |     | 0       |       |


#### INNODB_FILE_STATUS

| Field          | Type                | Null | Key | Default | Extra |
|----------------|---------------------|------|-----|---------|-------|
| FILE           | varchar(512)        | NO   |     |         |       |
| OPERATION      | varchar(512)        | NO   |     |         |       |
| REQUESTS       | bigint(21) unsigned | NO   |     | 0       |       |
| SLOW           | bigint(21) unsigned | NO   |     | 0       |       |
| BYTES          | bigint(21) unsigned | NO   |     | 0       |       |
| BYTES/R        | double              | NO   |     | 0       |       |
| SVC:SECS       | double              | NO   |     | 0       |       |
| SVC:MSECS/R    | double              | NO   |     | 0       |       |
| SVC:MAX_MSECS  | bigint(21) unsigned | NO   |     | 0       |       |
| WAIT:SECS      | double              | NO   |     | 0       |       |
| WAIT:MSECS/R   | double              | NO   |     | 0       |       |
| WAIT:MAX_MSECS | bigint(21) unsigned | NO   |     | 0       |       |


#### INNODB_FT_BEING_DELETED

| Field  | Type                | Null | Key | Default | Extra |
|--------|---------------------|------|-----|---------|-------|
| DOC_ID | bigint(21) unsigned | NO   |     | 0       |       |


#### INNODB_FT_CONFIG

| Field | Type         | Null | Key | Default | Extra |
|-------|--------------|------|-----|---------|-------|
| KEY   | varchar(193) | NO   |     |         |       |
| VALUE | varchar(193) | NO   |     |         |       |


#### INNODB_FT_DEFAULT_STOPWORD

| Field | Type        | Null | Key | Default | Extra |
|-------|-------------|------|-----|---------|-------|
| value | varchar(18) | NO   |     |         |       |


#### INNODB_FT_DELETED

| Field  | Type                | Null | Key | Default | Extra |
|--------|---------------------|------|-----|---------|-------|
| DOC_ID | bigint(21) unsigned | NO   |     | 0       |       |


#### INNODB_FT_INDEX_CACHE

| Field        | Type                | Null | Key | Default | Extra |
|--------------|---------------------|------|-----|---------|-------|
| WORD         | varchar(337)        | NO   |     |         |       |
| FIRST_DOC_ID | bigint(21) unsigned | NO   |     | 0       |       |
| LAST_DOC_ID  | bigint(21) unsigned | NO   |     | 0       |       |
| DOC_COUNT    | bigint(21) unsigned | NO   |     | 0       |       |
| DOC_ID       | bigint(21) unsigned | NO   |     | 0       |       |
| POSITION     | bigint(21) unsigned | NO   |     | 0       |       |


#### INNODB_FT_INDEX_TABLE

| Field        | Type                | Null | Key | Default | Extra |
|--------------|---------------------|------|-----|---------|-------|
| WORD         | varchar(337)        | NO   |     |         |       |
| FIRST_DOC_ID | bigint(21) unsigned | NO   |     | 0       |       |
| LAST_DOC_ID  | bigint(21) unsigned | NO   |     | 0       |       |
| DOC_COUNT    | bigint(21) unsigned | NO   |     | 0       |       |
| DOC_ID       | bigint(21) unsigned | NO   |     | 0       |       |
| POSITION     | bigint(21) unsigned | NO   |     | 0       |       |


#### INNODB_LOCKS

| Field       | Type                | Null | Key | Default | Extra |
|-------------|---------------------|------|-----|---------|-------|
| lock_id     | varchar(81)         | NO   |     |         |       |
| lock_trx_id | varchar(18)         | NO   |     |         |       |
| lock_mode   | varchar(32)         | NO   |     |         |       |
| lock_type   | varchar(32)         | NO   |     |         |       |
| lock_table  | varchar(1024)       | NO   |     |         |       |
| lock_index  | varchar(1024)       | YES  |     | NULL    |       |
| lock_space  | bigint(21) unsigned | YES  |     | NULL    |       |
| lock_page   | bigint(21) unsigned | YES  |     | NULL    |       |
| lock_rec    | bigint(21) unsigned | YES  |     | NULL    |       |
| lock_data   | varchar(8192)       | YES  |     | NULL    |       |


#### INNODB_LOCK_WAITS

| Field             | Type        | Null | Key | Default | Extra |
|-------------------|-------------|------|-----|---------|-------|
| requesting_trx_id | varchar(18) | NO   |     |         |       |
| requested_lock_id | varchar(81) | NO   |     |         |       |
| blocking_trx_id   | varchar(18) | NO   |     |         |       |
| blocking_lock_id  | varchar(81) | NO   |     |         |       |


#### INNODB_METRICS

| Field           | Type         | Null | Key | Default | Extra |
|-----------------|--------------|------|-----|---------|-------|
| NAME            | varchar(193) | NO   |     |         |       |
| SUBSYSTEM       | varchar(193) | NO   |     |         |       |
| COUNT           | bigint(21)   | NO   |     | 0       |       |
| MAX_COUNT       | bigint(21)   | YES  |     | NULL    |       |
| MIN_COUNT       | bigint(21)   | YES  |     | NULL    |       |
| AVG_COUNT       | double       | YES  |     | NULL    |       |
| COUNT_RESET     | bigint(21)   | NO   |     | 0       |       |
| MAX_COUNT_RESET | bigint(21)   | YES  |     | NULL    |       |
| MIN_COUNT_RESET | bigint(21)   | YES  |     | NULL    |       |
| AVG_COUNT_RESET | double       | YES  |     | NULL    |       |
| TIME_ENABLED    | datetime     | YES  |     | NULL    |       |
| TIME_DISABLED   | datetime     | YES  |     | NULL    |       |
| TIME_ELAPSED    | bigint(21)   | YES  |     | NULL    |       |
| TIME_RESET      | datetime     | YES  |     | NULL    |       |
| STATUS          | varchar(193) | NO   |     |         |       |
| TYPE            | varchar(193) | NO   |     |         |       |
| COMMENT         | varchar(193) | NO   |     |         |       |


#### INNODB_SYS_COLUMNS

| Field    | Type                | Null | Key | Default | Extra |
|----------|---------------------|------|-----|---------|-------|
| TABLE_ID | bigint(21) unsigned | NO   |     | 0       |       |
| NAME     | varchar(193)        | NO   |     |         |       |
| POS      | bigint(21) unsigned | NO   |     | 0       |       |
| MTYPE    | int(11)             | NO   |     | 0       |       |
| PRTYPE   | int(11)             | NO   |     | 0       |       |
| LEN      | int(11)             | NO   |     | 0       |       |


#### INNODB_SYS_DATAFILES

| Field | Type             | Null | Key | Default | Extra |
|-------|------------------|------|-----|---------|-------|
| SPACE | int(11) unsigned | NO   |     | 0       |       |
| PATH  | varchar(4000)    | NO   |     |         |       |


#### INNODB_SYS_DOCSTORE_FIELDS

| Field         | Type                | Null | Key | Default | Extra |
|---------------|---------------------|------|-----|---------|-------|
| INDEX_ID      | bigint(21) unsigned | NO   |     | 0       |       |
| POS           | int(11) unsigned    | NO   |     | 0       |       |
| DOCUMENT_PATH | blob                | YES  |     | NULL    |       |
| DOCUMENT_TYPE | int(11) unsigned    | NO   |     | 0       |       |


#### INNODB_SYS_FIELDS

| Field    | Type                | Null | Key | Default | Extra |
|----------|---------------------|------|-----|---------|-------|
| INDEX_ID | bigint(21) unsigned | NO   |     | 0       |       |
| NAME     | varchar(193)        | NO   |     |         |       |
| POS      | int(11) unsigned    | NO   |     | 0       |       |


#### INNODB_SYS_FOREIGN

| Field    | Type             | Null | Key | Default | Extra |
|----------|------------------|------|-----|---------|-------|
| ID       | varchar(193)     | NO   |     |         |       |
| FOR_NAME | varchar(193)     | NO   |     |         |       |
| REF_NAME | varchar(193)     | NO   |     |         |       |
| N_COLS   | int(11) unsigned | NO   |     | 0       |       |
| TYPE     | int(11) unsigned | NO   |     | 0       |       |


#### INNODB_SYS_FOREIGN_COLS

| Field        | Type             | Null | Key | Default | Extra |
|--------------|------------------|------|-----|---------|-------|
| ID           | varchar(193)     | NO   |     |         |       |
| FOR_COL_NAME | varchar(193)     | NO   |     |         |       |
| REF_COL_NAME | varchar(193)     | NO   |     |         |       |
| POS          | int(11) unsigned | NO   |     | 0       |       |


#### INNODB_SYS_INDEXES

| Field    | Type                | Null | Key | Default | Extra |
|----------|---------------------|------|-----|---------|-------|
| INDEX_ID | bigint(21) unsigned | NO   |     | 0       |       |
| NAME     | varchar(193)        | NO   |     |         |       |
| TABLE_ID | bigint(21) unsigned | NO   |     | 0       |       |
| TYPE     | int(11)             | NO   |     | 0       |       |
| N_FIELDS | int(11)             | NO   |     | 0       |       |
| PAGE_NO  | int(11)             | NO   |     | 0       |       |
| SPACE    | int(11)             | NO   |     | 0       |       |


#### INNODB_SYS_TABLES

| Field         | Type                | Null | Key | Default | Extra |
|---------------|---------------------|------|-----|---------|-------|
| TABLE_ID      | bigint(21) unsigned | NO   |     | 0       |       |
| NAME          | varchar(655)        | NO   |     |         |       |
| FLAG          | int(11)             | NO   |     | 0       |       |
| N_COLS        | int(11)             | NO   |     | 0       |       |
| SPACE         | int(11)             | NO   |     | 0       |       |
| FILE_FORMAT   | varchar(10)         | YES  |     | NULL    |       |
| ROW_FORMAT    | varchar(12)         | YES  |     | NULL    |       |
| ZIP_PAGE_SIZE | int(11) unsigned    | NO   |     | 0       |       |


#### INNODB_SYS_TABLESPACES

| Field         | Type             | Null | Key | Default | Extra |
|---------------|------------------|------|-----|---------|-------|
| SPACE         | int(11) unsigned | NO   |     | 0       |       |
| NAME          | varchar(655)     | NO   |     |         |       |
| FLAG          | int(11) unsigned | NO   |     | 0       |       |
| FILE_FORMAT   | varchar(10)      | YES  |     | NULL    |       |
| ROW_FORMAT    | varchar(22)      | YES  |     | NULL    |       |
| PAGE_SIZE     | int(11) unsigned | NO   |     | 0       |       |
| ZIP_PAGE_SIZE | int(11) unsigned | NO   |     | 0       |       |


#### INNODB_SYS_TABLESTATS

| Field             | Type                | Null | Key | Default | Extra |
|-------------------|---------------------|------|-----|---------|-------|
| TABLE_ID          | bigint(21) unsigned | NO   |     | 0       |       |
| NAME              | varchar(193)        | NO   |     |         |       |
| STATS_INITIALIZED | varchar(193)        | NO   |     |         |       |
| NUM_ROWS          | bigint(21) unsigned | NO   |     | 0       |       |
| CLUST_INDEX_SIZE  | bigint(21) unsigned | NO   |     | 0       |       |
| OTHER_INDEX_SIZE  | bigint(21) unsigned | NO   |     | 0       |       |
| MODIFIED_COUNTER  | bigint(21) unsigned | NO   |     | 0       |       |
| AUTOINC           | bigint(21) unsigned | NO   |     | 0       |       |
| REF_COUNT         | int(11)             | NO   |     | 0       |       |


#### INNODB_TRX

| Field                      | Type                | Null | Key | Default             | Extra |
|----------------------------|---------------------|------|-----|---------------------|-------|
| trx_id                     | varchar(18)         | NO   |     |                     |       |
| trx_state                  | varchar(13)         | NO   |     |                     |       |
| trx_started                | datetime            | NO   |     | 0000-00-00 00:00:00 |       |
| trx_requested_lock_id      | varchar(81)         | YES  |     | NULL                |       |
| trx_wait_started           | datetime            | YES  |     | NULL                |       |
| trx_weight                 | bigint(21) unsigned | NO   |     | 0                   |       |
| trx_mysql_thread_id        | bigint(21) unsigned | NO   |     | 0                   |       |
| trx_query                  | varchar(1024)       | YES  |     | NULL                |       |
| trx_operation_state        | varchar(64)         | YES  |     | NULL                |       |
| trx_tables_in_use          | bigint(21) unsigned | NO   |     | 0                   |       |
| trx_tables_locked          | bigint(21) unsigned | NO   |     | 0                   |       |
| trx_lock_structs           | bigint(21) unsigned | NO   |     | 0                   |       |
| trx_lock_memory_bytes      | bigint(21) unsigned | NO   |     | 0                   |       |
| trx_rows_locked            | bigint(21) unsigned | NO   |     | 0                   |       |
| trx_rows_modified          | bigint(21) unsigned | NO   |     | 0                   |       |
| trx_concurrency_tickets    | bigint(21) unsigned | NO   |     | 0                   |       |
| trx_isolation_level        | varchar(16)         | NO   |     |                     |       |
| trx_unique_checks          | int(1)              | NO   |     | 0                   |       |
| trx_foreign_key_checks     | int(1)              | NO   |     | 0                   |       |
| trx_last_foreign_key_error | varchar(256)        | YES  |     | NULL                |       |
| trx_adaptive_hash_latched  | int(1)              | NO   |     | 0                   |       |
| trx_adaptive_hash_timeout  | bigint(21) unsigned | NO   |     | 0                   |       |
| trx_is_read_only           | int(1)              | NO   |     | 0                   |       |
| trx_autocommit_non_locking | int(1)              | NO   |     | 0                   |       |


