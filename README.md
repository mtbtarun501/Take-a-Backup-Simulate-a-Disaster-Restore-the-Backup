Perfect, Tarun. You're about to master one of the most important skills for a real-world MySQL DBA: **Backup and Disaster Recovery using XtraBackup**.

---

## ✅ GOAL: Take a Backup → Simulate a Disaster → Restore the Backup

**Tools Used**: Percona XtraBackup (physical backup)
**DB Server**: Ubuntu (or Debian-based)

---

## 🔰 Prerequisites

Before we start:

* MySQL is installed and running
* You have `root` or admin access
* You know the database you want to back up (e.g., `payment_gateway`)

---

# ✅ STEP 1: Install XtraBackup

### 🔧 What it does:

Installs the `xtrabackup` tool, which copies **raw data files** from MySQL — not SQL commands like `mysqldump`.

```bash
sudo apt update
sudo apt install percona-xtrabackup-80 -y
```

### ✅ If this step works:

✔️ You'll now have `xtrabackup` and `innobackupex` command-line tools.

### ❌ If not:

* Make sure MySQL 8.0 is installed
* Run: `mysql --version` to verify

---

# ✅ STEP 2: Take Full Physical Backup

```bash
sudo mkdir -p /backups/full
sudo xtrabackup --backup --target-dir=/backups/full --user=root --password=your_password
```

### 🔍 What happens:

* It copies **all your MySQL data files** (`ibd`, logs, etc.)
* Backup is **non-blocking** (no downtime)
* Works even if users are inserting data during the backup

### ✅ If success:

✔️ You’ll see a message:
`xtrabackup: completed OK!`

### ❌ If fails:

* Check password or permissions
* Add `--databases='your_db'` if only one DB needed

---

# ✅ STEP 3: Prepare the Backup

```bash
sudo xtrabackup --prepare --target-dir=/backups/full
```

### 🔍 What this does:

* Applies **redo logs**
* Finalizes any **unfinished transactions**
* Without this, backup will be in an **inconsistent state**

### ✅ If success:

✔️ Message: `xtrabackup: prepare phase completed OK!`

### ❌ If you skip this:

* Restored DB might be corrupt
* You’ll get startup errors in MySQL

---

# ✅ STEP 4: Simulate a Disaster (Test)

> 🧨 Let’s say your `transactions` table has been wrongly deleted!

```sql
USE payment_gateway;
DELETE FROM transactions WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31';
```

> Or worse:

```sql
DROP DATABASE payment_gateway;
```

### 🎯 Real-world reason:

* A script failed
* Someone ran wrong SQL
* System crash corrupted files

---

# ✅ STEP 5: Stop MySQL and Prepare for Restore

```bash
sudo systemctl stop mysql
sudo mv /var/lib/mysql /var/lib/mysql_old   # Backup corrupted DB
sudo mkdir /var/lib/mysql
```

### 🔍 Why?

* You must replace MySQL’s internal DB files with backup
* If MySQL is not stopped, it’ll overwrite or corrupt new files

---

# ✅ STEP 6: Restore the Backup (Copy Back)

```bash
sudo xtrabackup --copy-back --target-dir=/backups/full
sudo chown -R mysql:mysql /var/lib/mysql
```

### 🔍 What this does:

* Moves the backup files into `/var/lib/mysql`
* Changes ownership to the `mysql` user

### ✅ After success:

✔️ MySQL is ready to start again

### ❌ If not:

* Make sure the `--prepare` step was done
* Fix file permissions

---

# ✅ STEP 7: Start MySQL and Verify Recovery

```bash
sudo systemctl start mysql
```

Now check in MySQL:

```sql
USE payment_gateway;
SELECT COUNT(*) FROM transactions;
```

You should see:

* The **rows you deleted are now restored** ✅
* Your DB is back just like it was at the backup time

---

## ✅ Summary Table (Each Step Simplified)

| Step | Command                       | What it Does                           |
| ---- | ----------------------------- | -------------------------------------- |
| 1    | Install XtraBackup            | Prepares the tool                      |
| 2    | `--backup`                    | Takes physical backup of live DB       |
| 3    | `--prepare`                   | Finalizes backup with all transactions |
| 4    | DELETE/DROP                   | Simulates disaster                     |
| 5    | Stop + Clean `/var/lib/mysql` | Gets ready to restore                  |
| 6    | `--copy-back`                 | Copies backup into MySQL               |
| 7    | Start MySQL                   | Bring DB back online                   |

---

## 🧠 Final Thoughts

### 🔥 Why this is important:

* Fast backup of large DBs
* No downtime
* Full disaster recovery
* Used in **banking, fintech, payment apps** daily

---

## Next Step for You

Would you like to:

1. Do **Incremental Backup & Restore**? (For daily/weekly backups)
2. Try **Point-in-Time Recovery** with binary logs?
3. Set up a **cron job for auto backups**?

Let me know, and I’ll guide you hands-on.
