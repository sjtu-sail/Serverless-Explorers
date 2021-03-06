# KVDB性能调研

### levelDB性能调研

##### Setup

我们使用具有一百万个条目的数据库。每个条目都有一个16字节的密钥和一个100字节的值。基准所使用的值压缩到其原始大小的一半左右。

```
LevelDB:    version 1.1
Date:       Sun May  1 12:11:26 2011
CPU:        4 x Intel(R) Core(TM)2 Quad CPU    Q6600  @ 2.40GHz
CPUCache:   4096 KB
Keys:       16 bytes each
Values:     100 bytes each (50 bytes after compression)
Entries:    1000000
Raw Size:   110.6 MB (estimated)
File Size:  62.9 MB (estimated)
```

##### 写性能

先使用“fill”按顺序或随机顺序创建一个全新的数据库。每次操作后，“ fillsync”基准都会将数据从操作系统刷新到磁盘。其他写入操作会将数据留在操作系统缓冲区高速缓存中一段时间。“覆盖”基准进行随机写入，以更新数据库中的现有密钥。

```
fillseq      :       1.765 micros/op;   62.7 MB/s
fillsync     :     268.409 micros/op;    0.4 MB/s (10000 ops)
fillrandom   :       2.460 micros/op;   45.0 MB/s
overwrite    :       2.380 micros/op;   46.5 MB/s
```

上面的每个“ op”对应于单个键/值对的写入。也就是说，随机写入基准测试的速度约为每秒40万次写入（因为每个键值对的大小约为116 Bytes）。

每个“ fillsync”操作的成本（0.3毫秒）比磁盘搜索（通常为10毫秒）少得多。我们怀疑这是因为硬盘本身正在将更新缓冲在其内存中，并且在将数据写入磁盘之前做出了响应。基于硬盘在断电时是否有足够的电量来保存其内存，这可能是安全的，也可能是不安全的。

##### 读性能

下面列出了正向和反向顺序读取的性能，以及随机查找的性能。请注意，基准测试创建的数据库很小。因此，当工作集适合内存时，该报告将表征leveldb的性能。读取操作系统缓冲区高速缓存中不存在的数据的成本将由从磁盘获取数据所需的一个或两个磁盘搜索决定。写入性能几乎不受工作集是否适合内存的影响。

```
readrandom  : 16.677 micros/op;  (approximately 60,000 reads per second)
readseq     :  0.476 micros/op;  232.3 MB/s
readreverse :  0.724 micros/op;  152.9 MB/s
```

LevelDB在后台压缩其基础存储数据以提高读取性能。经过大量随机写入后，立即完成了上面列出的结果。压缩后的结果（通常会自动触发）更好。

```
readrandom  : 11.602 micros/op;  (approximately 85,000 reads per second)
readseq     :  0.423 micros/op;  261.8 MB/s
readreverse :  0.663 micros/op;  166.9 MB/s
```

读取的高昂成本中有一些来自对磁盘读取的块的反复解压缩。如果我们为leveldb提供足够的缓存，以便它可以将未压缩的块保存在内存中，则读取性能将再次提高：

```
readrandom  : 9.775 micros/op;  (approximately 100,000 reads per second before compaction)
readrandom  : 5.215 micros/op;  (approximately 190,000 reads per second after compaction)
```

### rocksdb性能调研

##### Setup

```
rocksDB:     
CPU:        m5d.2xlarge 8
memory:     32 GB
disk:       1 x 300 NVMe SSD.
Kernel version: Linux 4.14.177-139.253.amzn2.x86_64
File System:XFS with discard enabled
```

* 测试一 bulkload

  ```
  NUM_KEYS = 900000000 
  NUM_THREADS = 32 
  CACHE_SIZE = 6442450944
  ```

  测试将9亿个键值对加载到数据库中。键值对以随机顺序插入。在此测试开始时，数据库为空，然后逐渐填满。在进行数据加载时，没有数据正在读取。

* 测试二 overwrite

  ```
  NUM_KEYS = 900000000 
  NUM_THREADS = 32 
  CACHE_SIZE = 6442450944 
  DURATION = 5400
  ```

  测试将键值对随机覆盖到数据库中。该数据库是由先前的测试首先创建的。

