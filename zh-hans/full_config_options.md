## 完整配置项


### 1.说明
下表中所有的环境变量均需要在使用时添加前缀 `TerarkZipTable_`，如对于 `localTempDir` 变量，您需要设置的环境变量是`TerarkZipTable_localTempDir`.

除了 TerarkDB 自身的参数(TerarkZipTableOptions 的参数)，其它属于 RocksDB 自身的参数，如果在环境变量中指定，就会覆盖应用程序指定的参数。例如，如果应用程序(例如 MyRocks)指定了 `write_buffer_size=67108864`(`64M`)，同时这里的环境变量也指定了 `TerarkZipTable_write_buffer_size=2G`，那么应用程序指定的 `64M` 就会被覆盖。

所以，如果(使用RocksDB的)应用程序自身有一套灵活完整的参数配置机制，最好使用它本身的参数配置机制。通过这里的环境变量指定 RocksDB 的参数只是作为一种补充（如果应用程序的参数配置机制不够完善的话）。

### 2.参数表

**1) `TerarkZipTableOptions` 类中的参数**

|type|env var `suffix` or<br/>TerarkZipTableOptions::<br/>`member`|Default|解释说明|
|----|-------|-----------------------|------------|
|string|localTempDir|*|临时文件目录|
|enum<br/>string|entropyAlgo|`NoEntropy`|熵编码对压缩率提高有限，<br/>但对读性能影响巨大|
|string|indexType|`IL_256`|指定 rank-select 的实现方式|
|int|checksumLevel|3|`3` 对整体数据做 checksum|
|int|indexNestLevel|3|越大，压缩率越高，读性能越低|
|int|indexNestScale|8|越大，压缩率越高，读性能越低|
|int|indexTempLevel|0|越大，索引创建的内存用量约小，索引创建的速度越慢，<br/>`-1`: 禁用临时文件<br/>`0`: 智能判断，仅对大索引使用临时文件|
|int|terarkZipMinLevel|0|仅在大于等于该 level 时，才使用 Terark 的 SSTable|
|int|keyPrefixLen|0(禁用)|For MongoRocks, MyRocks ...<br/>一般上层数据库会用一个 `BigEndianUint` 作为下层 KeyValue 引擎中 Key 的前缀，用来区分不同的 table/index/...，相同的 table/index/... 一般具有相同的 schema，从而 KeyValue 引擎可以进行更好的优化|
|int|offsetArrayBlockUnits|0(禁用)|合法值: {0,64,128}|
|bool|disableSecondPassIter|false|false 对临时文件需求小，但压缩速度慢，true: 磁盘空间必须充足|
|bool|useSuffixArrayLocalMatch|false|可以提高压缩率，但会降低压缩速度|
|bool|warmUpIndexOnOpen|true|预热 index|
|bool|warmUpValueOnOpen|false|预热 value data|
|float|estimateCompressionRatio|0.2|压缩率预估，SST Build 过程中，TerarkZipTable 并不会写 SST（Finish 时写才 SST），但 rocksdb 需要 SST Builder 报告 SST 文件尺寸，所以使用该预估值向 rocksdb 报告预估的 SST 文件尺寸，用来控制最终输出的 SST 文件尺寸|
|bool|enableCompressionProbe|true|自动检测数据冗余度（是否可压缩）。<br/>该参数用来更准确地预估 SST 文件的尺寸，并提高压缩速度（当数据不可压缩时，就禁用压缩）|
|float|sampleRatio|0.03|采样比率：对于单个 SST，全局字典的尺寸占 value 总尺寸的比例，全局字典的上限是 2G，如果超过 2G，也只保留 2G。<br/>全局字典需要常驻内存，所以，如果数据的总尺寸比内存大很多，应该将该值适当调小，建议**所有 SST 文件的全局字典的总尺寸不超过系统总内存的 30%**|
|float|indexCacheRatio|0|典型值 0 或者较小的值，如 0.003<br/>在使用嵌套 Trie 树索引时，indexCache 可以提高点查(Point Query)的性能，0.003 大约可以将点查的性能提高 10%|
|int64|softZipWorkingMemLimit|<sup>系统总内存</sup>&frasl;<sub>8</sub>|软限制 compact 的内存用量|
|int64|hardZipWorkingMemLimit|<sup>系统总内存</sup>&frasl;<sub>4</sub>|硬限制 compact 的内存用量|
|int64|smallTaskMemory|<sup>系统总内存</sup>&frasl;<sub>32</sub>|小 compact(或 memtable flush) 的优先级更高|

**2) `ColumnFamilyOptions` 类中的参数**

*请尽量使用应用程序自身的参数配置机制，因为这里的配置拥有最高的优先级，只在必要情况下作为一种补充机制*

|type|env var `suffix` or<br/>ColumnFamilyOptions::`member`|Default|
|----|-------|-----------------------|
|enum by<br/>string|compaction_style|universal|
|int|num_levels|7|
|int64|write_buffer_size|<sup>系统总内存</sup>&frasl;<sub>32</sub>|
|int|max_write_buffer_number|3|
|int64|target_file_size_base|<sup>系统总内存</sup>&frasl;<sub>2</sub>|
|int|target_file_size_multiplier|1|
|int|level0_file_num_compaction_trigger|rocksdb<br/>default|

**3) `DBOptions` 类中的参数**

*请尽量使用应用程序自身的参数配置机制，因为这里的配置拥有最高的优先级，只在必要情况下作为一种补充机制*

|type|env var `suffix` or <br/>DBOptions::`member`|Default|
|----|-------|-----------------------|
|int|base_background_compactions|3|
|int|max_background_compactions|5|
|int|max_background_flushes|3|
|int|max_subcompactions|1|

**3) `ColumnFamilyOptions::compaction_options_universal` 中的参数**

*这些参数虽然是 RocksDB 自身的参数，但是绝大多数应用都不使用 universal compaction，所以这几个参数可以看成是 TerarkDB 的参数。*

|type|env var `suffix` 或者 <br/>ColumnFamilyOptions::compaction_options_universal::`member`|Default|
|----|-------|-----------------------|
|int|min_merge_width|4|
|int|max_merge_width|50|

**5) 其他参数**

|type|env var full name|Default|
|----|-------|-----------------------|
|string|`TerarkZipTable_blackListColumnFamily`|`empty`|
|int|`TerarkDictZipBlobStore_zipThreads`|min(8, physical\_cpu\_num)|

- `TerarkZipTable_blackListColumnFamily`
  - 只要定义了环境变量 TerarkZipTable_localTempDir，就会使用 TerarkZipTable，用户代码中设置的 option 参数就会全部失效
  - 用户可以把一些 `column families` 放在黑名单，他们将不会使用 `TerarkZipTable`
  - 多个 `column families` 需要通过 ',' 分割:<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`TerarkZipTable_blackListColumnFamily=cf1,cf2,cf3`
  - 可以考虑将 `日志数据` 或者短时间存在的数据放到黑名单

- `TerarkDictZipBlobStore_zipThreads`
  - 如果这个变量不是 `0`, TerarkDB 的 SST Builder 会通过 `read -> compress -> write` 这个过程来压缩，这个过程由所有的 SST Builder 共享。这个变量代表上述过程的 `compress` 阶段的线程数.
  - 如果该变量比物理 CPU 数更大，将使用物理 CPU 数作为使用值.
  - 如果该变量是 `0`, TerarkDB 将不会使用 `read -> compress -> write` 这个过程，而是由调用线程来执行压缩.

