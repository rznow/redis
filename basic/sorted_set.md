你列出的是 **Redis Sorted Set（有序集合）** 的完整命令集。Sorted Set 是 Redis 中最强大的数据结构之一，每个元素关联一个 **score（分数）**，元素按分数排序，同时支持集合运算和范围查询。

---

# Redis Sorted Set 常用命令总结

Redis 的 **Sorted Set（有序集合）** 是一个**有序、不重复**的字符串集合，每个元素关联一个 `score`（双精度浮点数），元素按分数从小到大排序。底层使用**跳表（Skip List）+ 哈希表**实现，支持高效的**范围查询、排名计算、集合运算**，非常适合实现：

- 排行榜（游戏积分、销量榜）
- 延迟队列（按时间排序）
- 优先级队列
- 滑动窗口限流
- 社交关系（关注时间线）
- 自动补全（按前缀）

---

# 一、添加与更新元素

## ZADD

向有序集合添加一个或多个成员，或更新已存在成员的 score。

```redis
ZADD leaderboard 100 "Alice" 95 "Bob" 88 "Charlie"
```

**可选参数**：

| 参数 | 说明 |
|------|------|
| `NX` | 只添加新元素，不更新已存在的 |
| `XX` | 只更新已存在的元素，不添加新元素 |
| `GT` | 仅当新 score > 当前 score 时才更新 |
| `LT` | 仅当新 score < 当前 score 时才更新 |
| `CH` | 返回被修改的元素数量（包括新增和更新） |
| `INCR` | 对指定成员的 score 增加增量（相当于 `ZINCRBY`） |

---

## ZINCRBY

对指定成员的 score 增加增量。

```redis
ZINCRBY leaderboard 5 "Alice"
```

---

# 二、查看与统计元素

## ZCARD

返回有序集合的基数（元素个数）。

```redis
ZCARD leaderboard
```

---

## ZCOUNT

返回 score 在指定区间内的元素数量。

```redis
ZCOUNT leaderboard 80 100
```

---

## ZSCORE

返回指定成员的 score。

```redis
ZSCORE leaderboard "Alice"
```

---

## ZMSCORE

批量返回多个成员的 score（Redis 6.2+）。

```redis
ZMSCORE leaderboard "Alice" "Bob" "Charlie"
```

---

## ZRANK

返回指定成员的排名（从 0 开始，按 score 从小到大）。

```redis
ZRANK leaderboard "Bob"
```

---

## ZREVRANK

返回指定成员的排名（从 0 开始，按 score 从大到小）。

```redis
ZREVRANK leaderboard "Bob"
```

---

## ZRANDMEMBER

随机返回一个或多个成员（Redis 6.2+）。

```redis
ZRANDMEMBER leaderboard 2 WITHSCORES
```

---

# 三、范围查询

## ZRANGE

按排名范围获取成员（从小到大）。

```redis
ZRANGE leaderboard 0 10 WITHSCORES
```

**可选参数**：

| 参数 | 说明 |
|------|------|
| `BYSCORE` | 按 score 范围查询 |
| `BYLEX` | 按字典序范围查询 |
| `REV` | 倒序 |
| `LIMIT offset count` | 分页 |
| `WITHSCORES` | 同时返回 score |

---

## ZREVRANGE

按排名范围获取成员（从大到小）。

```redis
ZREVRANGE leaderboard 0 10 WITHSCORES
```

---

## ZRANGEBYSCORE

按 score 范围获取成员（从小到大）。

```redis
ZRANGEBYSCORE leaderboard 80 100 WITHSCORES LIMIT 0 10
```

特殊值：
- `-inf`：负无穷
- `+inf`：正无穷
- `(80`：不包含 80

---

## ZREVRANGEBYSCORE

按 score 范围获取成员（从大到小）。

```redis
ZREVRANGEBYSCORE leaderboard 100 80 WITHSCORES
```

---

## ZRANGEBYLEX

按字典序范围获取成员（所有元素 score 必须相同）。

```redis
ZRANGEBYLEX myzset [a [c
```

前缀匹配：
- `[`：包含
- `(`：不包含
- `-`：负无穷
- `+`：正无穷

---

## ZREVRANGEBYLEX

按字典序范围获取成员（倒序）。

---

## ZLEXCOUNT

统计字典序区间内的元素数量。

