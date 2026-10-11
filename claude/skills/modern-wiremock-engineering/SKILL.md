---
name: modern-wiremock-engineering
description: "Automatically enforces modern WireMock engineering standards: declarative stubbing (mappings/ vs __files/), response body separation, Handlebars dynamic templating, deterministic priority scoring, stateful scenarios, official WireMock container orchestration, and mandatory dedicated README.md living documentation."
---

# Modern WireMock Engineering Guardrails & Skill Router

Entry point for this plugin's guardrails in Claude Code. Before acting, read the full standard:

`${CLAUDE_PLUGIN_ROOT}/rules/modern-wiremock-engineering.md`

It routes each kind of task to the specialized skill of this plugin and lists the non-negotiable rules.
In Claude Code those skills are namespaced: invoke `modern-wiremock-best-practices:<skill-name>`.

> Generated from `rules/modern-wiremock-engineering.md`, which is also the Antigravity rule. Edit the rule, not this file.