* 测试三 

  ```
  NUM_KEYS = 900000000 
  NUM_THREADS = 32 
  CACHE_SIZE = 6442450944 
  DURATION = 5400
  ```

  测试将随机读取键值对以及对现有键值对进行持续更新。该测试的数据库是测试二最后的结果。

* 测试四

  ```
  NUM_KEYS = 900000000 
  NUM_THREADS = 32 
  CACHE_SIZE = 6442450944 
  DURATION = 5400 
  ```

  衡量数据库的随机读取性能。

  下面显示了使用各种发行版和参数进行这些测试的结果。

##### 性能测试

* **方案一**  block size与性能关系的测试

  ```
  8K: Complete bulkload in 4560 seconds 
  4K: Complete bulkload in 5215 seconds 
  16K: Complete bulkload in 3996 seconds 
  DIO: Complete bulkload in 4547 seconds 
  8K RL: Complete bulkload in 4388 seconds
  ```

  其中DIO的block size 大小也为8k

  其中RL表示在测试之间进行定时限速操作。比如测试一和测试二之间进行限时30分钟，限速2MB/s的缓冲器，确保所有的flush操作和后台操作都能完成。

  * 测试一 bulkload

    | BLOCK | OPS/SEC | MB/SEC | SIZE-GB | L0_GB | SUM_GB | W-AMP | W-MB/S | USEC/OP | P50  | P75  | P99  | P99.9 | P99.99 | UPTIME | STALL-TIME   | STALL% | DU -S - K |
    | :---- | :------ | :----- | :------ | :---- | :----- | :---- | :----- | :------ | :--- | :--- | :--- | :---- | :----- | :----- | :----------- | :----- | :-------- |
    | 8K    | 924468  | 370.3  | 0.2     | 157.1 | 157.1  | 1.0   | 167.5  | 1.1     | 0.5  | 0.8  | 2    | 4     | 1119   | 960    | 00:03:45.193 | 23.5   | 101411592 |
    | 4K    | 853217  | 341.8  | 0.2     | 165.3 | 165.3  | 1.0   | 165.9  | 1.2     | 0.5  | 0.8  | 2    | 4     | 1159   | 1020   | 00:04:41.465 | 27.6   | 108748512 |
    | 16K   | 1027567 | 411.6  | 0.1     | 149.0 | 149.0  | 1.0   | 181.6  | 1.0     | 0.5  | 0.8  | 2    | 3     | 1021   | 840    | 00:02:23.600 | 17.1   | 99070240  |
    | DIO   | 921342  | 369.0  | 0.2     | 156.6 | 156.6  | 1.0   | 167.0  | 1.1     | 0.5  | 0.8  | 2    | 4     | 1104   | 960    | 00:03:27.280 | 21.6   | 101412440 |
    | 8K RL | 989786  | 396.5  | 0.2     | 159.4 | 159.4  | 1.0   | 179.5  | 1.0     | 0.5  | 0.8  | 2    | 4     | 1043   | 909    | 00:02:41.514 | 17.8   | 101406496 |

  * 测试二 overwrite

    | BLOCK | OPS/SEC | MB/SEC | SIZE-GB | L0_GB | SUM_GB | W-AMP | W-MB/S | USEC/OP | P50   | P75   | P99   | P99.9 | P99.99 | STALL-TIME   | STALL%       | DU -S -K  |
    | :---- | :------ | :----- | :------ | :---- | :----- | :---- | :----- | :------ | :---- | :---- | :---- | :---- | :----- | :----------- | :----------- | :-------- |
    | 8K    | 85756   | 34.3   | 0.1     | 161.4 | 739.9  | 4.5   | 142.2  | 373.1   | 9.7   | 274.1 | 5613  | 25620 | 47726  | 00:20:18.388 | 22.9         | 159903832 |
    | 4K    | 79856   | 32.0   | 0.2     | 166.0 | 716.9  | 4.3   | 136.3  | 400.7   | 9.7   | 268.9 | 5914  | 25394 | 47296  | 00:25:37.183 | 28.5         | 168094916 |
    | 16K   | 93678   | 37.5   | 0.1     | 174.4 | 825.0  | 4.7   | 156.8  | 341.6   | 9.4   | 279.2 | 4453  | 24796 | 47038  | 00:16:24.878 | 18.3         | 155953232 |
    | DIO   | 85655   | 34.3   | 0.1     | 163.9 | 734.9  | 4.4   | 140.7  | 373.6   | 9.7   | 263.1 | 6250  | 25807 | 47678  | 00:18:51.145 | 21.2         | 159470752 |
    | 8K RL | 85542   | 34.3   | 0.1     | 161.2 | 757.8  | 4.7   | 143.6  | 748.1   | 340.5 | 735.8 | 11852 | 30851 | 59137  | 5401         | 00:08:18.359 | 9.2       |

  * 测试三 read while writing

    | BLOCK | OPS/SEC | MB/SEC | SIZE-GB | L0_GB | SUM_GB | W-AMP | W-MB/S | USEC/OP | P50   | P75   | P99  | P99.9 | P99.99 | STALL-TIME   | STALL%      | DU -S -K  |
    | :---- | :------ | :----- | :------ | :---- | :----- | :---- | :----- | :------ | :---- | :---- | :--- | :---- | :----- | :----------- | :---------- | :-------- |
    | 8K    | 89285   | 28.0   | 0.1     | 4.2   | 199.6  | 47.5  | 37.9   | 358.4   | 281.1 | 427.9 | 2935 | 7587  | 19029  | 00:13:7.325  | 14.6        | 139287936 |
    | 4K    | 116759  | 36.2   | 0.1     | 3.6   | 203.8  | 56.6  | 38.9   | 274.1   | 224.4 | 328.0 | 2534 | 6131  | 13678  | 00:20:58.789 | 23.5        | 147504716 |
    | 16K   | 64393   | 20.4   | 0.1     | 4.1   | 194.0  | 47.3  | 36.8   | 496.9   | 402.3 | 642.7 | 3488 | 7251  | 8880   | 00:10:58.906 | 12.2        | 138132068 |
    | DIO   | 98698   | 30.9   | 0.1     | 3.9   | 197.4  | 50.6  | 37.6   | 324.2   | 257.7 | 353.7 | 2764 | 6583  | 13742  | 00:16:47.979 | 18.8        | 139319040 |
    | 8K RL | 101598  | 31.9   | 0.1     | 3.2   | 97.2   | 30.3  | 18.4   | 629.9   | 587.5 | 805.9 | 3922 | 6881  | 19699  | 5402         | 00:00:0.054 | 0.0       |

  * 测试四 randomread

    | BLOCK | OPS/SEC | MB/SEC | SIZE-GB | L0_GB | SUM_GB | W-AMP | W-MB/S | USEC/OP | P50   | P75   | P99  | P99.9 | P99.99 | DU =S -K  |
    | :---- | :------ | :----- | :------ | :---- | :----- | :---- | :----- | :------ | :---- | :---- | :--- | :---- | :----- | :-------- |
    | 8K    | 101647  | 32.0   | 0.1     | 0.0   | 3.9    | 0     | .7     | 314.8   | 410.7 | 498.8 | 761  | 1247  | 3092   | 139119060 |
    | 4K    | 130846  | 40.7   | 0.1     | 0.0   | 1.0    | 0     | .1     | 244.6   | 291.7 | 347.5 | 663  | 865   | 2626   | 147417776 |
    | 16K   | 70884   | 22.6   | 0.1     | 0.0   | 1.3    | 0     | .2     | 451.4   | 547.5 | 715.0 | 1039 | 1397  | 2598   | 138040824 |
    | DIO   | 144737  | 45.5   | 0.1     | 0.1   | 0.7    | 7.0   | .1     | 221.1   | 239.8 | 320.9 | 578  | 866   | 2133   | 139239620 |
    | 8K RL | 105790  | 33.4   | 0.1     | 0.0   | 0.0    | 0     | 605.0  | 683.0   | 807.9 | 1579  | 3133 | 6152  | 5403   | 139681920 |

