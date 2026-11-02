# Provisioning Shell Heredoc Policy

**Revision:** 1.0.0  
**Status:** Proposed baseline  
**Authority:** Normative only for provisioning scripts and projects that explicitly adopt this policy.  
**Scope:** Multi-line content, file writes, and user-facing reports in provisioning shell scripts.  
**Provenance:** Refined through the Hermes provisioning script series (019–025) and predecessors, as supplied for this contribution. This records provenance; it does not independently certify every historical test.

## 1. Purpose and scope

This policy governs how provisioning shell scripts represent, emit, and write multi-line content. It preserves readable control flow, literal content, safe privilege boundaries, reusable content units, and predictable behaviour across repeated runs.

It is deliberately scoped. It is not a general shell style guide and does not automatically apply to unrelated scripts. A project adopts it by reference or explicit compatibility decision.

## 2. Normative rules

### P1 — Function-wrap content-producing heredocs

A heredoc whose body is reusable or conceptually distinct content to be written somewhere is emitted by a descriptively named conf_print_* function. The function describes the content; the call site describes the action.

~~~bash
conf_print_hermes_soul() {
  cat <<'VERBATIM'
[... content ...]
VERBATIM
}
~~~

This keeps substantial content from obscuring control flow and gives it a stable extraction seam.

**Report exception:** user-facing reports are governed by P3. They use a direct glow heredoc rather than a conf_print_* function. This is a deliberate exception to P1.

### P2 — Write files through tee

The standard file-write path is tee or tee -a, with output silenced by >/dev/null unless output is intentionally being inspected. Use sudo tee for privileged destinations. Do not use sudo command > file for root-owned files: the invoking shell performs the redirection before the privileged command runs.

~~~bash
conf_print_hermes_soul | tee "$SOUL" >/dev/null
conf_print_hermes_soul | sudo tee "$SOUL" >/dev/null
conf_print_something | tee -a "$FILE" >/dev/null
~~~

**Narrow inline exception:** Pattern A in §3 permits a short, guarded, two-line declaration to use inline cat > file where wrapping would be pure ceremony. It never applies to root-owned paths and does not permit sudo cat <<... > /root/file.

### P3 — Render user-facing reports with glow

User-facing multi-line reports use one glow invocation and one heredoc per report. Do not use multi-line echo; do not pipe reports through tee. glow writes the rendered report to stdout.

~~~bash
glow - <<MESSAGE
**Layer complete.**
- Item: value
MESSAGE
~~~

This assumes glow is available in the target environment. If unavailable, record the dependency gap rather than silently substituting another output convention.

### P4 — Precompute state before report heredocs

Any command substitution ($(...) or backticks) needed in a glow report is evaluated before the heredoc and stored in a variable. Interpolate the variable in the report; do not put command substitutions inside the report body.

~~~bash
if [[ -s "$FILE" ]]; then STATE="active"; else STATE="empty"; fi
glow - <<MESSAGE
- Constraint file: $FILE ($STATE)
MESSAGE
~~~

This keeps report construction readable and makes state determination independently inspectable.

### P5 — Quote heredoc delimiters by default

Use <<'EOF' or <<'VERBATIM' for literal content. Quoted delimiters prevent parameter, command, and arithmetic expansion in the body.

An unquoted delimiter is permitted only when an expansion must occur while the shell constructs the content. Every unquoted heredoc must have an adjacent comment naming the required substitution and why it is required.

~~~bash
# Literal content: no expansion
write_file "$SOUL" <<'VERBATIM'
# SOUL.md — written literally
VERBATIM

# Expansion required: openssl generates a value at write time
write_file "$API_RC" <<EXPAND
export HERMES_API_KEY="$(openssl rand -hex 32)"
EXPAND
~~~

Do not use <<"EOF". It is functionally equivalent to <<'EOF' for a fixed delimiter and adds no useful distinction in this domain. Use the single-quoted convention for literal content.

### P6 — Keep short inline heredocs short; wrap content-shaped heredocs

A heredoc shorter than approximately ten lines and used at one call site may remain inline when incidental to the surrounding logic.

Wrap a heredoc when it is long, interrupts the reader's view of control flow, or represents conceptually distinct content such as SOUL.md, AGENTS.md, configuration files, or tmuxp layouts. Content-shaped heredocs are function-wrapped regardless of line count when the function seam is useful.

The threshold is qualitative, not mechanical: if a reader must scroll through content to find the next control-flow statement, wrap it.

### P7 — Treat the content function as a refactoring seam

A function-wrapped content unit is movable. It may begin inline and later be extracted into a sourced file without changing the write call site.

