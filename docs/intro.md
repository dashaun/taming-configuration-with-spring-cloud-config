# Taming Configuration
## A Practical Guide to Externalizing Config with Spring Boot

### Spring Cloud Config · Kubernetes · HashiCorp Vault

Ryan Baxter | Spring Cloud Team

DaShaun Carter | Spring Developer Advocate

Notes:
- Welcome everyone. Over the next 50 minutes we'll build a repeatable decision framework for externalizing Spring Boot configuration.
- The goal is not rigid rules — it's operational agility. By the end you'll have a mental model you can apply immediately.
- Ask questions anytime. There's a Q&A block at the end but interrupt if something needs clarifying.

---

## Overview

A repeatable decision framework for externalizing Spring Boot config — not a set of rigid rules.

| Section | Time |
|---|---|
| Introduction & Framing | ~5 min |
| The Configuration Spectrum | ~10 min |
| Decision Tree | ~10 min |
| Four-Tier Layering Model | ~15 min |
| The etcd Size Problem | ~10 min |
| Patterns & Anti-Patterns + Flowchart | ~5 min |
| Q&A | ~5 min |

Notes:
- Quick roadmap. We'll move at a good pace but stop for demos and questions.
- Sections build on each other — the spectrum informs the decision tree, the decision tree drives the layering model.

]]]

<!-- .slide: data-background-color="#6db33f" data-background-transition="zoom" -->
# Section 1
## Introduction & Framing

Notes:
- Let's set the stage. Why are we even talking about this?

---

## The Core Tension

> Hardcoded simplicity vs. operational flexibility

* **Day 1**: `localhost:5432` works. Ship it.
* **Day 30**: staging needs a different DB. Copy the JAR, tweak a constant. Fine.
* **Day 180**: production incident. The timeout is wrong. Redeploy required. 2 AM.

Notes:
- Every team starts with hardcoded values. It's the right call early on.
- The pain comes when the app is running in N environments with N slightly-different configs and the only lever you have is a redeploy.

---

## "It'll Never Change"

> The most dangerous phrase in software configuration.

* Environment-specific values **always** diverge eventually
* Credentials **must** rotate — and sooner than you expect
* Distributed systems amplify config drift across services

Notes:
- The assumption of stability is almost always wrong in the long run.
- In distributed systems, config drift compounds — one service's wrong assumption breaks a dependency chain.

---

## Goal of This Talk

**A repeatable decision framework — not rigid rules.**

Three questions to answer for any property:

1. Where does this value belong?
2. Who owns it?
3. How does it change safely?

> Over-externalizing creates its own maintenance burden.
> The goal is operational agility, not configuration maximalism.

Notes:
- This is the key message. We're not saying "externalize everything." We're saying "externalize the right things."
- The framework gives you a defensible answer for every property.

]]]

<!-- .slide: data-background-color="#191e1e" data-background-transition="zoom" -->
# Section 2
## The Configuration Spectrum

Notes:
- Before we can decide where to put config, we need a way to classify it.

---

## Two Axes

| Axis | Question |
|---|---|
| **Volatility** | How likely is this to change? Per environment? At runtime? |
| **Scope** | Is this shared across apps, or specific to one? |

Notes:
- Plot any property on these two axes and the right home becomes obvious.
- High volatility + shared scope = top priority for externalization.
- Low volatility + single app = probably fine to leave in application.properties.

---

## The Spectrum in Practice

| Property | Volatility | Scope | Lives in… |
|---|---|---|---|
| Database URL | High | Shared | Spring Cloud Config |
| DB credentials | High | Shared | **Vault** |
| Feature flags | Medium | App-specific | ConfigMap or Config Server |
| Thread pool size | Low | App-specific | ConfigMap or application.properties |
| Request timeout | Low | App-specific | application.properties |
| Magic business constant | Near-zero | App-specific | Code (document why) |

Notes:
- Walk through each row. The pattern becomes intuitive quickly.
- Credentials are always Vault — even if "low volatility." Sensitivity overrides everything.

---

## Volatility: The Time Dimension

* **Per-environment volatility**: dev uses H2, prod uses Aurora → externalize
* **Runtime volatility**: feature flag toggled without redeploy → externalize
* **Rotation volatility**: DB password rotated quarterly → Vault

Notes:
- Three flavors of "likely to change." Each one drives a slightly different externalization strategy.

---

## Scope: Who Else Cares?

* **Shared**: multiple apps reference the same DB URL → Spring Cloud Config
* **App-specific**: this app's own thread pool → ConfigMap or application.properties
* **Org-wide defaults**: standard timeouts, common endpoints → Config Server

Notes:
- Scope determines whether one team or many teams own the value.
- Shared config that lives in multiple places is a consistency problem waiting to happen.