​       

* **方案二** 测试默认配置下的性能，参数为RocksDB 6.10, 2K Value size, 100M Keys.

  | TEST             | OPS/SEC | MB/SEC | SIZE-GB | L0_GB | SUM_GB | W-AMP | W-MB/S | USEC/OP | P50   | P75   | P99  | P99.9 | P99.99 | UPTIME | STALL-TIME   | STALL% | DU -S -K  |
  | :--------------- | :------ | :----- | :------ | :---- | :----- | :---- | :----- | :------ | :---- | :---- | :--- | :---- | :----- | :----- | :----------- | :----- | :-------- |
  | bulkload         | 272448  | 537.3  | 0.1     | 85.3  | 85.3   | 1.0   | 242.6  | 3.7     | 0.7   | 1.1   | 3    | 1105  | 1285   | 360    | 00:03:52.679 | 64.6   | 57285876  |
  | overwrite        | 22940   | 45.2   | 0.1     | 229.3 | 879.4  | 3.8   | 169.0  | 1394.9  | 212.9 | 350.7 | 7603 | 26151 | 160352 | 5328   | 01:06:21.977 | 74.7   | 110458852 |
  | readwhilewriting | 87093   | 154.2  | 0.1     | 5.4   | 162.6  | 30.1  | 31.0   | 367.4   | 369.2 | 491.9 | 2209 | 6302  | 13544  | 5360   | 00:00:1.160  | 0.0    | 92081776  |
  | readrandom       | 95666   | 169.9  | 0.1     | 0.0   | 0.0    | 0     | 0      | 334.5   | 411.1 | 498.7 | 742  | 1214  | 2789   | 5358   | 00:00:0.000  | 0.0    | 92092164  |