Inline heredocs are not equivalently movable. Use a function boundary when content may grow, be shared, versioned, tested, or independently reviewed. The function's location is deferred; the seam is preserved now.

### P8 — Keep write, quote, justify, and guard concerns separate

For file writes, each concern belongs in one place:

| Concern | Lives in | Responsibility |
|---|---|---|
| **Write** | write_file helper | Owns directory creation, tee, append, and privilege handling |
| **Quote** | Call site | Selects quoted literal or justified unquoted expansion |
| **Justify** | Comment beside the heredoc | Names and explains each required expansion |
| **Guard** | Call site | Determines whether writing or content rotation is appropriate |

Do not move caller-specific content stability into a generic write helper. Do not make the helper infer whether a write is safe to repeat.

### P9 — Guard non-deterministic content or document intentional rotation

Every unquoted heredoc must identify its expansion in a comment. For non-deterministic substitutions such as openssl rand, date, or uuidgen, the comment must also state whether a new value is intended on every provisioning run.

- If rotation is intended, say so explicitly.
- If rotation is not intended, guard the write so the generated value remains stable across runs.

~~~bash
# Rotation on every run is intentional: generated_at records this provisioning run.
write_file "$FILE" <<EXPAND
generated_at=$(date -u +%FT%TZ)
EXPAND

# Stable across runs: create the secret only when the file does not exist.
if [[ ! -f "$FILE" ]]; then
  # Expansion: generate the initial secret once; the guard prevents rotation.
  write_file "$FILE" <<EXPAND
secret="$(openssl rand -hex 32)"
EXPAND
fi
~~~

The helper is idempotent only in the limited sense that it writes supplied stdin (truncating by default or appending when requested). It does not make generated content stable. Content stability is the caller's responsibility, established by deterministic input, a guard, or an explicitly intended rotation policy.

### P10 — Use one write_file helper for standard writes

A single helper owns the standard write operation. Given a destination, optional append and privilege flags, and content on stdin, it creates the destination's parent directory with the same privilege level as the write, then writes silently through tee or tee -a.

~~~bash
# write_file [-a] [-s] DEST
#   -a  append (default: truncate)
#   -s  use sudo for a root-owned destination
# Creates DEST's parent directory with the same privilege as the write.
# Reads content from stdin.
write_file() {
  local append=0 use_sudo=0
  while [[ $# -gt 0 ]]; do
    case "$1" in
      -a) append=1; shift ;;
      -s) use_sudo=1; shift ;;
      *) break ;;
    esac
  done
  local dest="$1"

  if [[ "$use_sudo" == 1 ]]; then
    sudo mkdir -p "$(dirname "$dest")"
    if [[ "$append" == 1 ]]; then
      sudo tee -a "$dest" >/dev/null
    else
      sudo tee "$dest" >/dev/null
    fi
  else
    mkdir -p "$(dirname "$dest")"
    if [[ "$append" == 1 ]]; then
      tee -a "$dest" >/dev/null
    else
      tee "$dest" >/dev/null
    fi
  fi
}
~~~

The four supported call forms are:

~~~bash
write_file       "$FILE"      <<'VERBATIM'
...
VERBATIM

write_file -a    "$FILE"      <<'VERBATIM'
...
VERBATIM

write_file -s    "$ROOT_FILE" <<'VERBATIM'
...
VERBATIM

write_file -s -a "$ROOT_FILE" <<'VERBATIM'
...
VERBATIM
~~~

The -s flag is per call, not an environment variable. A persistent environment setting can leak privilege behaviour between calls when a script writes both user-owned and root-owned files.

Directory creation belongs inside the privilege branch. An unconditional unprivileged mkdir -p can fail on a root-owned parent before the privileged write path is reached.

This helper is a reference implementation, not a universal drop-in library. Consumers must check their shell, error-handling mode, sudo policy, path assumptions, and desired failure propagation. Ensure a failed mkdir or tee cannot be mistaken for a successful write; test under the script's actual set -e, pipeline, and execution settings.

## 3. Pattern selection

Four patterns are available. Select according to content length and whether the content may be reused, independently reviewed, or extracted.

