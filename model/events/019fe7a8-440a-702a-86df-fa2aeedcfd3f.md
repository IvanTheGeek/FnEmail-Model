# `TraceSourceNameStampedWithoutConnectionAddress`

A trace line's FROM clause carried only the name the client claimed, so nothing in the message records the address the server actually saw the connection come from.

| Grouping | Value | Fixed at |
|:--|:--|:--|
| Workflow | 4. Trace and Provenance | Step 1 — timeline discovery |
| Stream root | Message | Step 1 — stream roots |
| Ownership swimlane | — | Step 6 — events section, per team or system |
| Originating actor | Receiving SMTP server | Step 3 — actor swimlane; provenance, not this event's row |

🟧 Event · confidence *likely* · recorded 2026-08-09, Step 1 Brain Storming, whole-spec sweep of RFC 5321 · `019fe7a8-440a-702a-86df-fa2aeedcfd3f`

## Data

| Field | Type | Cardinality | Example | Example source |
|:--|:--|:--|:--|:--|
| `claimed_source_name` | Domain | Single | bar.com | walk scene |
| `observed_source_address` | IPv4Address | Single | 203.0.113.20 | walk scene |

## Where RFC 5321 says so

| Section | Lines | Text |
|:--|:--|:--|
| §4.4 | 3165-3168 | *"The FROM clause, which MUST be supplied in an SMTP environment, SHOULD contain both (1) the name of the source host as presented in the EHLO command and (2) an address literal containing the IP address of the source, determined from the TCP connection."* |
| §4.4 | 3321-3322 | *"Extended-Domain = Domain /"* |

Line numbers are into [`docs/rfc/rfc5321.txt`](../../docs/rfc/rfc5321.txt), the archived copy. Quotations keep the specification's own words and spellings (AGENTS.md rule 2).

## Notes

RECOVERED BY THE RE-SWEEP. This is the SHOULD's other arm and it is *conforming* — Extended-Domain's first alternative at 3321 is a bare Domain with no TCP-info at all, so the grammar explicitly permits what the prose discourages. That makes it the most interesting kind of violated arm: not misbehavior, just a weaker record. The RFC's own worked example does exactly this — D.3 line 5064 reads 'Received: from bar.com by foo.com' with no address literal — so the spec ships a non-recommended trace line in its own appendix. Real data matters here per rule 8: instantiating observed_source_address with 203.0.113.20 makes visible that the fact existed and was discarded, which a placeholder hides entirely. Sibling: FromClauseComposed (the recommended form). Opposite: FromClauseOmittedFromTrace. FnEmail screens: the consumer test / G1 — nothing inbound reads a trace line the server itself wrote. ⚠️ FIELD HYGIENE PASS: struck the fabricated `message_ref` identity key — the spec names no message identifier, and f2C8D14 is not in the RFC. Identity is the session position: the server that observed 203.0.113.20 is the server writing the line, in the session that address opened, so the channel carries the identity. The rule-8 argument above survives intact and is now the event's whole payload — the discarded observation is the finding.