### TIDB性能调研

##### 测试方案

1. 通过 TiUP 部署 TiDB v4.0 和 v3.0。
2. 通过 Sysbench 导入 16 张表，每张表有 1000 万行数据。
3. 分别对每个表执行 `analyze table` 命令。
4. 备份数据，用于不同并发测试前进行数据恢复，以保证每次数据一致。
5. 启动 Sysbench 客户端，进行 `point_select`、`read_write`、`update_index` 和 `update_non_index` 测试。通过 AWS NLB 向 TiDB 加压，单轮预热 1 分钟，测试 5 分钟。
6. 每轮完成后停止集群，使用之前的备份的数据覆盖，再启动集群。

##### 测试结果

* Point Select 性能

| Threads | v3.0 QPS    | v3.0 95% latency (ms) | v4.0 QPS    | v4.0 95% latency (ms) | QPS 提升 |
| :------ | :---------- | :-------------------- | :---------- | :-------------------- | :------- |
| 150     | 117085.701  | 1.667                 | 118165.1357 | 1.608                 | 0.92%    |
| 300     | 200621.4471 | 2.615                 | 207774.0859 | 2.032                 | 3.57%    |
| 600     | 283928.9323 | 4.569                 | 320673.342  | 3.304                 | 12.94%   |
| 900     | 343218.2624 | 6.686                 | 383913.3855 | 4.652                 | 11.86%   |
| 1200    | 347200.2366 | 8.092                 | 408929.4372 | 6.318                 | 17.78%   |
| 1500    | 366406.2767 | 10.562                | 418268.8856 | 7.985                 | 14.15%   |

* Update Non-index 性能

| Threads | v3.0 QPS    | v3.0 95% latency (ms) | v4.0 QPS    | v4.0 95% latency (ms) | QPS 提升 |
| :------ | :---------- | :-------------------- | :---------- | :-------------------- | :------- |
| 150     | 15446.41024 | 11.446                | 16954.39971 | 10.844                | 9.76%    |
| 300     | 22276.15572 | 17.319                | 24364.44689 | 16.706                | 9.37%    |
| 600     | 28784.88353 | 29.194                | 31635.70833 | 28.162                | 9.90%    |
| 900     | 32194.15548 | 42.611                | 35787.66078 | 38.942                | 11.16%   |
| 1200    | 33954.69114 | 58.923                | 38552.63158 | 51.018                | 13.54%   |
| 1500    | 35412.0032  | 74.464                | 40859.63755 | 62.193                | 15.38%   |

* Update Index 性能

