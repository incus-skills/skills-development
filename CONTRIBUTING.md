# Contributing

This repository develops reusable methods for authoring, extending, reviewing, and validating agent skills. Contributions should make the next safe change easier without obscuring the requirements or evidence that justify the current design.

## Before changing files

1. Read [SKILL_DEVELOPMENT.md](SKILL_DEVELOPMENT.md) and the relevant canonical policy.
2. Identify the problem and the authoritative source or observation that motivates the change.
3. Separate established requirements, observations, assumptions, and unresolved questions.
4. Check whether the change belongs in an existing skill, a new skill, a canonical policy, a runtime-specific adapter, or supporting documentation/tests.
5. Record the expected behaviour and the criteria that will establish correctness.

## Change discipline

- Prefer the smallest coherent change.
- Preserve explicit distinctions and relationships; do not silently simplify semantics.
- Keep policy (normative requirements) separate from procedures that apply it.
- Keep portable behaviour separate from runtime-specific installation details.
- Use Mermaid for diagrams, following [diagram-policy/POLICY.md](diagram-policy/POLICY.md).
- Preserve source documents and Mermaid source; rendered images are not substitutes for source.
- Do not describe candidate tests as passed unless they were actually run and their results recorded.
- Do not convert missing evidence into an assumption of correctness.

## Policy inclusion review

A new policy must provide enough information for a contributor to determine what it governs and whether it applies. At minimum, include:

- **Purpose and scope:** the subject matter and explicit applicability boundary; state what is not governed.
- **Authority and status:** revision, status, and whether the policy is proposed, approved, or otherwise provisional.
- **Provenance:** originating source, decisions, observations, or prior artifacts; distinguish supplied provenance from independently verified evidence.
- **Normative requirements:** unambiguous rules, with exceptions and precedence made explicit.
- **Rationale:** the properties the rules preserve and why the constraints exist.
- **Compatibility and conflict handling:** how adoption works, what happens when a higher-authority requirement conflicts, and when affected work must stop.
- **Validation:** checks appropriate to the policy's risk, including operational checks where a policy governs executable behaviour.
- **Maintenance:** revision history and conditions that justify change.

A policy may be domain-scoped. It must not be presented as universally binding merely because it is stored in this repository. The repository index must identify its scope and adoption status. Where examples expose an ambiguity or contradiction between rules, resolve it explicitly rather than leaving readers to infer precedence.

## Review checklist

- [ ] Purpose, activation conditions, and scope are clear.
- [ ] Authority and dependencies are identified.
- [ ] Requirements are distinguished from observations and assumptions.
- [ ] Conflicts, missing context, and stopping conditions are explicit.
- [ ] Validation criteria and evidence are recorded.
- [ ] Compatibility and revision declarations are coherent.
- [ ] Diagrams comply with the canonical policy and preserve intended meaning.
- [ ] Documentation does not overstate what was tested.
- [ ] Limitations, unresolved decisions, and next steps are visible.
- [ ] The handoff permits another contributor to reconstruct why the change was made.

## Validation status

Use precise status language:

- **Draft:** content is proposed and not yet approved as a governing baseline.
- **Registered:** a skill is present and discoverable.
- **Validated:** specified checks have been executed and the evidence/result recorded.
- **Provisional:** checks were performed against a draft or incomplete reference.
- **Unresolved:** evidence or an authorized decision is insufficient to settle a material question.

Registration is not validation. A passing syntax check is not semantic validation. A hash identifies content; it does not establish what that content ought to mean.

## Handoff

Summarize what changed and why, which source/policy revisions governed the change, what checks were run and their outcomes, what remains uncertain, and the next safe action. Where practical, include immutable commit links or identifiers rather than relying on a floating branch name.
