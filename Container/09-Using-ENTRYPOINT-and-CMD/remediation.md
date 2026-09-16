# Lab 9: Using ENTRYPOINT and CMD — Remediation and Best Practices

## 1. Prefer Exec Form for Predictable Execution

For applications where explicit process execution is important, prefer exec-form syntax:

```dockerfile
ENTRYPOINT ["application"]
CMD ["--option"]
```

rather than relying on shell-form commands.

Exec form provides clearer separation between the executable and its arguments.

---

## 2. Keep ENTRYPOINT and CMD Responsibilities Clear

A practical pattern is:

```dockerfile
ENTRYPOINT ["application"]
CMD ["default-argument"]
```

This allows the image to define the primary application while still allowing users to provide different runtime arguments.

---

## 3. Avoid Sensitive Information in CMD or ENTRYPOINT

Do not place passwords, API keys, access tokens, or other secrets directly into:

```dockerfile
CMD [...]
```

or:

```dockerfile
ENTRYPOINT [...]
```

Sensitive configuration should be supplied through appropriate secret-management mechanisms or runtime configuration.

---

## 4. Validate Arguments in ENTRYPOINT Scripts

When using a script as an entrypoint, validate input before using it.

For example:

```sh
#!/bin/sh

if [ -z "$1" ]; then
    echo "Error: required argument is missing"
    exit 1
fi
```

Input validation helps prevent unexpected application behavior.

---

## 5. Use Explicit Executable Paths

When using custom scripts, use an explicit path:

```dockerfile
ENTRYPOINT ["/usr/local/bin/greet.sh"]
```

This makes the intended executable clear and reduces ambiguity.

---

## 6. Avoid Unnecessary Shell Interpretation

Shell-form instructions should only be used when shell features are actually required.

For simple commands, exec form is generally easier to reason about:

```dockerfile
ENTRYPOINT ["echo", "Hello"]
```

instead of:

```dockerfile
ENTRYPOINT echo "Hello"
```

---

## 7. Run Containers as Non-Root Where Practical

Container applications should use a dedicated non-root user whenever application requirements permit it.

This reduces the impact of a potential container compromise.

Example:

```dockerfile
USER 1001
```

The UID should be selected according to the application's permissions and base-image requirements.

---

## 8. Keep Startup Behavior Simple

Container startup commands should be predictable and easy to troubleshoot.

Avoid unnecessary command chaining and complex shell logic when the same behavior can be implemented using explicit executable arguments.

---

## 9. Test Runtime Overrides

Before publishing a container image, test:

```bash
podman run --rm image
```

and, where applicable:

```bash
podman run --rm image custom-arguments
```

Also verify whether the image's `ENTRYPOINT` can or should be overridden using:

```bash
podman run --rm --entrypoint <command> image
```

---

## 10. Security Validation

Before using an image in a production environment, verify:

* Entrypoint behavior
* Runtime argument handling
* Script permissions
* User privileges
* Secret handling
* Shell interpretation
* Signal handling
* Application startup behavior

---

## Conclusion

Correct use of `ENTRYPOINT` and `CMD` improves container predictability, configurability, and operational security.

The recommended approach is to keep the primary application defined through `ENTRYPOINT`, use `CMD` for sensible defaults, validate script arguments, avoid exposing secrets, and prefer exec-form syntax when shell functionality is unnecessary.
