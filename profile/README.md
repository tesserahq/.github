<p align="center">
  <img width="100px" src="../assets/logo.png">
</p>


# Tessera

## 🧩 What is Tessera?

**Tessera** is a modular developer framework designed to help you build modern products faster.

It brings together a collection of Python-based services — like AI orchestration, user identity, file storage, notifications, and RBAC — into a composable, scalable foundation for any digital product. Each service is independent yet seamlessly interoperable, empowering teams to focus on delivering value instead of reinventing core infrastructure.

Tessera gives you:
- ⚡ **Speed** — Build and ship features faster by using ready-to-go components.
- 🧱 **Composability** — Plug in only what you need: identity, files, emails, AI, and more.
- 🔐 **Security-first design** — Auth, RBAC, and secure storage baked in.
- 🧠 **Smart defaults** — Optimized patterns and best practices built from experience.
- 💬 **Extensibility** — Easily integrate custom workflows, AI agents, and third-party tools.

It powers services like:
- `quore` – AI orchestration and reasoning
- `identies` – user identity and auth
- `vaulta` – file storage and signing
- `sendly` – notifications and events
- `custos` – RBAC and authorization
- `looply` – Mailing lists and contacts
- `orcha` – Workflows

## 🧩 Why the name “Tessera”?

A **tessera** is a small tile used in mosaics — simple on its own, but when combined with others, it forms something beautiful and powerful.

The same is true for this framework: each service (identity, storage, AI, etc.) is a tile — a **tessera** — and together they compose the foundation of your product.

This name reflects the philosophy behind the framework:
- **Modularity**: each service stands on its own.
- **Harmony**: together, they create a cohesive system.
- **Craftsmanship**: like a mosaic, it’s designed with care and precision.

## 🧩 Intentional Modularity and Service Boundaries

Tessera’s architecture — many small services, each responsible for exactly one thing — is intentional.

This framework started as an AI-first experiment built by a very small team: one backend engineer and one frontend engineer. Early on, we discovered a critical constraint when working with AI models: smaller, well-defined contexts produce dramatically better results than large, monolithic codebases. Splitting responsibilities into focused services allowed AI tools to reason more accurately, generate safer changes, and operate with far less ambiguity.

As a result:
* Each service does one thing, and only one thing
* Each service has its own management portal
* Each codebase stays small, explicit, and easy to reason about

This structure makes the system easier to evolve over time. Improving or replacing a service rarely requires understanding the entire platform. It also significantly lowers the barrier to onboarding new contributors, since there is no massive shared context to absorb before being productive.

Trade-offs and Why They’re Acceptable

Yes, this approach introduces overhead:

* More repositories
* More services
* More web apps and deployments

This is a real cost, and it’s acknowledged.

However, modern AI tooling changes the equation. With tools like Claude Code and similar agents, it is now practical to perform coordinated changes across multiple repositories using a single prompt. To go further, we built our own MCP (Model Context Protocol) to orchestrate multi-service workflows and reduce friction when working across the ecosystem.

In practice, this means:

* Cross-repo refactors are fast
* Repetitive work is automated
* Cognitive load stays low, even as the system grows

Philosophy Summary

Tessera optimizes for:

* Small contexts over large abstractions
* Clear boundaries over convenience
* Long-term adaptability over short-term simplicity

The result is a system that scales with people, AI, and change — without collapsing under its own complexity.
