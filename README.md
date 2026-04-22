# Taming Configuration

**A Practical Guide to Externalizing Spring Boot Config with Spring Cloud Config, Kubernetes, and HashiCorp Vault**

A 50-minute conference presentation by [DaShaun Carter](https://dashaun.com) | [@dashaun](https://twitter.com/dashaun)

## View the Slides

Requires Java 25+ (see `.sdkmanrc`). From the repo root:

```bash
sdk env
jwebserver -p 8000 -b 0.0.0.0 -d docs
```

Open [http://localhost:8000](http://localhost:8000).

| Key | Action |
|---|---|
| `→` / `Space` | Next slide |
| `↓` | Next vertical slide |
| `S` | Speaker notes |
| `F` | Fullscreen |
| `O` | Slide overview |

## Talk Outline

| Section | Topic | Time |
|---|---|---|
| 1 | Introduction & Framing | ~5 min |
| 2 | The Configuration Spectrum | ~10 min |
| 3 | The "Should I Externalize It?" Decision Tree | ~10 min |
| 4 | Where to Put It: The Four-Tier Layering Model | ~15 min |
| 5 | The etcd Size Problem — A Diagnostic Lens | ~10 min |
| 6 | Patterns & Anti-Patterns + Decision Flowchart | ~5 min |
| Q&A | Open Discussion | ~5 min |

## Key Takeaways

1. Classify config by **volatility × scope** before deciding where it lives
2. **Sensitivity overrides everything** — secrets go to Vault, no exceptions
3. Four tiers: Vault → Spring Cloud Config → Kubernetes ConfigMap → `application.properties`
4. The **ConfigMap lean rule**: if Kubernetes doesn't need to know about it directly, it doesn't belong in a ConfigMap
5. The 4 GB etcd limit is a config hygiene symptom, not a storage problem

## Built With

- [RevealJS](https://revealjs.com) — presentation framework
- [Spring brand fonts](https://fonts.google.com) — Work Sans + Open Sans
