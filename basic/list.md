# Redis List 常用命令总结

Redis 的 **List（列表）** 底层可以理解为一个**双端队列（Deque）**，支持从两端高效插入、删除元素，非常适合实现：

- 消息队列
- 任务队列
- 聊天记录
- 最近浏览记录
- 时间线（Timeline）

---

# 一、插入元素

## LPUSH

从**左侧**插入元素。

```redis
LPUSH mylist A
LPUSH mylist B C
```

结果：

```
C
B
A
```

时间复杂度：

```
O(1)
```

适用于：

- 栈（Stack）

---

## RPUSH

从**右侧**插入元素。

```redis
RPUSH mylist A
RPUSH mylist B
```

结果：

```
A
B
```

适用于：

- 队列（Queue）

---

## LPUSHX

只有 List 已存在才插入。

```redis
LPUSHX mylist hello
```

如果不存在：

```
(nil)
```

不会创建新的 List。

---

## RPUSHX

与 LPUSHX 相同，不过从右侧插入。

---

# 二、弹出元素

## LPOP

从左侧删除并返回元素。

```
A B C
↑
```

```redis
LPOP mylist
```

返回：

```
A
```

剩余：

```
B C
```

Redis 6.2 起支持：

```redis
LPOP mylist 3
```

一次弹出多个元素。

---

## RPOP

从右侧删除并返回元素。

```
A B C
      ↑
```

返回：

```
C
```

---

# 三、查看元素

## LINDEX

查看指定位置元素。

```redis
LINDEX mylist 0
```

返回：

```
第一个元素
```

支持负数索引：

```redis
LINDEX mylist -1
```

返回最后一个元素。

---

## LRANGE

获取区间元素。

获取全部：

```redis
LRANGE mylist 0 -1
```

例如：

```
A
B
C
D
E
```

获取部分：

```redis
LRANGE mylist 1 3
```

得到：

```
B
C
D
```

这是 List 最常用命令。

---

## LLEN

查看 List 长度。

```redis
LLEN mylist
```

返回：

```
5
```

---

## LPOS

查找元素第一次出现的位置。

```redis
LPOS mylist hello
```

返回：

```
3
```

查找第二次出现：

```redis
LPOS mylist hello RANK 2
```

---

# 四、修改元素

## LSET

修改指定位置元素。

```redis
LSET mylist 2 Redis
```

结果：

```
A
B
Redis
D
```

---

## LINSERT

指定元素前后插入。

例如：

```
A
B
C
```

前插：

```redis
LINSERT mylist BEFORE B X
```

结果：

```
A
X
B
C
```

后插：

```redis
LINSERT mylist AFTER B X
```

结果：

```
A
B
X
C
```

---

# 五、删除元素

## LREM

删除指定值。

例如：

```
A
B
A
C
A
```

删除一个：

```redis
LREM mylist 1 A
```

结果：

```
B
A
C
A
```

删除全部：

```redis
LREM mylist 0 A
```

结果：

```
B
C
```

从尾部开始删除：

```redis
LREM mylist -1 A
```

---

## LTRIM

保留指定区间。

例如：

```
A
B
C
D
E
```

```redis
LTRIM mylist 0 2
```

结果：

```
A
B
C
```

典型应用：

最近100条记录

```redis
LPUSH history msg
LTRIM history 0 99
```

---

# 六、移动元素

## LMOVE

将一个 List 中的元素移动到另一个 List。

例如：

```
list1

A B C

list2

X Y
```

```redis
LMOVE list1 list2 LEFT RIGHT
```

表示：

- 从 list1 左边取
- 放到 list2 右边

结果：

```
list1

B C

list2

X Y A
```

---

## BLMOVE

阻塞版 LMOVE。

如果源 List 没有数据：

等待 timeout 秒。

适用于：

- 消费者

---

## RPOPLPUSH（旧命令）

等价于：

```
LMOVE source destination RIGHT LEFT
```

从右边取元素，放到另一个 List 左边。

---

## BRPOPLPUSH

阻塞版本。

---

# 七、多 List 弹出（Redis 7）

## LMPOP

多个 List 中弹出元素。

例如：

```redis
LMPOP 3 list1 list2 list3 LEFT
```

Redis 自动找到第一个非空 List 并弹出。

---

## BLMPOP

阻塞版本。

如果所有 List 都为空，则等待。

---

# 八、阻塞读取

Redis 消息队列最经典命令。

---

## BLPOP

阻塞左弹。

```redis
BLPOP queue 30
```

表示：

等待30秒。

生产者：

```redis
LPUSH queue job
```

消费者立即被唤醒。

---

## BRPOP

阻塞右弹。

与 BLPOP 相同，只是从右边读取。

---

# 九、命令总结

| 命令 | 功能 | 是否阻塞 |
|------|------|---------|
| LPUSH | 左插 | ❌ |
| RPUSH | 右插 | ❌ |
| LPUSHX | 左插（必须存在） | ❌ |
| RPUSHX | 右插（必须存在） | ❌ |
| LPOP | 左弹 | ❌ |
| RPOP | 右弹 | ❌ |
| BLPOP | 左弹 | ✅ |
| BRPOP | 右弹 | ✅ |
| LLEN | 长度 | ❌ |
| LRANGE | 获取区间 | ❌ |
| LINDEX | 获取指定元素 | ❌ |
| LPOS | 查找元素位置 | ❌ |
| LSET | 修改元素 | ❌ |
| LINSERT | 前后插入 | ❌ |
| LREM | 删除元素 | ❌ |
| LTRIM | 保留区间 | ❌ |
| LMOVE | List 间移动 | ❌ |
| BLMOVE | 阻塞移动 | ✅ |
| RPOPLPUSH | 右弹左插（旧） | ❌ |
| BRPOPLPUSH | 阻塞版（旧） | ✅ |
| LMPOP | 多 List 弹出 | ❌ |
| BLMPOP | 阻塞多 List 弹出 | ✅ |

---

# 总结

Redis List 最适合以下几类场景：

- ✔ 消息队列
- ✔ 工作队列
- ✔ 最近浏览
- ✔ 操作日志
- ✔ 聊天记录
- ✔ 时间线（Timeline）
- ✔ 通知列表
