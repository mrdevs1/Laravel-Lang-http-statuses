# Laravel-Lang/http-statuses: Supply-Chain Attack – Complete Technical Guide

## 1. Project Overview

**Repository:** `Laravel-Lang/http-statuses`

Laravel-Lang/http-statuses is an open-source localization package that provides translated HTTP status messages for Laravel applications. It helps developers display localized descriptions of HTTP response codes across different languages.

**GitHub:** https://github.com/Laravel-Lang/http-statuses

## 2. Incident Overview

On May 22–23, 2026, the Laravel Lang ecosystem experienced a supply-chain attack involving compromised Git version tags. The attacker changed historical version tags to point to malicious commits instead of their original legitimate commits.

Affected repositories included:

- `Laravel-Lang/lang`
- `Laravel-Lang/http-statuses`
- `Laravel-Lang/attributes`
- `Laravel-Lang/actions`

## 3. How the Attack Worked

The attack involved three stages:

1. **Git tag compromise:** Historical version tags were modified to point to attacker-controlled commits.
2. **Composer autoload execution:** A malicious `src/helpers.php` file was registered under `autoload.files` in `composer.json`.
3. **Second-stage payload:** The injected PHP code downloaded an external payload designed to steal sensitive credentials and environment secrets.

## 4. Technical Details

### 4.1 Git Tag Manipulation

Git tags identify particular commits. In this incident, familiar version tags were changed to reference malicious commits. Developers could therefore install different source code while requesting a familiar package version.

### 4.2 Malicious Composer Configuration

The compromised commit introduced `src/helpers.php` and registered it under Composer's file autoload mechanism:

```json
{
  "autoload": {
    "files": ["src/helpers.php"]
  }
}
```

Composer automatically includes files listed under `autoload.files` when its autoloader is loaded.

For example:

```php
require __DIR__ . '/vendor/autoload.php';
```

This created an automatic execution path for the malicious code.

### 4.3 Second-Stage Payload

Researchers identified the malicious infrastructure as `flipboxstudio[.]info`.

The payload was designed to collect sensitive information, including:

- Cloud credentials
- SSH keys
- GitHub tokens
- Database credentials
- CI/CD secrets
- Environment variables

## 5. Why the Attack Was Dangerous

- Developer workstation compromise.
- Production credential exposure.
- CI/CD pipeline compromise.
- Unauthorized cloud infrastructure access.
- Database and application secret exposure.

## 6. Investigating an Affected Laravel Project

### Step 1: Identify Installed Dependencies

Inspect `composer.json`:

```bash
cat composer.json
```

Search for the affected package:

```bash
grep -n -A 12 -B 2 "laravel-lang/http-statuses" composer.lock
```

Also investigate the other affected Laravel-Lang packages.

### Step 2: Inspect composer.lock

Check the package name, version, source reference, and distribution reference.

Do not rely on the version number alone to establish integrity.

### Step 3: Inspect Composer Autoload Configuration

Check:

```text
vendor/laravel-lang/http-statuses/composer.json
```

Review unexpected entries under `autoload.files` and inspect unexpected PHP helper files.

### Step 4: Review Installation History

Examine:

- CI/CD job logs
- Deployment logs
- Composer installation logs
- GitHub Actions logs
- Developer machine security logs

## 7. Indicators of Compromise

| Indicator | Investigation |
|---|---|
| `flipboxstudio[.]info` | DNS and network connection history |
| `src/helpers.php` | Unexpected package file |
| `LARAVEL_LANG_HELPERS` | Suspicious code references |
| `.laravel_locale` | Unexpected temporary directory |
| `DebugChromium.exe` | Unexpected executable |
| Unexpected PHP child processes | Process execution history |

These indicators require investigation. Their absence does not prove a system was unaffected.

## 8. How to Fix and Recover

1. Stop deployments from potentially affected CI runners.
2. Preserve relevant logs and forensic evidence.
3. Verify installed package references against independently trusted source code.
4. Remove the affected dependency if it is not essential.
5. If the package is needed, restore it only from a verified release or immutable commit.
6. Rotate potentially exposed GitHub tokens, cloud credentials, database passwords, API keys, SSH keys, and deployment secrets.
7. Review access logs for unauthorized activity.
8. Rebuild compromised environments from trusted sources.
9. Monitor network and system activity after recovery.

**Important:** Removing `vendor/` or reinstalling Composer packages is not sufficient to clean a machine that has already executed malware.

## 9. Preventing Similar Attacks

| Security practice | Purpose |
|---|---|
| Pin dependencies to verified commits | Reduce dependence on mutable tags |
| Review `composer.lock` changes | Detect unexpected dependency changes |
| Audit Composer autoload files | Identify unexpected eager execution |
| Enable dependency scanning | Identify known malicious packages |
| Enable GitHub 2FA | Protect maintainer accounts |
| Use short-lived CI tokens | Limit credential exposure |
| Restrict network egress | Reduce unauthorized data transfers |
| Monitor Git tag changes | Detect suspicious release manipulation |

## 10. Safe Technical Reproduction

To understand the underlying Composer behavior, use a disposable local project and a harmless PHP helper file.

Create `src/helpers.php`:

```php
<?php

file_put_contents(
    __DIR__ . '/autoload-test.log',
    "Composer autoload executed\n",
    FILE_APPEND
);
```

Register the file under `autoload.files` in a local test package, then load Composer's autoloader.

Observe that the helper executes automatically.

This demonstrates Composer's behavior without downloading or executing the malicious payload.

## 11. Important GitHub and Security References

- Repository: https://github.com/Laravel-Lang/http-statuses
- Security issue #277: https://github.com/Laravel-Lang/http-statuses/issues/277
- Related ecosystem report #257: https://github.com/Laravel-Lang/common/issues/257
- StepSecurity investigation: https://www.stepsecurity.io/blog/laravel-lang-supply-chain-attack
- Snyk advisory: https://snyk.io/blog/laravel-lang-supply-chain-advisory/
- Snyk vulnerability details: https://security.snyk.io/vuln/SNYK-PHP-LARAVELLANGHTTPSTATUSES-16801060
- BleepingComputer report: https://www.bleepingcomputer.com/news/security/laravel-lang-packages-hijacked-to-deploy-credential-stealing-malware/

## 12. Technical Summary

| Item | Details |
|---|---|
| Repository | Laravel-Lang/http-statuses |
| Ecosystem | Laravel / PHP |
| Package manager | Composer |
| Incident type | Supply-chain attack |
| Attack method | Git tag rewriting |
| Malicious file | `src/helpers.php` |
| Execution mechanism | Composer `autoload.files` |
| Main impact | Credential and secret theft |
| Attack period | May 22–23, 2026 |
| Security report | #277 |
| CWE | CWE-506 – Embedded Malicious Code |

## 13. What Laravel Developers Should Learn

After studying this incident, developers should understand:

1. What a Composer supply-chain attack is.
2. How Git tags and commits work.
3. The difference between PSR-4 autoloading and file autoloading.
4. Why `autoload.files` can execute code automatically.
5. How compromised dependencies can expose Laravel `.env` secrets.
6. How to investigate suspicious Composer dependencies.
7. How to secure GitHub Actions and Composer dependencies.
8. How to recover from a compromised dependency.

## Conclusion

A trusted package name and familiar version number do not guarantee trusted source code. Dependency integrity must be verified independently.

This incident demonstrates why Laravel developers must understand Composer autoloading, Git release integrity, dependency security, and incident response.