| Threads | v3.0 QPS    | v3.0 95% latency (ms) | v4.0 QPS    | v4.0 95% latency (ms) | QPS 提升 |
| :------ | :---------- | :-------------------- | :---------- | :-------------------- | :------- |
| 150     | 11164.40571 | 16.706                | 11954.73635 | 16.408                | 7.08%    |
| 300     | 14460.98057 | 28.162                | 15243.40899 | 28.162                | 5.41%    |
| 600     | 17112.73036 | 53.85                 | 18535.07515 | 50.107                | 8.31%    |
| 900     | 18233.83426 | 86.002                | 20339.6901  | 70.548                | 11.55%   |
| 1200    | 18622.50283 | 127.805               | 21390.25122 | 94.104                | 14.86%   |
| 1500    | 18980.34447 | 170.479               | 22359.996   | 114.717               | 17.81%   |

* Read Write 性能

| Threads | v3.0 QPS    | v3.0 95% latency (ms) | v4.0 QPS    | v4.0 95% latency (ms) | QPS 提升 |
| :------ | :---------- | :-------------------- | :---------- | :-------------------- | :------- |
| 150     | 43768.33633 | 71.83                 | 53912.63705 | 59.993                | 23.18%   |
| 300     | 55655.63589 | 121.085               | 71327.21336 | 97.555                | 28.16%   |
| 600     | 64642.96992 | 223.344               | 84487.75483 | 176.731               | 30.70%   |
| 900     | 68947.25293 | 325.984               | 90177.94612 | 257.95                | 30.79%   |
| 1200    | 71334.80099 | 434.829               | 92779.71507 | 344.078               | 30.06%   |
| 1500    | 72069.9115  | 580.017               | 95088.50812 | 434.829               | 31.94%   |

### 调研结果

RocksDB支持一次获取多个K-V，还支持Key范围查找。LevelDB只能获取单个Key

RocksDB除了简单的Put、Delete操作，还提供了一个Merge操作，说是为了对多个Put操作进行合并。站在引擎实现者的角度来看，相比其带来的价值，其实现的成本要昂贵很多。但有时过于追求完美不见得是好事，性能的瓶颈其实主要在合并上，多一次少一次Put对性能的影响并无大碍。

RocksDB提供一些方便的工具，这些工具包含解析sst文件中的K-V记录、解析MANIFEST文件的内容等。有了这些工具，就不用再像使用LevelDB那样，只能在程序中才能知道sst文件K-V的具体信息了。

RocksDB支持多线程合并，而LevelDB是单线程合并的。LSM型的数据结构，最大的性能问题就出现在其合并的时间损耗上，在多CPU的环境下，多线程合并那是LevelDB所无法比拟的。不过据其官网上的介绍，似乎多线程合并还只是针对那些与下一层没有Key重叠的文件，只是简单的rename而已，至于在真正数据上的合并方面是否也有用到多线程，就只能看代码了。

RocksDB增加了合并时过滤器，对一些不再符合条件的K-V进行丢弃，如根据K-V的有效期进行过滤。

压缩方面RocksDB可采用多种压缩算法，除了LevelDB用的snappy，还有zlib、bzip2。LevelDB里面按数据的压缩率（压缩后低于75%）判断是否对数据进行压缩存储，而RocksDB典型的做法是Level 0-2不压缩，最后一层使用zlib，而其它各层采用snappy。

在故障方面，RocksDB支持增量备份和全量备份，允许将已删除的数据备份到指定的目录，供后续恢复。

RocksDB支持在单个进程中启用多个实例，而LevelDB只允许单个实例。

那么TiDB呢？TiDB相当于rocksDB进阶版，它在本地实现rocksDB存储，然后使用raft协议实现分布式存储。这样的方式保证了在超大数据集时能够保证数据存储的安全性和持久性。

此次调研有很多不足，首先，调研时未能将所有的datebase部署进行测试，而仅仅是调研了官方的测试文档和测试结果。这样的原因有两点：对于大多数的database来说需要一个统一的运行环境，这就需要部署到集群上进行测试，但是我并未有使用集群的权限。第二，每次测试需要编写不同的benchmark脚本，进行全部测试benchmark脚本编写近百个，工作量巨大，实在无法在有限时间内完成。

