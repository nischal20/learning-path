# PostgreSQL Row Locking - Quick Reference

## 🔑 Three Key Rules

### 1. Transaction is MANDATORY ⚠️
- `FOR UPDATE SKIP LOCKED` without transaction = **USELESS**
- Locks released immediately when query completes (auto-commit)
- **Must use transaction** to maintain locks across operations

### 2. Locks are ROW-LEVEL, not Column-Level 🔒
- Lock applies to **entire row**, regardless of columns selected
- `SELECT *` vs `SELECT col1` makes **no difference** for locking
- Once row locked, **any part** of that row is protected

### 3. Reads ALLOWED, Writes BLOCKED 📖

**✅ ALLOWED on locked rows:**
- `SELECT * FROM table WHERE id=1` - Regular reads
- `SELECT col1 FROM table WHERE id=1` - Column reads  
- `SELECT COUNT(*) FROM table` - Aggregations

**❌ BLOCKED/SKIPPED on locked rows:**
- `SELECT * FROM table WHERE id=1 FOR UPDATE` - Lock attempts
- `UPDATE table SET col='x' WHERE id=1` - Modifications  
- `DELETE FROM table WHERE id=1` - Deletions

## 🏗️ Distributed Work Queue Pattern

### Problem Without Locks
Multiple processes grab same work items → Race conditions & duplicate processing

### Solution With Locks
```
Process A                           Process B
┌─────────────────────────┐        ┌─────────────────────────┐
│ BEGIN                   │        │ BEGIN                   │
│ FOR UPDATE SKIP LOCKED  │        │ FOR UPDATE SKIP LOCKED  │
│ → Gets items 1,2,3      │        │ → Gets items 4,5,6      │
│ Process safely          │        │ Process safely          │  
│ COMMIT                  │        │ COMMIT                  │
└─────────────────────────┘        └─────────────────────────┘
```

## 📊 Lock Behavior Matrix

| Query Type | On Locked Row | Result |
|------------|---------------|--------|
| `SELECT` | ✅ | Always works |
| `SELECT FOR UPDATE` | ❌ | Blocked |
| `SELECT FOR UPDATE SKIP LOCKED` | ❌ | Skipped |
| `UPDATE/DELETE` | ❌ | Blocked |

## 🚨 Common Mistakes

1. **No Transaction**: Locks released immediately
2. **Wrong Connection**: Using `db` instead of `tx` for subsequent queries
3. **Long Transactions**: Holding locks too long blocks other processes

## 💡 Best Practices

- Keep transactions **short**
- Always handle **empty results**
- Add proper **indexes** on status/timestamp columns
- Use reasonable **batch sizes** (10-100 items)
- Monitor for **lock contention**

## 🎯 Perfect For

- Distributed job processing
- Work queues with multiple workers  
- Preventing race conditions
- Coordinating concurrent processes

---

**Bottom Line**: Transaction + Row-level locks + Read-allowed/Write-blocked = Safe distributed processing!
