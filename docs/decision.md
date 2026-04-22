<!-- .slide: data-background-color="#6db33f" data-background-transition="zoom" -->
# Section 3
## The "Should I Externalize It?" Decision Tree

Notes:
- Now let's turn the spectrum into a repeatable checklist. Five questions. Ask them in order.

---

## The Five Questions

Ask these in order for any property:

1. Does this value differ between environments?
2. Could this change without a redeploy?
3. Is this value sensitive?
4. Is this value shared across multiple apps?
5. None of the above?

Notes:
- First "yes" wins. You don't need to ask all five if you hit a yes early.
- Question 3 is the override: sensitivity beats everything else. Even a low-volatility secret belongs in Vault.

---

## Q1: Environment-Specific?

> Does this value differ between dev, staging, and prod?

```properties
# dev
spring.datasource.url=jdbc:h2:mem:testdb

# prod
spring.datasource.url=jdbc:postgresql://prod-db.internal:5432/myapp
```

**Yes → externalize it.**

Notes:
- If you have to change the JAR or the base application.properties to deploy to a different environment, you have environment-specific config that should be externalized.

---

## Q2: Runtime Changeable?

> Could this value need to change without a code redeploy?

* Feature flags that marketing toggles
* Rate limits that ops adjusts under load
* Connection pool sizes tuned after a capacity event

**Yes → externalize it** (and consider `@RefreshScope` for hot-reload).

Notes:
- @RefreshScope + Spring Cloud Bus lets you push a config change and have running instances pick it up without a restart.
- Not everything needs live reload, but if the answer to "could this change at runtime" is yes, design for it.

---

## Q3: Sensitive?

> Is this a secret, credential, API key, or PII-adjacent value?

```yaml
# BAD — this is a ConfigMap (or worse, source control)
spring:
  datasource:
    password: s3cr3t

# GOOD — Vault reference resolved at startup
spring:
  datasource:
    password: ${db.password}
```

**Yes → Vault. No exceptions.**

Notes:
- Sensitivity is the hardest override. Even if the password hasn't changed in 3 years, it belongs in Vault.
- Vault gives you audit trails, rotation, and access policies that ConfigMaps and application.properties cannot provide.

---

## Q4: Shared Across Apps?

> Do multiple services reference the same value?

* Shared DB endpoint used by 4 microservices
* Organization-wide API gateway URL
* Common feature flag controlling a cross-service behavior

**Yes → Spring Cloud Config** (Git-backed, treat config as code).

Notes:
- If you update a shared value in one app's ConfigMap, the other three services still have the old value. That's a consistency incident.
- Config Server is the source of truth for shared, non-sensitive config.

---

## Q5: None of the Above?

> It's not environment-specific, not runtime-changeable, not sensitive, not shared.

```properties
# Truly static default — fine to leave here
server.shutdown=graceful
spring.jpa.open-in-view=false
```

**Reasonable to leave in `application.properties` or hardcoded — but document why.**

Notes:
- The documentation step matters. "Why does this live here?" prevents the next engineer from moving it unnecessarily.
- A comment or ADR entry is sufficient. Just leave a breadcrumb.

---

## Decision Tree Summary

| Question | Yes → | No → |
|---|---|---|
| Environment-specific? | Externalize | Next question |
| Runtime changeable? | Externalize | Next question |
| Sensitive? | **Vault** | Next question |
| Shared across apps? | Spring Cloud Config | Next question |
| None of the above? | `application.properties` | — |

Notes:
- This table is the take-home artifact. Print it, put it in your team wiki, reference it in code review.
- The goal is a defensible, consistent answer — not perfection on the first try.
