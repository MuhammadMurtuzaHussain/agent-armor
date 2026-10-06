---
name: agent-armor
description: Use when building or reviewing an LLM agent (tool-calling, RAG, MCP, or memory-backed) to harden it against prompt injection, data exfiltration and tool abuse: a control checklist mapping each attack class to a concrete defence, with OWASP LLM Top 10, NIS2 and ISO 27001 framing.
---

# AgentArmor: Hardening LLM Agents Against Prompt Injection

## When to use

When building or reviewing an LLM agent - tool-calling, RAG, MCP-connected, or carrying persistent memory - to make it resistant to prompt injection, data exfiltration, and tool abuse. Use it as a design checklist and as a red-team review guide. It is the defender's mirror of the attack techniques in the companion skill, AgentBreaker.

## The core principle

Any guardrail the model enforces by its own reading of untrusted text is bypassable. The attacker's instructions and the data the model processes arrive through the same channel, so the model cannot reliably tell "my operator said this" from "this is content I am processing". Move every security-relevant decision - identity, egress, data classification, memory writes, high-impact approval - to a layer **outside** the model that an attacker cannot argue with. A better sentence in the system prompt is not a control.

## Treat these as untrusted input, never as instructions

- Tool and MCP descriptions (third-party metadata is attacker-controllable text).
- Retrieved documents, uploaded files, and scraped web pages (RAG content).
- Tool results, and any user text that is *shaped like* a tool result or system message.
- Persistent memory entries, especially when the write path is attacker-reachable - a compromised log, or the assistant itself summarising poisoned content into its own memory.
- The model's own prior output.

## Checklist by attack class

| Attack | Control |
| --- | --- |
| Tool-description / MCP metadata poisoning | Treat tool metadata as untrusted data, not instructions; constrain at the schema level what each argument may carry, so a free-text field cannot hold an email address, a transcript, or PII at all |
| Memory poisoning | Store facts with provenance, not directives; strip format and behaviour instructions on write; require explicit confirmation before a memory entry can change how the assistant responds; give users visibility into and deletion rights over stored memory |
| Indirect injection via retrieved documents or pages | Treat retrieved content as data; strip or neutralise instruction-like and delimiter-like blocks before they reach the model; do not let apparent "user turns" inside retrieved content be trusted as real turns |
| Planted record / authority framing | Validate structured output against the source facts; do not let a labelled "record" in a document set the output fields |
| Security rules turned into permissions | Keep actionable permissions out of natural-language prose; enforce egress allow-lists and DLP on outbound tool calls, so no "policy update" text can widen them |
| Fake tool-output / identity spoof | Enforce sender and caller identity server-side in the tool; never trust user-supplied text shaped like a tool result |
| System-prompt extraction | Keep secrets out of the system prompt entirely; a "never reveal this" instruction is not a control |
| Output-constraint break | Normalise input before classifying; state the output rule explicitly; separate understanding the request from producing the forbidden content |

## Cross-cutting controls

- **Instruction hierarchy.** Give the model a trustworthy way to distinguish operator instructions from processed content (structured roles, signed/stamped system context), and assume any boundary expressed only as text in the context can be forged.
- **Egress allow-list.** Recipients and outbound destinations are checked against a hard list outside the model; log and alert on any call to a new domain.
- **DLP on tool arguments.** Scan outbound tool-call arguments (email bodies, API payloads) for sensitive data before the call is allowed to proceed.
- **Schema constraints.** Make the dangerous value unrepresentable: a recipient field as an allow-list enum, a notes field that cannot carry free text, output fields typed and bounded.
- **Server-side identity.** Sender address, caller identity, and tool permissions are set by the runtime, never decided by the model.
- **Memory write gating.** A write to persistent memory that changes future behaviour (a new rule, a format instruction, a standing preference) gets confirmed with the user before it takes effect, separately from an ordinary fact being filed.
- **Human-in-the-loop** for high-impact actions: outbound email containing PII, a financial recommendation, a dependency added to a codebase, a code merge.
- **Input normalisation** (Unicode fold, language handling) before any classifier runs.
- **Output handling.** Validate output against the source and against protected content; remember that a verbatim-only filter still leaks paraphrase, so filter on meaning where it matters.
- **Grounding.** Weight a consistent source body over short injected notes; validate recommendations against source facts.
- **Defence in depth.** No single layer should be asked to be perfect; a classifier plus an explicit output rule plus architectural separation absorbs attacks that any one layer would leak.

## How to red-team it

- Submit a plainly bad payload first, purely to harvest the agent's own refusal reasoning - a good refusal names what to defeat next.
- Exercise each injection channel: tool description, retrieved document (with delimiter-escape and a fake user turn), fake tool-output, memory write, direct override, system-prompt extraction.
- Test for non-determinism: run each payload several times; a filter that passes an attack one time in three is not a filter.
- Probe the output filter: does paraphrase leak what verbatim cannot?
- Confirm that identity, egress, classification, and memory-write decisions hold even when the prompt is told they changed.

## Governance mapping

OWASP Top 10 for LLM Applications (2025): LLM01 Prompt Injection, LLM02 Sensitive Information Disclosure, LLM03 Supply Chain, LLM05 Improper Output Handling, LLM06 Excessive Agency, LLM07 System Prompt Leakage, LLM09 Misinformation. For NIS2 and ISO 27001: extend supplier and third-party controls to the *natural-language content* of AI components (a plugin's tool description is attacker-controllable code); treat egress control and data-loss prevention as enforced technical controls, not model behaviour; and require human review for automated decisions that carry real-world impact.