| Pattern | Content lives in | Write form | Use when |
|---|---|---|---|
| **A — inline heredoc** | At the write site | cat > "$FILE" <<'VERBATIM' | Guarded, two-line declaration where a function would be ceremony; never for privileged paths |
| **B — inline content function** | A conf_print_* function in the script | conf_print_x | write_file "$FILE" | Long, one-shot, content-shaped content |
| **C — helper-wrapped heredoc** | At the write site | write_file "$FILE" <<'VERBATIM' | Short, one-shot content that does not need a separate content seam |
| **D — sourced content function** | An external content/*.sh file | conf_print_x | write_file "$FILE" | Long or high-reuse content meriting independent review/versioning |

Decision rule:

> If content might be referenced by name, edited as a standalone unit, or reused from another layer, use Pattern B or D regardless of length. Otherwise use C for short one-shot content. Use A only for a guarded two-line declaration where wrapping adds no useful seam.

Pattern C does not preserve the same refactoring seam as B or D: the content remains embedded at the write site. Do not use C when independent addressability or reuse matters.

## 4. Applicability

Applies to provisioning-script file content (SOUL.md, AGENTS.md, configuration fragments, .env templates, rc fragments, tmuxp layouts, and similar generated files); user-facing multi-line reports (completion messages, warnings, and state summaries); heredocs longer than approximately ten lines; heredocs that could reasonably be extracted into sourced content functions; and writes to privileged destinations, subject to the privilege requirements above.

Does not automatically apply to scripts and repositories that have not adopted this policy; two-line declarations meeting the narrow Pattern A exception; short inline guards or conditionals that are not content emission; or heredocs established as non-reusable and non-growing, where the exception is documented and does not weaken quoting or privilege safety.

When in doubt, preserve the function seam. The cost of an unnecessary wrapper is generally lower than the cost of embedding content that later needs extraction.

## 5. Anti-patterns

- **Inline command substitution in report heredocs:** precompute state and interpolate a variable instead (P4).
- **Unquoted heredocs for literal content:** variables and backticks can be expanded or interpreted; use a quoted delimiter (P5).
- **Shell redirection for root-owned writes:** do not rely on sudo to elevate a redirection performed by the invoking shell; use write_file -s or sudo tee (P2/P10).
- **Unconditional unprivileged directory creation:** keep mkdir -p inside the corresponding privilege branch (P10).
- **Duplicated content functions:** extract shared content into one sourced file (P7).
- **Multi-line echo reports:** use one glow heredoc (P3).
- **Unjustified unquoted heredocs:** name and justify each required substitution (P5).
- **Non-deterministic content without a guard or rotation note:** guard stable values or document intentional rotation (P9).

## 6. Rationale summary

| Property to preserve | Policy mechanism |
|---|---|
| Script structure remains visible | P1, P6 |
| Content is addressable, testable, and reusable | P1, P7; Patterns B and D |
| Writes respect privilege boundaries | P2, P10 |
| Write operations are uniform and silent by default | P2, P10 |
| Reports are consistently formatted | P3 |
| Report state is independently inspectable | P4 |
| Literal content is not accidentally expanded | P5 |
| Short scripts avoid unnecessary ceremony | P6; narrow Pattern A exception |
| Future extraction remains possible | P7 |
| Write, quote, justification, and guard responsibilities remain separate | P8 |
| Repeated runs do not silently rotate generated values | P9 |
| Parent directories are created with appropriate privilege | P10 |

## 7. Adoption, compatibility, and conflict handling

A consuming project should identify this policy by revision and source location when adopting it. A copied or adapted version should preserve provenance and record its divergence.

This policy does not override a consuming project's explicit security requirements, platform/shell constraints, or a higher-authority project policy.

If a conflict arises, identify the exact provisions, affected operation, and practical impact. Stop the affected write/report pattern until an authorized compatibility decision or policy revision is recorded. Do not silently treat a local convention as an exception.

If glow, sudo, or another assumed utility is unavailable, record the dependency mismatch and resolve it explicitly. Do not claim compatibility merely because the script parses.

## 8. Validation expectations

A policy adoption or implementation review should distinguish:

- **Structural:** required helper and content-function patterns, quoting forms, comments, and call-site guards are present.
- **Semantic:** each heredoc is classified correctly; expansions are intentional; guards preserve stated rotation behaviour; privilege boundaries match the destination.
- **Operational:** representative writes and reports are executed in the target shell/environment, including user-owned and privileged paths where applicable.
- **Reproducibility:** shell version, utility versions, test inputs, privilege setup, and observed outcomes are recorded.

At minimum, test:
1. Literal content containing dollar signs, backticks, and backslashes.
2. A justified expansion.
3. Append and truncate modes.
4. User-owned and privileged destination paths.
5. Parent-directory creation in both privilege modes.
6. Stable non-deterministic content under a guard.
7. Intentional rotation.
8. Report rendering with precomputed state.
9. Failure propagation for directory creation and writes.

A syntax check is not operational validation. A successful write in one environment does not establish compatibility in every target environment. Record checks not run and unresolved platform assumptions.

## 9. Revision history

| Revision | Summary |
|---|---|
| 1.0.0 | Initial proposed repository policy, scoped to adopting provisioning scripts; clarifies report and inline-write exceptions and records validation expectations. |
