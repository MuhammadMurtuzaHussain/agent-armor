# AgentArmor

A hardening checklist for LLM agents — tool-calling, RAG, MCP and memory-backed — against prompt injection, data exfiltration and tool abuse.

## What it covers

- The core principle: never let the model police its own untrusted input
- A control checklist mapping seven attack classes (tool-description poisoning, memory poisoning, indirect document injection, planted-record framing, system-prompt-as-permission attacks, identity spoofing, system-prompt extraction) to concrete defences
- Cross-cutting controls: instruction hierarchy, egress allow-lists, DLP on tool arguments, schema constraints, server-side identity, memory write gating, human-in-the-loop approval
- A red-team review procedure and OWASP LLM Top 10 / NIS2 / ISO 27001 mapping

**Companion skill:** [AgentBreaker](https://github.com/REPLACE_ME/agent-breaker) — the offensive playbook for the same attack classes.

## Install

As a Claude Skill:

```bash
git clone https://github.com/REPLACE_ME/agent-armor ~/.claude/skills/agent-armor
```
# agent-armor