```redis
ZLEXCOUNT myzset [a [c
```

---

# 四、排名与分数更新

## ZPOPMAX

移除并返回 score 最高的一个或多个成员。

```redis
ZPOPMAX leaderboard 3
```

---

## ZPOPMIN

移除并返回 score 最低的一个或多个成员。

```redis
ZPOPMIN leaderboard 3
```

---

## BZPOPMAX

阻塞版 ZPOPMAX。

```redis
BZPOPMAX leaderboard 30
```

---

## BZPOPMIN

阻塞版 ZPOPMIN。

```redis
BZPOPMIN leaderboard 30
```

---

## BZMPOP

阻塞版 ZMPOP（Redis 7.0+）。

```redis
BZMPOP 30 2 leaderboard queue MAX COUNT 5
```

---

# 五、删除元素

## ZREM

移除一个或多个成员。

```redis
ZREM leaderboard "Alice" "Bob"
```

---

## ZREMRANGEBYRANK

按排名范围移除元素。

```redis
ZREMRANGEBYRANK leaderboard 0 10
```

---

## ZREMRANGEBYSCORE

按 score 范围移除元素。

```redis
ZREMRANGEBYSCORE leaderboard 0 50
```

---

## ZREMRANGEBYLEX

按字典序范围移除元素（所有元素 score 必须相同）。

```redis
ZREMRANGEBYLEX myzset [a [c
```

---

# 六、集合间运算

## ZINTER

返回多个有序集合的**交集**（Redis 6.2+）。

```redis
ZINTER 2 set1 set2 WITHSCORES
```

---

## ZINTERSTORE

将交集结果存储到目标集合。

```redis
ZINTERSTORE destination 2 set1 set2 WEIGHTS 2 3 AGGREGATE SUM
```

---

## ZINTERCARD

返回交集的基数，可设置提前终止（Redis 7.0+）。

```redis
ZINTERCARD 2 set1 set2 LIMIT 100
```

---

## ZUNION

返回多个有序集合的**并集**（Redis 6.2+）。

```redis
ZUNION 2 set1 set2 WITHSCORES
```

---

## ZUNIONSTORE

将并集结果存储到目标集合。

```redis
ZUNIONSTORE destination 2 set1 set2 WEIGHTS 2 3 AGGREGATE SUM
```

---

## ZDIFF

返回多个有序集合的**差集**（Redis 6.2+）。

```redis
ZDIFF 2 set1 set2 WITHSCORES
```

---

## ZDIFFSTORE

将差集结果存储到目标集合。

```redis
ZDIFFSTORE destination 2 set1 set2
```

---

# 七、移动与弹出

## ZMPOP

从多个有序集合中弹出 score 最高/最低的元素（Redis 7.0+）。

```redis
ZMPOP 2 leaderboard queue MAX COUNT 5
```

---

## ZRANGESTORE

将范围查询结果存储到目标集合（Redis 6.2+）。

```redis
ZRANGESTORE dst src 0 10 BYSCORE WITHSCORES
```

---

# 八、扫描

## ZSCAN

游标迭代有序集合中的元素。

```redis
ZSCAN leaderboard 0 MATCH A* COUNT 10
```

---

# 九、命令总结

