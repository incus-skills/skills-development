---
name: diagram-policy
description: Authors, reads, extracts, reviews, and validates Mermaid diagrams against the canonical diagram policy. Use when creating, editing, reviewing, or explaining diagrams in project artifacts.
version: "1.1.0"
metadata:
  version: "1.1.0"
  policy_revision: "2.1"
---

# Diagram Policy Skill

## Quick procedure

1. Identify the diagram concern and authoritative source.
2. Establish the intended structure independently of the diagram being checked.
3. Author, read, or extract the Mermaid source without silently changing meaning.
4. Validate syntax and semantics separately.
5. Report evidence, differences, limitations, and unresolved questions.

Do not skip step 2 just because a diagram already exists.

## 1. Purpose

Apply the canonical requirements in [`POLICY.md`](POLICY.md) when authoring, reading, extracting, reviewing, or validating diagrams. This skill is procedural; the policy is normative.

**Policy compatibility gate:** this skill declares policy revision `2.0`. Before use, inspect the governing policy and verify its revision. If the policy is missing or unreadable, stop the affected operation and report compatibility as unverified. If the skill is behind the policy, stop until updated. If the skill declares a later revision than the available policy, stop and verify the authoritative policy distribution. If policy and skill conflict, identify the conflicting provisions and impact, then stop the affected operation until an authorized resolution is recorded. Do not treat missing policy as absence of constraints.

## 2. Establish intended structure

Find the source that specifies what the diagram is supposed to mean: a specification, source document, explicit decision, protocol, or other authoritative reference. Record its path/identifier and immutable revision where available.

Create or consult an intended-structure reference with:

- stable reference ID, concern, source location/revision, capture time, and status (`Draft`, `Approved`, or `Unresolved`);
- required nodes with IDs, intended labels, and roles;
- required edges with source, target, type, labels, conditions, and outcomes;
- required cycles, boundaries, terminal states, and invariants;
- assumptions, competing interpretations, and unresolved questions.

Keep intended requirements separate from observations of the current diagram. Do not infer the intended structure solely from the artifact being validated. If the reference is incomplete, mark it Draft or Unresolved and limit the strength of the acceptance claim accordingly.

If no structured baseline exists, build the smallest reference that the authoritative source supports. If the source is unstable, record its available identity, manual version or section, capture time, and checksum where practical. State the limitation: a checksum establishes identity, not semantic intent.

## 3. Authoring procedure

1. Define the concern and intended audience. Apply the single-concern rule, checking the canonical policy for scoped exceptions before splitting a graph.
2. List the required nodes, relationships, decisions, conditions, cycles, boundaries, terminal states, and invariants before writing Mermaid.
3. Use Mermaid source only. Prefer clear node IDs and labels that preserve domain terms.
4. Use `{"Decision?"}` for flowchart decision nodes and label every outcome edge.
5. Represent cycles and feedback as explicit edges.
6. Use edge styles consistently: `-->` transition/flow; `-.-` observational/relational; `-.->` directional observational. Label relationships where style alone would be ambiguous.
7. Use labelled `subgraph` blocks for meaningful phases, scopes, and boundaries.
8. Apply only semantic styling: red fill for error paths, grey fill for terminal states, green fill for primary output. Essential meaning must remain available without color.
9. Keep one concern per diagram, subject to the canonical policy’s scoped exceptions. Review diagrams exceeding approximately 15 nodes for splitting, but do not split a normative routing graph whose complete task-to-action path is the unit of meaning. Keep that path together; exclude unrelated concerns and validate the full path semantically.
10. Preserve the Mermaid source as canonical; any rendered image is derivative.

Do not add a relationship because it makes the diagram visually smoother. Do not remove an awkward relationship because it makes the diagram simpler.

## 4. Validation procedure

Perform and report the following separately.

### Syntax validation

- Check Mermaid syntax using an available parser or renderer.
- Record the tool or method used and whether it succeeded.
- Do not treat successful rendering as semantic acceptance.

### Semantic validation

Compare the diagram with the independent intended-structure reference:

- required and unexpected nodes;
- required and unexpected edges, endpoints, direction, type, and labels;
- decision branches and their explicit outcomes;
- cycles and feedback paths;
- subgraph boundaries and grouping constraints;
- terminal states, invariants, and error paths;
- semantic changes introduced since the reference revision.

Classify each difference as equivalent representation, non-semantic formatting change, authorized semantic change, unauthorized semantic change, or unresolved. Do not silently correct the diagram while validating it.

### Acceptance report

Report:
- overall result;
- syntax result and method;
- semantic result and baseline/reference ID, revision, status, and location;
- detected differences and their disposition;
- authorization for any semantic change;
- unresolved questions, limitations, and checks not performed;
- source revision and evidence needed to reproduce the check.

If there is no independent reference, report semantic validation as incomplete. If the reference is Draft, identify the result as provisional. Do not claim an execution or test that was not actually performed.

## 5. Read and extraction procedure

When interpreting or extracting an existing diagram:

1. Read the Mermaid source rather than relying only on a rendered image.
2. Extract node IDs and labels, edge endpoints and directions, edge labels/types, decision outcomes, cycles, subgraphs, and explicit terminal states.
3. Separate facts explicit in source from inferred meaning.
4. Preserve ambiguous, conflicting, or malformed elements in the report rather than silently repairing them.
5. Compare extracted structure with the authoritative source where one exists.
6. State what cannot be determined from the source alone.

Mermaid improves extractability but does not guarantee that the diagram is semantically correct or independently interpretable.

## 6. Correction procedure

When a defect is found:

1. Describe the discrepancy against the reference.
2. Identify whether it is syntactic, semantic, or both.
3. State the intended correction and its evidence.
4. If the correction changes meaning, obtain explicit authorization or record the authorized decision before applying it.
5. Make the smallest correction that preserves all other required relationships.
6. Re-run syntax and semantic checks.
7. Record the new source revision, outcome, and remaining uncertainty.

If policy and skill disagree, do not resolve the conflict by choosing whichever text is easier to follow. Apply the compatibility gate and stop the affected operation until resolution is recorded.

## 7. Regression tests and status

The single-concern rule includes a scoped exception for normative routing graphs that must preserve the complete task-to-action path. When authoring or reviewing such a graph, verify that the graph contains only that path, that required relationship semantics are interpreted in the context of the whole path, and that the complete graph is validated against an independent intended-structure reference. The exception does not waive any other policy requirement.

The initial regression cases are:

- A — SPEC architecture;
- B — RSS loop.

Each needs an independently established intended-structure reference and an actual comparison against the authoritative source. They are `candidate / unvalidated` until that work is executed and recorded. Cases C–E (policy architecture, validation/conflict procedure, lifecycle/governance procedure) are further candidates, not evidence of completed tests.

Skill registration means the skill is present and discoverable. Validation means defined checks have actually been run and their results recorded. Until the relevant checks are executed, report status as `registered, unvalidated`.

## 8. Completion criteria

A diagram task is complete only to the level claimed in its report. For a full semantic acceptance claim, there must be an independent intended-structure reference, a comparison of actual source to that reference, recorded syntax and semantic results, and explicit treatment of differences and uncertainty.

Preserve enough rationale and provenance that a later contributor can reconstruct why the diagram has its current structure and what evidence would justify changing it.
