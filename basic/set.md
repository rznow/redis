# Redis Set 常用命令总结

Redis 的 **Set（集合）** 是一个**无序、不重复**的字符串集合，底层基于哈希表实现，支持高效的**添加、删除、查找**操作，以及集合间的**交集、并集、差集**运算，非常适合实现：

- 标签系统（Tagging）
- 共同好友/关注
- 去重统计
- 用户权限/角色
- 抽奖/投票去重
- 推荐系统（基于集合运算）

---

# 一、添加与删除元素

## SADD

向集合添加一个或多个成员（已存在则忽略）。

```redis
SADD tags "Redis" "Database" "C++"
```

返回：实际新增的元素数量。

---

## SREM

从集合中移除一个或多个成员。

```redis
SREM tags "Database"
```

返回：实际移除的元素数量。

---

## SMOVE

将元素从一个集合移动到另一个集合（原子操作）。

```redis
SMOVE source destination member
```

如果 `source` 中不存在该成员，则不做任何事。

---

# 二、查看与统计元素

## SMEMBERS

返回集合中所有成员（无序）。

```redis
SMEMBERS tags
```

**注意**：当集合很大时，慎用此命令，会消耗大量带宽。

---

## SISMEMBER

判断某个成员是否在集合中。

```redis
SISMEMBER tags "Redis"
```

返回：
- `1`：存在
- `0`：不存在

---

## SMISMEMBER

批量判断多个成员是否在集合中（Redis 6.2+）。

```redis
SMISMEMBER tags "Redis" "MySQL" "C++"
```

返回：`[1, 0, 1]`

---

## SCARD

返回集合的基数（元素个数）。

```redis
SCARD tags
```

返回：集合中元素的数量。

---

## SRANDMEMBER

随机返回一个或多个成员（**不移除**）。

```redis
SRANDMEMBER tags 2
```

不传入 count 时，返回单个随机成员。

---

## SPOP

随机移除并返回一个或多个成员。

```redis
SPOP tags
SPOP tags 3
```

适用于抽奖、随机出队等场景。

---

## SSCAN

游标迭代集合中的元素（适合大集合）。

```redis
SSCAN tags 0 MATCH Redis* COUNT 10
```

---

# 三、集合间运算（交、并、差）

## SINTER

返回所有给定集合的**交集**（共同元素）。

```redis
SINTER set1 set2 set3
```

---

## SINTERSTORE

将交集结果存储到目标集合（原子操作）。

```redis
SINTERSTORE destination set1 set2
```

返回：结果集合的元素数量。

---

## SINTERCARD

返回交集的基数，可设置提前终止限制（Redis 7.0+）。

```redis
SINTERCARD 2 set1 set2 LIMIT 100
```

---

## SUNION

返回所有给定集合的**并集**（所有元素去重）。

```redis
SUNION set1 set2 set3
```

---

## SUNIONSTORE

将并集结果存储到目标集合。

```redis
SUNIONSTORE destination set1 set2
```

---

## SDIFF

返回第一个集合与其它集合的**差集**（在第一个集合中但不在其它集合中）。

```redis
SDIFF set1 set2 set3
```

---

## SDIFFSTORE

将差集结果存储到目标集合。

```redis
SDIFFSTORE destination set1 set2
```

---

# 四、命令总结

| 命令 | 功能 | 是否阻塞 | 时间复杂度 |
|------|------|---------|-----------|
| `SADD` | 添加元素 | ❌ | O(N) |
| `SREM` | 删除元素 | ❌ | O(N) |
| `SMOVE` | 移动元素 | ❌ | O(1) |
| `SMEMBERS` | 获取所有元素 | ❌ | O(N) |
| `SISMEMBER` | 判断是否存在 | ❌ | O(1) |
| `SMISMEMBER` | 批量判断存在 | ❌ | O(N) |
| `SCARD` | 获取元素个数 | ❌ | O(1) |
| `SRANDMEMBER` | 随机取元素 | ❌ | O(N) |
| `SPOP` | 随机弹出元素 | ❌ | O(N) |
| `SSCAN` | 游标迭代 | ❌ | O(1) 每步 |
| `SINTER` | 交集 | ❌ | O(N*M) |
| `SINTERSTORE` | 交集存目标 | ❌ | O(N*M) |
| `SINTERCARD` | 交集基数 | ❌ | O(N*M) |
| `SUNION` | 并集 | ❌ | O(N) |
| `SUNIONSTORE` | 并集存目标 | ❌ | O(N) |
| `SDIFF` | 差集 | ❌ | O(N) |
| `SDIFFSTORE` | 差集存目标 | ❌ | O(N) |

---

# 五、使用场景示例

## 场景 1：用户标签系统

```redis
SADD user:100:tags "C++" "Redis" "MySQL"
SADD user:101:tags "Python" "Redis" "Linux"
```

---

## 场景 2：共同好友/共同关注

```redis
SADD user:1:follow "user2" "user3" "user4"
SADD user:2:follow "user3" "user5"

SINTER user:1:follow user:2:follow
# 返回：user3
```

---

## 场景 3：抽奖系统（随机不重复抽取）

```redis
SADD lottery:2024 "user100" "user101" "user102"
SPOP lottery:2024 2   # 随机抽取两名获奖者并移除
```

---

## 场景 4：数据去重统计（UV 统计）

```redis
SADD page:1:visitors "ip1" "ip2" "ip3"
SCARD page:1:visitors  # 返回独立访客数（UV）
```

---

## 场景 5：点赞/收藏去重

```redis
SADD post:100:likes "user1" "user2"
SISMEMBER post:100:likes "user1"  # 检查是否已点赞
```

---

## 场景 6：好友推荐（差集）

```redis
# user1 的朋友
SADD user1:friends "A" "B" "C"
# A 的朋友
SADD A:friends "B" "C" "D" "E"

# 推荐 user1 可能认识的人
SDIFF A:friends user1:friends
# 返回：D, E（A的朋友中，user1不认识的人）
```

---

# 六、与传统数据库对比

| 操作 | MySQL | Redis Set |
|------|-------|-----------|
| 去重插入 | 需要先 SELECT 再 INSERT | `SADD` 原子去重 |
| 判断存在 | `SELECT COUNT(*)` | `SISMEMBER` O(1) |
| 交集 | `JOIN` 子查询 | `SINTER` O(N) |
| 随机取元素 | `ORDER BY RAND()` | `SRANDMEMBER` 高效 |
| 批量判断 | `IN` 子查询 | `SMISMEMBER` 一次搞定 |

---

# 七、最佳实践

1. **避免 `SMEMBERS` 大集合**：当集合包含数万成员时，使用 `SSCAN` 迭代。
2. **用 `SINTERCARD` 替代 `SINTER` + `SCARD`**：减少网络传输。
3. **合理使用 `SPOP` 和 `SRANDMEMBER`**：前者弹出（破坏性），后者只读（非破坏性）。
4. **Set 元素推荐 < 1 万**：虽然 Redis Set 支持百万级元素，但大集合的交集运算可能较慢。
5. **结合 `EXPIRE` 设置过期时间**：临时集合（如抽奖、活动）自动清理。

---

# 八、总结

Redis Set 是处理**去重、成员判断、随机抽选、集合运算**的最佳数据结构：

- ✔ 标签系统
- ✔ 共同好友/关注
- ✔ 抽奖/投票去重
- ✔ UV 统计
- ✔ 点赞/收藏去重
- ✔ 好友推荐（差集）
- ✔ 权限/角色管理
- ✔ 数据同步对比