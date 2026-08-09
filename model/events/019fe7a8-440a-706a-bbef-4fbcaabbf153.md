# `SourceRouteStrippedFromReturnPath`

An explicit source route in the return path was reduced to its final hop before a failure notification was addressed to it.

| Grouping | Value | Fixed at |
|:--|:--|:--|
| Workflow | 5. Responsibility Handoff and Final Delivery | Step 1 — timeline discovery |
| Stream root | Message | Step 1 — stream roots |
| Ownership swimlane | — | Step 6 — events section, per team or system |
| Originating actor | Receiving SMTP server | Step 3 — actor swimlane; provenance, not this event's row |

🟧 Event · confidence *certain* · recorded 2026-08-09, Step 1 Brain Storming, whole-spec sweep of RFC 5321 · `019fe7a8-440a-706a-bbef-4fbcaabbf153`

## Data

| Field | Type | Cardinality | Example | Example source |
|:--|:--|:--|:--|:--|
| `original_return_path` | ReversePath | Single | @a,@b:user@d | rfc5321.txt:3994 |
| `stripped_return_path` | ReversePath | Single | user@d | rfc5321.txt:3998 |
| `stripping_host` | Domain | Single | bar.com | walk scene |

## Where RFC 5321 says so

| Section | Lines | Text |
|:--|:--|:--|
| §6.1 | 3988-3989 | *"If the address is an explicit source route, it MUST be stripped down to its final hop."* |
| §6.1 | 3994 | *"MAIL FROM:\<@a,@b:user@d>"* |
| §6.1 | 3998 | *"RCPT TO:\<user@d>"* |
| §6.1 | 3987-3989 | *"If the address is an explicit source route, it MUST be stripped down to its final hop."* |
| §6.1 | 3996 | *"The notification message MUST be sent using:"* |

Line numbers are into [`docs/rfc/rfc5321.txt`](../../docs/rfc/rfc5321.txt), the archived copy. Quotations keep the specification's own words and spellings (AGENTS.md rule 2).

## Notes

Inherited candidate 276, kept. MUST, and the RFC supplies its own worked example, so the field values above are the RFC's verbatim and are deliberately NOT restyled into this project's foo.com/bar.com set — rule 2 governs, they are quoted evidence. Source routes are deprecated in §3.6.1 yet must still be accepted, and this stripping is a large part of why. The stripping applies to the notification's recipient, not to the stored message — a distinction easy to lose, and the reason this is a separate event from any envelope rewrite. Cross-reference: SourceRouteStripped (inherited candidates 71 and 170) covers the forward-path case during the transaction and belongs to the envelope chapter; a return-path strip for bounce addressing is not a forward-path strip for routing. Screens: the HELO-only charter and the twelve-slice model contain no bounce path at all. ⚠️ FIELD HYGIENE PASS 2026-08-09 — NOTHING STRUCK. This is the only event in the chapter that arrived clean, and the reason is exactly the one rule 8 states: it was walked with real data. Both path values are the RFC's own worked example, verified this pass at lines 3994 and 3998, so both now carry line-number example_source and neither could have been fabricated. It is the control case for the whole audit — where a sweep quoted the source instead of minting a value, no key had to be invented, because the values themselves are the identity.

COLLISION RESOLVED — merged from 2 independently brainstormed records. One MUST, one actor, one worked example, same section; the copies differ only in which of the two printed values they kept and in the stream root. Notification wins the root because the copy that rooted at Message argues for Notification in its own notes: 'The stripping applies to the notification's recipient, not to the stored message — a distinction easy to lose, and the reason this is a separate event from any envelope rewrite.' That is exactly the distinction that makes this a separate event, so the root should carry it.

From the duplicate in 12. Non-Delivery Notification: Inherited from raw candidate SourceRouteStrippedFromReturnPath, kept intact. MUST, and the RFC supplies its own worked example, so the field values above are the RFC's verbatim (@a,@b:user@d and user@d) and are kept unchanged under project rule 2 rather than restyled into the foo.com/bar.com set — they are quoted evidence, not illustration. Note what is being stripped: the RETURN path, for the purpose of addressing the notification. That is a different act from stripping a source route out of a FORWARD path on receipt (raw candidates SourceRouteStripped, §4.1.1.3 1925-1928, and SourceRouteIgnored, §3.3 1080-1082), which belongs to the envelope chapter — cross-referenced, not merged, because the direction and the purpose differ. Source routes are deprecated in §3.6.1 (heading verified at line 1421) yet must still be accepted, and this stripping is much of why. Adjacent rule outside this chapter's anchors and NOT merged: Appendix C tells a server needing to return a message to ignore the source route and use the domain from the Mailbox — the same outcome for @a,@b:user@d by a different route, recorded here so a reader does not take the two rules for a contradiction. Rejected alternative names: ReturnPathReducedToFinalHop, NotificationRecipientDerivedFromSourceRoute (rejected — 'derived' invites a computed field). Which screen would have killed it: the HELO-only charter and the twelve-slice model, which contain no bounce path whatsoever.

=== FIELD HYGIENE PASS === Struck two of three fields. (1) notification_ref (f2C8D14-ndn1) — fabricated, as everywhere in this chapter; what carries identity here instead is the return path itself, which is the only thing the RFC's own worked example names. (2) stripped_return_path — struck as a COMPUTED value under the method's own rule: §6.1 3987-3989 makes it a deterministic function of original_return_path ("MUST be stripped down to its final hop"), and the same value already appears as notification_recipient on NonDeliveryNotificationSent, so keeping it here both duplicates a fact across two events and puts a fold inside one. Nothing is lost — the RFC keeps both sides visible at rfc5321.txt:3994 and 3998, and those lines are cited above, so a facilitator who wants the output back can restore it from the source. The rejected alternative name already warned about this: 'derived' invites a computed field. The surviving example gains the angle brackets the RFC prints, since in SMTP the brackets are path syntax and MAIL FROM: is not MAIL FROM:\<>; the value stays the RFC's own under project rule 2 rather than being restyled into the walk scene's addresses.
