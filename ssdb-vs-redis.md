
# Redis 迁移到 SSDB 注意事项

SSDB 是一个与 Redis 高度类型的 key value 存储系统，SSDB 相比 Redis 最大不同在于  使用 LevelDB 存储数据到硬盘，内存中只保留部分数据，相比 Redis 大幅减少内容消耗，同时对数据提供更好的保护。

薄荷的伙伴 Status 项目存储大量 Redis 滥用情况，Redis Cluster 最高峰时 消耗内存接近 200 GB。这两周对 Redis 的存储进行大幅重构，把大部分 Redis 存储迁移到 SSDB 上，全部迁移之后，SSDB 占用内存将最大限制在 8 GB，而 Redis 消耗内存只有不到 500 M，成效非常显著。

在 SSDB 迁移到 Redis 过程中，有几个注意事项：

1. SSDB 并不支持所有的 Redis 数据类型，SSDB 只提供了四种类型：KV 类型，Hash 类型，List 类型 和 SortedSet 类型，没有单独的 Set 类型。如果遇到要迁移 Redis 的 Set 类型，可以替换为 Hash 或者 SortedSet。Status 项目中 Redis 存储的 followings 和 followers 都是 Set 类型，迁移到 SSDB 后，全部转换为 SortedSet 。

2. SSDB 有 Redis 高度相似，命令接口高度兼容，所以连接 SSDB 可以直接使用 Redis 的 driver。SSDB 绝大部分命令与 Redis 兼容，不过也有一些命令 SSDB 中不存在或者处理过程不同，在使用过程中要特别注意。下面是我在使用过程中遇到不一致的命令：

2.1 DEL 命令
Redis 的 del 命令可以删除任何数据对象，但是在 SSDB 中只能删除 KV 类型对象，其它类型对象用 DEL 不会报错，但是不会删除效果。如果要删除非 KV 类型对象该怎么办？
只能用每个类型对象特用的 clear 方法，如 Hash 的是 hclear, SortedSet 是 zclear, List 是 qclear。

2.2 KEYS 命令
SSDB 的 keys 命令与 Redis 很不一样，SSDB 中不同数据类型的 key 是隔离的。如果想循环遍历 SSDB 中的 key，不同的数据类型有不同的命令，KV 的是 keys, Hash 的是 hkeys, SortedSet 的是 zkeys，List 的是 qkeys。

2.3 SSDB 不支持数据集合排序命令，也不支持集合的交集和并集命令。

除去上面要注意的地方，其它大部分都是兼容的，所以他们共用一套 Driver，只是使用时候稍微注意上述事项。