| 命令 | 功能 | 是否阻塞 | 时间复杂度 |
|------|------|---------|-----------|
| `ZADD` | 添加/更新元素 | ❌ | O(log N) 每个 |
| `ZINCRBY` | 增加分数 | ❌ | O(log N) |
| `ZCARD` | 获取元素个数 | ❌ | O(1) |
| `ZCOUNT` | 统计分数区间数量 | ❌ | O(log N) |
| `ZSCORE` | 获取分数 | ❌ | O(1) |
| `ZMSCORE` | 批量获取分数 | ❌ | O(N) |
| `ZRANK` | 获取排名（升序） | ❌ | O(log N) |
| `ZREVRANK` | 获取排名（降序） | ❌ | O(log N) |
| `ZRANGE` | 范围查询（排名） | ❌ | O(log N + M) |
| `ZREVRANGE` | 范围查询（排名，倒序） | ❌ | O(log N + M) |
| `ZRANGEBYSCORE` | 范围查询（分数） | ❌ | O(log N + M) |
| `ZREVRANGEBYSCORE` | 范围查询（分数，倒序） | ❌ | O(log N + M) |
| `ZRANGEBYLEX` | 范围查询（字典序） | ❌ | O(log N + M) |
| `ZPOPMAX` | 弹出最高分元素 | ❌ | O(log N) |
| `ZPOPMIN` | 弹出最低分元素 | ❌ | O(log N) |
| `BZPOPMAX` | 阻塞弹出最高分 | ✅ | O(log N) |
| `BZPOPMIN` | 阻塞弹出最低分 | ✅ | O(log N) |
| `BZMPOP` | 阻塞多集合弹出 | ✅ | O(log N) |
| `ZREM` | 删除元素 | ❌ | O(log N) 每个 |
| `ZREMRANGEBYRANK` | 按排名删除 | ❌ | O(log N + M) |
| `ZREMRANGEBYSCORE` | 按分数删除 | ❌ | O(log N + M) |
| `ZREMRANGEBYLEX` | 按字典序删除 | ❌ | O(log N + M) |
| `ZINTER` | 交集 | ❌ | O(N*K) |
| `ZINTERSTORE` | 交集存目标 | ❌ | O(N*K) |
| `ZINTERCARD` | 交集基数 | ❌ | O(N*K) |
| `ZUNION` | 并集 | ❌ | O(N) |
| `ZUNIONSTORE` | 并集存目标 | ❌ | O(N) |
| `ZDIFF` | 差集 | ❌ | O(N) |
| `ZDIFFSTORE` | 差集存目标 | ❌ | O(N) |
| `ZSCAN` | 游标迭代 | ❌ | O(1) 每步 |

---

# 十、使用场景示例

## 场景 1：游戏排行榜

```redis
# 更新玩家分数
ZADD leaderboard 1500 "player1" 2300 "player2" 1800 "player3"

# 获取前三名
ZREVRANGE leaderboard 0 2 WITHSCORES

# 获取玩家排名
ZREVRANK leaderboard "player1"

# 获取分数区间玩家
ZRANGEBYSCORE leaderboard 2000 3000 WITHSCORES
```

---

## 场景 2：延迟队列（按时间排序）

```redis
# 添加任务（score = 执行时间戳）
ZADD delay_queue 1700000000 "task1" 1700000100 "task2"

# 取出到期的任务
ZRANGEBYSCORE delay_queue 0 1700000050

# 执行后删除
ZREM delay_queue "task1"
```

---

## 场景 3：滑动窗口限流

```redis
# 用户行为记录（score = 时间戳）
ZADD rate_limit:user1 1700000000 "req1"

# 统计最近60秒请求数
ZCOUNT rate_limit:user1 (1700000000-60) 1700000000
```

---

## 场景 4：自动补全（字典序）

```redis
# 所有元素 score 相同
ZADD autocomplete 0 "apple" 0 "application" 0 "apply"

# 查询以 "appl" 开头的词
ZRANGEBYLEX autocomplete [appl [appm
```

---

## 场景 5：关注时间线

```redis
# 每个用户的帖子按时间排序
ZADD timeline:user1 1700000000 "post1" 1700000100 "post2"

# 获取最近10条
ZREVRANGEBYSCORE timeline:user1 +inf -inf LIMIT 0 10
```

---

# 十一、最佳实践

1. **合理使用 `WITHSCORES`**：只在需要分数时返回，减少网络传输。
2. **大集合分页**：使用 `LIMIT offset count` 避免一次性返回大量数据。
3. **阻塞命令慎用**：`BZPOPMAX`/`BZPOPMIN` 会阻塞 Redis，确保设置合理的 timeout。
4. **集合运算优化**：多个集合的交集/并集运算时，将**最小的集合放在前面**可提高性能。
5. **字典序查询要求所有元素 score 相同**：否则结果不可预测。
6. **ZSET 元素推荐 < 1 万**：虽然 ZSET 支持百万级元素，但大规模集合运算可能较慢。

---

# 十二、总结

Redis Sorted Set 是处理**排序、排名、范围查询、延迟队列**的最佳数据结构：

- ✔ 游戏排行榜
- ✔ 积分/销量排行
- ✔ 延迟队列（定时任务）
- ✔ 优先级队列
- ✔ 滑动窗口限流
- ✔ 自动补全
- ✔ 时间线（Timeline）
- ✔ 社交关系（关注列表按时间排序）
- ✔ 实时统计（Top N 查询）