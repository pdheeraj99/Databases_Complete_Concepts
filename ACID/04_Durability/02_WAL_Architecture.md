# The Mechanism: Write Ahead Log (WAL) 📝

## 1. "Log First, Data Next" Rule

Database heavy lifting (Main Tables update) chese mundu, changes ni oka simple **Log File** lo rastundi. Deenne **WAL** antaru.

```mermaid
graph LR
    User[👤 User Update] -->|1. Change| Buffer[📝 Log Buffer<br/>RAM]
    Buffer -->|2. Sequential Write| WAL[📁 WAL File<br/>Disk/SSD]
    WAL -.->|3. Checkpoint<br/>Later| DB[(📊 Main Tables)]
    
    style WAL fill:#90EE90,stroke:#333,stroke-width:2px
    style Buffer fill:#FFB6C1,stroke:#333,stroke-width:2px
```

### Why WAL is Fast?

| Feature | Description |
|:---|:---|
| **Sequential Writes** | Like adding new line to end of book - no searching |
| **Minimal Data** | Only "Delta" (change) stored, not entire row |
| **Example** | `"AccountID 101: 500 → 1000"` instead of full row |

---

## 2. The Villain: OS Cache 🤥

```text
Database: "OS, ee log ni Disk meeda rayi."
OS:       "Ha rasesa ✅" (lies!)
Reality:  OS keeps data in its own RAM cache 🤫
```

> ⚠️ **The Risk:** Power pothe, OS Cache lo unna data gallo kalisipothundi!

---

## 3. The Hero: `fsync` Command 🦸

OS abaddham cheppakunda undadaniki, Database **`fsync`** system call vadutundi.

```sql
-- PostgreSQL configuration
synchronous_commit = on  -- Wait for disk write (Safe)
synchronous_commit = off -- Don't wait (Fast but risky)

-- MySQL/InnoDB
innodb_flush_log_at_trx_commit = 1  -- fsync every commit (Safe)
innodb_flush_log_at_trx_commit = 2  -- fsync every second (Fast)
```

| Setting | Speed | Durability |
|:---|:---|:---|
| `fsync` ON | 🐢 Slower | ✅ Guaranteed |
| `fsync` OFF | 🚀 Faster | ⚠️ Risky |

---

## 🎯 Key Takeaways

1. **WAL = Insurance Policy** - Even if crash happens, log has the data
2. **OS Lies!** - Always use `fsync` for critical data
3. **Trade-off Available** - Choose speed vs safety based on use case
4. **Sequential = Speed** - Append-only design is the key
