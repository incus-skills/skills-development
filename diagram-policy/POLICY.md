# Canonical Diagram Policy

**Revision:** 2.0  
**Status:** Initial published baseline  
**Authority:** This document is normative for diagram work in this repository and for consumers that explicitly adopt it.

## 1. Purpose and scope

This policy governs diagrams in project artifacts, README files, methodology documents, skill documentation, and relevant derivatives. Its purpose is not merely to standardize appearance: diagrams must preserve intended meaning, explicit relationships, and enough source information to be interpreted independently.

## 2. Normative rules

1. **Mermaid only.** Diagrams must use Mermaid markup. ASCII art, Unicode box drawing, and indented pseudo-diagrams are prohibited. If a structure cannot be adequately expressed in Mermaid, use a table and explain the limitation in prose.
2. **Explicit decisions.** Decision nodes use Mermaid flowchart decision syntax, for example `{"Validation passed?"}`, with explicitly labelled outcome edges.
3. **Explicit cycles.** Represent loops and feedback as explicit edges. Do not turn a cycle into a terminal state or a one-way path when doing so changes meaning.
4. **Label edge semantics.** Edge labels carry meaningful conditions, outcomes, or qualifications. Do not leave a decision outcome implicit.
5. **Consistent edge meaning.** Use:
   - `-->` for flow or transition;
   - `-.-` for observational or relational connections;
   - `-.->` for directional observational connections.
   
   Use these distinctions consistently within a diagram. Do not rely on line style alone where the relationship's meaning would otherwise be ambiguous; label it when needed.
6. **Node-local properties.** Put a node's properties in its node label. Put extended commentary and interpretation in surrounding prose.
7. **Named boundaries.** Use labelled Mermaid `subgraph` blocks to group phases, scopes, or boundaries when those groupings are part of the intended structure.
8. **Semantic styling only.** Styling must encode meaning, not decoration. Red fill is reserved for error paths, grey fill for terminal states, and green fill for primary output. Do not add aesthetic styling. Where color could be unavailable or ambiguous, labels and graph structure must still convey the essential meaning.
9. **One concern per diagram.** Each diagram should address one concern. Diagrams with more than approximately 15 nodes must be reviewed for possible splitting. Split only when doing so preserves the relationships needed to understand each concern. For example, a SPEC architecture diagram and an RSS loop diagram are distinct concerns and must be separate.
10. **Source is canonical.** Mermaid source is the canonical diagram artifact. Rendered PNG/SVG images do not replace or supersede the source.
11. **Validate semantics, not just syntax.** Successful parsing or rendering is not proof of correctness. Compare nodes, edges, direction, labels, outcomes, cycles, boundaries, and invariants against an independently established intended structure.
12. **Preserve meaning across edits.** Preserve explicit graph relationships and intended meaning. Do not silently repair, simplify, rename, merge, reorder, or remove structures if that changes meaning. Proposed semantic changes require explicit authorization and a recorded rationale.
13. **Make the source independently interpretable.** Do not rely on color, layout, or surrounding prose alone to convey essential relationships. Mermaid improves extractability but does not guarantee correct interpretation.

## 3. Authority and conflict handling

This policy is the canonical authority for the diagram-policy skill.

If the skill and policy conflict:
1. identify the exact provisions in conflict;
2. describe the affected operation and likely impact;
3. halt the affected operation;
4. record the conflict and resume only after an authorized resolution or clarification is recorded.

Unrelated work that does not depend on the conflict may continue. A missing or unreadable policy is not permission to proceed without constraints; compatibility is unverified and the affected operation must stop.

## 4. Intended-structure reference

Semantic validation requires an intended-structure reference established independently of the diagram under test. Do not derive intent solely from the diagram being validated.

Record, proportionate to the diagram's risk and complexity:

- **Identity:** stable reference ID, diagram concern, authoritative source path/section/specification ID, source revision (prefer commit SHA or immutable revision), capture time, and status (`Draft`, `Approved`, or `Unresolved`).
- **Nodes:** stable node ID, intended label, role, and whether the node is required.
- **Edges:** source, target, relationship type (transition, observational/relational, or directional observational), intended label/condition, and whether the edge is required.
- **Constraints:** required cycles, decision outcomes, boundaries/subgraphs, terminal states, invariants, and other semantic constraints.
- **Uncertainty:** ambiguities, competing interpretations, assumptions, and unresolved questions.
- **Provenance:** who or what established the reference, when, and from which source revision.

Distinguish source requirements from observations of the current diagram. A reference may be Draft; provisional checks may proceed if useful, but the report must identify it as provisional. Ambiguity must remain unresolved until evidence or an authorized decision resolves it.

If no stable source history exists, record the available source identity, manual version or section, capture time, and checksum where practical, together with the reproducibility limitation. A hash proves content identity, not semantic intent.

## 5. Validation and acceptance report

Report syntax and semantic results separately. An acceptance report should include:

- overall result and scope;
- syntax status and how it was checked;
- semantic status and which intended-structure reference was used;
- reference ID, revision, status, and authoritative location;
- missing, extra, renamed, redirected, or relabelled nodes and edges;
- cycle, decision-outcome, boundary, terminal-state, and invariant checks;
- differences from the reference and whether each semantic change was authorized;
- unresolved ambiguity, limitations, and checks not performed;
- source revision and enough evidence to repeat the validation.

Do not claim full semantic validation if no independent intended-structure reference exists. Do not claim operational validation unless the relevant procedure was actually executed.

## 6. Initial regression cases

Two independent regression cases are designated:

- **A — SPEC architecture**
- **B — RSS loop**

Establish an independent intended-structure reference for each before validating the corresponding diagram. The actual authoritative source, revision, reference, execution evidence, and result must be recorded. Mentioning a case is not evidence that it has been tested. Until executed and recorded, each case is `candidate / unvalidated`.

Further self-application candidates may include:
- **C — Policy architecture**
- **D — Validation and conflict procedure**
- **E — Lifecycle and governance procedure**

These are candidate tests, not existing or validated diagrams. Self-application must be supplemented by independent use cases.

## 7. Revision history

| Revision | Summary |
|---|---|
| 2.0 | Formalized the 13-rule policy; added semantic validation, meaning preservation, independent interpretability, intended-structure references, and explicit authority/conflict handling. |
| 1.0 | Initial 10-rule version. |
