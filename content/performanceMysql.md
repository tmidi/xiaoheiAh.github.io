---
title: "performancemysql"
date: 2019-08-15T13:19:18+08:00
draft: true
---

#### 隐式和显示锁定

Innodb采用的是**两段式锁定协议**，事务执行时，任何情况下都可以实现锁定，只有在执行**commit**和**rollback**时才能释放锁。
显示锁定是通过特定的语句实现，例如:
```sql
SELECT ... LOCK IN SHARE MODE
SELECT ... FOR UPDATE
```

#### 多版本并发控制 MVCC

