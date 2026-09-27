---
name: ai-receptionist-tenant-onboarding
description: Onboarding pattern for multi-tenant inbound AI phone agents. Covers per-tenant routing, prompt/voice config, calendar or CRM integration, LLM backend selection per tenant, and the acceptance test a tenant has to pass before going live. Use when adding a new client tenant to a multi-tenant phone agent system, cloning the demo configuration for a real customer, diagnosing why a tenant's calls aren't routing correctly, or reviewing whether a tenant is actually ready to take production calls.
---

# AI Receptionist Tenant Onboarding

A multi-tenant inbound AI phone agent has a sharp asymmetry: the demo number takes one call at a time, the tenant number takes the client's actual business. The onboarding process is what separates a clever demo from a system a real business trusts to answer its phone.

This skill describes the pattern for adding a tenant correctly. It is implementation-agnostic. The components named here exist in any sane multi-tenant phone agent, even if the exact file paths differ between stacks.

## The five components of a tenant

Every tenant has exactly five pieces of configuration. If any one is missing, the tenant is not ready.

1. **Phone routing.** A dedicated inbound number that routes to this tenant's configuration, not the shared default. The number is the tenant's identity to the system; all routing decisions key off it.
2. **Prompt and persona config.** The system prompt, greeting line, voice selection, and any tenant-specific knowledge (business hours, service list, pricing guidance, escalation triggers). This is versioned; it is not hot-edited in production.
3. **Integration target.** Where bookings / leads / messages land. Google Calendar, a CRM, an email inbox, a webhook to the tenant's own system. The integration is tenant-owned; the agent writes, the tenant reads.
4. **LLM backend selection.** Which model serves this tenant's calls. This is per-tenant, not global. Internal tenants (your own business's) can run on local vLLM if you host one. Client tenants run on whatever the client is paying for, usually hosted. The choice is documented in the tenant config, not implicit in the codebase.
5. **Acceptance test.** A documented call script the tenant has run themselves, end-to-end, with the tenant confirming the booking / lead / message landed where it was supposed to. Not "we tested it"; the tenant ran the test.

## The demo number is not a tenant

Critical rule: the public demo number exists to prove the system works. It is not a fallback for a tenant whose number isn't provisioned yet, it is not a test environment for prompt iteration on a live tenant's config, and it is not shared with any paying customer.

New tenants get their own number. Always. If a tenant's number isn't provisioned yet, the tenant is not onboarded yet. Don't route their calls through the demo.

## Onboarding order of operations

Follow this order. Skipping steps has caused every tenant-onboarding failure that has ever happened on this kind of system.

1. **Deposit clears.** Per `consulting-sales-playbook`. No tenant goes live before the deposit is in the bank. "But they need it working for a Monday demo" is not a reason. If the deposit isn't cleared by Friday, the demo isn't Monday.
2. **Provision the tenant's phone number.** Inside the phone provider, not an alias. Map the number to a stub tenant config in the system. Confirm the number rings and the stub answers before proceeding.
3. **Write the prompt config.** In a versioned file, reviewed against the tenant's actual business: their hours, their services, their voice preferences. Copy from a previous tenant as a starting template if one fits; do not start blank.
4. **Wire the integration target.** Calendar account, CRM API key, webhook URL, whichever applies. Test the write path with a synthetic event before live calls. A calendar that "looks connected" is not connected.
5. **Choose and configure the LLM backend.** Set the tenant's model in their config. Verify it with a test call that exercises the model (ask a question that requires the model to answer, not just read a script).
6. **Run the acceptance test with the tenant.** The tenant calls their own number from their own phone. The agent handles the call. The result lands in the tenant's integration target. The tenant confirms they see it. This is the ship gate.
7. **Flip the tenant to live.** Marked live in the system. Dashboard shows the tenant. Deposit is noted as applied to this tenant's setup cost, not still floating in escrow.

If any step fails, stop and fix it before moving on. Do not move forward with a known-broken step and plan to come back.

## Per-tenant LLM swap

The system supports per-tenant model selection. Use it deliberately, not reflexively.

- **Your own internal tenant can run on local vLLM** (e.g. a Qwen-class model on unified-memory hardware) if you host an endpoint. That's the point of owning the hardware: internal calls are free and private. See `local-llm-ops` for the operational rules. No local host → run internal tenants hosted like everyone else.
- **Client tenants usually run on hosted.** Reliability, SLA, and "if it breaks, there's a vendor to call" all favor hosted for paying customers. A client tenant on local inference is a support liability unless the client has explicitly signed off on the tradeoffs in writing.
- **A tenant switching backends is a re-acceptance-test event.** The voice sounds different. The latency profile is different. The tool-calling behavior is different. If a tenant moves from hosted to local or vice versa, re-run the acceptance test with them. Do not swap silently.

## Common failure modes

These are the recurring failures this skill exists to prevent. If any of these show up during onboarding, stop and fix before going live.

- **Calls routing but no response.** The number is provisioned and hits the system, but the tenant config isn't finished. Calls get a stub greeting or dead air. Fix: finish step 3 before letting step 2's number go live.
- **Booking confirmed to caller, nothing in the calendar.** The integration is misconfigured. The agent thinks it wrote to the calendar; the calendar disagrees. Fix: synthetic write test in step 4, never skip.
- **Tenant voice doesn't match their brand.** Default voice was never changed. Step 3 was skipped or done with a stale template. Fix: voice selection is part of the prompt config checklist, not an afterthought.
- **Model loaded is not the model configured.** Per `local-llm-ops`: verify with the `/v1/models` endpoint. An older cached artifact can win silently.
- **Tenant's "acceptance test" was run by the engineer, not the tenant.** This is the single most common failure. The engineer tests happy path and everything looks fine; the tenant calls and the experience is off. Fix: the acceptance test is not complete until the tenant ran it themselves.

## When this skill should say no

- **Before routing a new tenant through the demo number "temporarily."** Stop. Provision the dedicated number first or delay the launch.
- **Before going live without a tenant-run acceptance test.** Stop. The tenant calls or it isn't live.
- **Before swapping a tenant's LLM backend mid-flight without re-testing.** Stop. Run the acceptance test again.
- **Before hot-editing a production tenant's prompt config.** Stop. Version the change, test it, deploy.
- **Before going live without the deposit cleared.** Stop. The tenant isn't a tenant until they've paid.

## The tenant is ready when

The tenant is ready when:

- The tenant has called their own number.
- The agent handled the call in the tenant's voice, with the tenant's knowledge.
- The tenant's integration received the result.
- The tenant has confirmed they saw it.
- The deposit is applied, not pending.

Not before.
