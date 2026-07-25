# Home Lab Penetration Test Report

**Date:** July 2026
**Tester:** Nikko
**Attacking Machine:** Kali Linux 2026.2 (VirtualBox, bridged networking)
**Target:** Ubuntu Server VM (VirtualBox, bridged networking) — self-hosted home lab

## Scope

Authorized, self-testing of a personally owned home lab environment. Target services running on the Ubuntu Server VM via Docker:

| Service | Port(s) | Purpose |
|---|---|---|
| SSH | 22 | Remote administration |
| PiHole | 53, 8082 (admin), 8443 (HTTPS) | DNS filtering |
| Nextcloud | 8080, 8444 (HTTPS) | File sync |
| Vaultwarden | 8081, 8445 (HTTPS) | Password management |
| Nginx | 80, 8443/8444/8445 | Reverse proxy (self-signed TLS) |

Prior to testing, a VirtualBox snapshot (`pre-pentest-baseline`) was taken of the Ubuntu Server VM to allow rollback if needed.

## Methodology

1. **Environment setup** — Imported the Kali Linux VirtualBox appliance (`.vbox`/`.vdi`) via VirtualBox's *Machine → Add* option, configured with a bridged network adapter to reach the Ubuntu Server VM on the same subnet.
2. **Reconnaissance** — Port and service enumeration with `nmap`.
3. **Vulnerability research** — Local exploit database lookups (`searchsploit`) against identified service versions.
4. **Web application testing** — Automated scanning with `nikto` and manual endpoint checks with `curl`.
5. **Configuration review** — Manual verification of authentication requirements, admin panel exposure, and signup/registration settings.

## Findings Summary

### 1. SSH (Port 22) — ✅ No issues found
- `nmap --script ssh-auth-methods` confirmed **publickey-only authentication**; password authentication is disabled.
- Fail2ban is active for additional brute-force protection.
- **Result:** Password-based brute-forcing is not viable against this service.

### 2. Nextcloud (Port 8080) — ✅ No issues found
- Running version **33.0.0.16** (current at time of test).
- `/remote.php/webdav/` (WebDAV sync endpoint) correctly returns `401 Unauthorized` for unauthenticated requests.
- `searchsploit` results for older Nextcloud versions (e.g., a v17 CSRF exploit) did not apply — version far exceeds the affected range.
- No exposed configuration files or public status leaks beyond the expected `status.php` metadata.

### 3. PiHole Admin (Port 8082) — ✅ No issues found
- Admin interface returns `302 Found` (redirect to login) for unauthenticated requests — properly gated.
- Password confirmed to be set and functional (not blank or default).

### 4. Vaultwarden (Port 8081) — ⚠️ Two issues found and remediated

**Issue A: Open user signups**
- `curl http://<host>:8081/api/config` showed `signupsAllowed` was not set to `false`.
- **Risk:** Any device on the local network could register a new Vaultwarden account.
- **Fix:** Added `SIGNUPS_ALLOWED=false` to the Vaultwarden service environment in `docker-compose.yml`.

**Issue B: No admin token configured**
- The `/admin` panel initially responded with a message indicating the admin panel was disabled because no `ADMIN_TOKEN` was set — a safe default, but meant admin functionality (user management, diagnostics) was entirely unavailable.
- **Fix:** Generated a proper Argon2-hashed admin token via:
  ```bash
  sudo docker exec -it <vaultwarden-container-name> ./vaultwarden hash
  ```
  and added it to `docker-compose.yml`. Note: within Docker Compose, every `$` character in the Argon2 hash must be escaped as `$$`, or Compose's variable interpolation will corrupt the value.
- **Result:** `/admin` now correctly prompts for the token/password before granting access; no dashboard data is exposed pre-auth.

## Remediation Actions Taken

- [x] Disabled Vaultwarden signups (`SIGNUPS_ALLOWED=false`)
- [x] Configured a properly hashed Vaultwarden `ADMIN_TOKEN`
- [x] Verified `/admin` requires authentication post-fix

## Notes / Lessons Learned

- Docker Compose interprets `$` as the start of variable substitution, even inside quoted strings — Argon2 hashes (which are full of `$` delimiters) must have every `$` doubled (`$$`) to be passed through correctly.
- A `200 OK` HTTP status code is not sufficient on its own to judge whether an endpoint is secure — the response body must be inspected, since Vaultwarden returned `200` both when the admin panel was safely disabled and when displaying the login form.
- `nmap`'s service/version detection can mislabel non-standard web servers (e.g., misidentified Vaultwarden and PiHole's lighttpd as unrelated services); banner-based fingerprinting should not be trusted without manual verification.

## Follow-Up / Future Testing

- [ ] Brute-force resistance testing against PiHole admin login (no default rate-limiting/lockout mechanism confirmed yet)
- [ ] Brute-force / rate-limit testing against Vaultwarden's user login (separate from the admin panel)
- [ ] TLS cipher suite review (`nmap --script ssl-enum-ciphers`) against self-signed cert ports 8443/8444/8445
- [ ] Directory/file brute-forcing (`gobuster`) against Nextcloud and Vaultwarden for exposed backup or config files
