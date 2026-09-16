# Lab 10: Findings

## Finding 1 — Named Volume Provides Persistent Storage

**Observation:**
The `myapp_data` volume retained `index.html` after the original `webapp` container was removed.

**Evidence:**

```text
Hello, Volume!
```

was still available inside the newly created `webapp_new` container.

**Impact:**
Application data stored in a named volume is not tied to the lifecycle of a single container.

**Status:** Verified

---

## Finding 2 — Bind Mount Provides Direct Host-to-Container Mapping

**Observation:**
The host directory `~/host_data` was successfully mounted into the Nginx container.

**Evidence:**

```text
Hello, Bind Mount!
```

was visible inside the container.

**Impact:**
Applications running inside the container can access data directly from the mapped host directory.

**Status:** Verified

---

## Finding 3 — Bind Mount Changes Are Immediately Visible

**Observation:**
After updating the host-side `index.html`, the updated content was visible inside the running container.

**Evidence:**

```text
Hello, Bind Mount!
Updated content!
```

**Impact:**
Bind mounts allow changes to host files to be reflected directly inside the container without rebuilding the image.

**Status:** Verified

---

## Finding 4 — Shell History Expansion During Data Creation

**Observation:**
The exclamation mark in `Hello, Volume!` initially triggered Bash history expansion.

**Error:**

```text
-bash: !': event not found
```

**Resolution:**
The command was changed to use single quotes around the outer shell command.

**Status:** Resolved

---

## Overall Result

The lab successfully demonstrated:

* Named volume creation.
* Volume inspection.
* Volume mounting.
* Persistent data storage.
* Container recreation with preserved data.
* Host bind mounts.
* Live file updates through bind mounts.
