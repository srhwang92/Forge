# Decisions — [PROJECT NAME]

> Domain-organized reasoning memory. NOT chronological — grouped by what
> part of the system they govern. Each decision captures: what was chosen,
> what was rejected and why, what assumptions must hold, and how to verify.
>
> Updated when non-trivial decisions are made, not at session end.
>
> **Superseded decisions use a visually distinct format** to prevent
> skim-reading confusion. They start with `~~[SUPERSEDED]~~` (visible
> strikethrough in rendered markdown) and are followed by a blank line
> then `> **DO NOT IMPLEMENT — superseded by [newer-decision]**` as a
> blockquote warning. During Tier 2 reads, skip superseded entries.
> Human readers skimming the file must never mistake a superseded
> decision for current guidance.
>
> Domain knowledge (terminology, business rules, regulatory constraints)
> goes at the top of each domain section.
>
> **Standard domains** (use these names for consistency; add project-
> specific domains as needed): Auth, Data Model, API, Payments/Billing,
> UI/UX, Infrastructure, Testing, Performance, Security, Integrations.
> Using consistent names ensures Tier 2 reads find the right section
> after compaction.

---

<!-- 
## [Domain Name]

### Domain Knowledge
- [Key terms, business rules, regulatory constraints]
- [Values that must be remembered across sessions]

### Decisions

**[Decision title]**
- **Rejected:** [what was considered and why it lost]
- **Assumptions:** [what must be true for this to be correct — falsifiable]
- **Verify:** [runnable command or test, not a prose claim]
- **Date:** [YYYY-MM-DD]

~~**[SUPERSEDED]** Old decision title~~

> **DO NOT IMPLEMENT — superseded by [newer-decision-title].**
> Preserved for history only. If you are implementing based on this
> entry, stop and read the current decision instead.

- [original content preserved for history]
- **Date:** [YYYY-MM-DD]
-->
