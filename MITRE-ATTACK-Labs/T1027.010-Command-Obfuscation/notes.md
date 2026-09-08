# T1027.010 — Command Obfuscation Notes

## 1. What is Command Obfuscation?

Command obfuscation is the modification or reconstruction of a command so that its intended content or structure is less obvious while preserving its functionality.

The command may still perform the same operation, but the syntax presented to a basic signature-based detector can look different.

---

## 2. Techniques Demonstrated

### String Splitting

A recognizable string was divided between multiple variables.

```bash
a="MITRE"
b="T1027.010"
echo "$a $b controlled test"
```

The shell expands the variables before execution.

---

### Environment Variables

The technique identifier was stored in an environment variable.

```bash
export TECHNIQUE="T1027.010"
printf "%s\n" "MITRE $TECHNIQUE Command Obfuscation"
```

Environment variables can make static command inspection more difficult because important values may not appear directly in the command string.

---

### Character Construction

A word was assembled from several variables.

```bash
c1="M"
c2="I"
c3="TRE"

printf "%s\n" "$c1$c2$c3 T1027.010 controlled test"
```

The resulting value is equivalent to the original word after shell expansion.

---

### Command Reconstruction

The command name itself was constructed from string fragments.

```bash
cmd="ec""ho"
$cmd "MITRE T1027.010 reconstructed command"
```

Bash resolves the variable to:

```text
echo
```

and executes the resulting command.

---

## 3. Bash Execution Trace

The `bash -x` option provides execution tracing.

The trace showed:

```text
+ a=MITRE
+ b=T1027.010
+ echo 'MITRE T1027.010 controlled test'
```

It also showed:

```text
+ cmd=echo
+ echo 'MITRE T1027.010 reconstructed command'
```

This demonstrates that Bash performs variable expansion and command reconstruction before executing the resulting command.

---

## 4. Detection Lessons

Simple string matching may fail when important command components are split or dynamically constructed.

Detection should therefore consider:

* Process creation
* Parent-child relationships
* Shell interpreters
* Command-line arguments
* Environment variables
* Execution context
* Repeated suspicious shell activity
* Behavioral patterns

---

## 5. Process Telemetry

A background process-monitoring loop was used to observe active processes inside the container.

The process table showed the monitoring Bash process and controlled test processes.

Because `docker exec bash -c` creates short-lived processes, commands may disappear from the process table quickly. Long-running controlled processes were therefore used to provide observable process evidence.

---

## 6. Command History Observation

A history file was configured for the controlled Bash session.

The experiment demonstrated an important operational limitation: non-interactive `bash -c` sessions do not behave exactly like normal interactive shell sessions with respect to history.

Therefore, shell history alone should not be treated as a complete source of Linux command telemetry.

---

## 7. Docker Isolation

The lab network was configured as:

```text
Internal: true
```

The resulting network was:

```text
172.22.0.0/16
```

with:

```text
t1027-command-lab -> 172.22.0.3
t1027-log-monitor -> 172.22.0.2
```

An outbound connectivity test returned:

```text
EXPECTED: outbound connectivity blocked
```

This reduced the risk of accidental external communication during testing.

---

## 8. Security Takeaways

1. Obfuscation can change command representation without changing behavior.
2. Static signatures may miss fragmented commands.
3. Shell execution telemetry is valuable for Linux detection.
4. Process relationships provide important investigation context.
5. Detection should combine syntax and behavior.
6. Controlled security testing should use isolated environments.
7. Non-interactive shell history is not a reliable standalone telemetry source.

---

## 9. Key Learning

The most important lesson from this lab is:

> Detecting command obfuscation should focus on what the command ultimately does, not only how the original command string looks.
