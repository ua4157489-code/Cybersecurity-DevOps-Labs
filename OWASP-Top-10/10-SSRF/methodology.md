# Methodology - OWASP A10: Server-Side Request Forgery

## Target
OWASP Juice Shop (local Docker container, `127.0.0.1:3000`)

## Approach
1. **Black-box probing first** - tested the `/redirect?to=` endpoint with an arbitrary external URL to establish baseline behavior (rejected, confirming an allowlist exists rather than open redirection to anything).
2. **Informed guessing from prior recon** - attempted URLs seen in earlier labs' admin-configuration output as plausible allowlist candidates. These failed, indicating the actual allowlist differs from URLs referenced elsewhere in the app's configuration.
3. **Source-level verification** - rather than continuing to guess, extracted the actual route handler and validation logic directly from the running container's filesystem using `docker cp` (chosen over `docker exec` after discovering the container image has no shell or `node` binary available for interactive exec). This is a legitimate black-box-to-white-box escalation technique when black-box guessing stalls and the target is a container you control.
4. **Root cause identification** - read the extracted source to find the exact validation function (`isRedirectAllowed`) and its real allowlist (`redirectAllowlist`), revealing the use of `.includes()` rather than strict matching.
5. **Exploit construction from the root cause** - built two distinct working bypass payloads directly informed by the confirmed logic flaw: one placing the allowed string as a URL-prefix with an attacker suffix appended, one placing the allowed string inside a query parameter on a fully attacker-controlled domain. Both were verified to produce a live `302 Found` redirect to attacker-controlled infrastructure.

## Tools Used
- `curl -i` to inspect full response headers (status code and `Location` header are the key indicators of a successful redirect)
- `docker cp` to extract server-side source files for white-box root-cause analysis
- `grep` for isolating the relevant allowlist/validation code from the extracted source

## Scope
Limited to the `/redirect` endpoint's URL validation logic. The image-upload-by-URL feature (a second commonly-cited Juice Shop SSRF vector) was identified as an area for future testing but not exercised in this session, since a complete, source-verified finding on the redirect endpoint was already achieved.
