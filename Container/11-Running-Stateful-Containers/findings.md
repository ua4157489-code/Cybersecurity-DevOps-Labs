# Lab 11: Findings

## Finding 1 — Database Data Can Outlive the Container

**Observation:**
The MySQL database was stored on a host-mounted directory.

After the original MySQL container was removed, a replacement container was started using the same storage.

**Result:**
The previously created `lab_data` table and test record remained available.

**Status:** Verified

---

## Finding 2 — MySQL Successfully Operated as a Stateful Container

**Observation:**
MySQL was configured with persistent storage at:

```text
/var/lib/mysql
```

The database contained the test record:

```text
Persistent test data
```

**Result:**
The database state survived container replacement.

**Status:** Verified

---

## Finding 3 — PostgreSQL Successfully Used Persistent Storage

**Observation:**
PostgreSQL was configured with a host-mounted storage directory at:

```text
/var/lib/postgresql/data
```

A `lab_data` table and test record were created.

**Result:**
The data remained available after container recreation.

**Status:** Verified

---

## Finding 4 — Container Lifecycle and Data Lifecycle Are Separate

**Observation:**
Removing a database container did not remove the host storage directory.

**Result:**
A replacement container could reuse the existing database state.

**Security/Operational Impact:**
Stateful workloads require deliberate storage lifecycle management because deleting a container does not necessarily delete its associated data.

**Status:** Verified

---

## Finding 5 — Host Storage Requires Protection

Database files were stored directly on the host filesystem.

This means host filesystem permissions and access controls become important security boundaries.

**Status:** Security consideration identified

---

## Overall Result

The lab demonstrated that persistent storage allows stateful database applications to survive container replacement.

Both MySQL and PostgreSQL were successfully used with persistent host storage.
