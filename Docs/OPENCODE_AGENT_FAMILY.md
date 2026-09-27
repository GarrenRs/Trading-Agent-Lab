# OPENCODE AGENT FAMILY - TRADING AGENT LAB

**Status:** Active
**Authority:** This document governs agent roles, model routing, permissions,
invocation policy, workflow, and closure rules.
**Established under:** Phase V0 (work-engine establishment).
**Reconciled under:** Phase V0.1 (documentation reconciliation and agent runtime
validation).
**Repository root:** `F:\Trading` - independent Git repository, no remote. See
`Docs/OPENCODE_AGENT_FAMILY_DECISIONS.md` decision D-015.
**Verified against:** OpenCode `v2.0.13` (config schema OpenCode V2).
**Coordinator model:** `Big Pickle` (main session), which resolves to
`opencode/big-pickle` on the `opencode` provider. It is the Coordinator only and
is deliberately **not** one of the `opencode-go/*` family models.

Where this document and an individual agent prompt conflict, this document
wins. Report the conflict to the Coordinator.

---

## 1. Coordinator

The **Coordinator is the main session itself** - not a subagent.

No competing coordinator agent is defined. This is deliberate: one coordinator,
one session, one point of authority.

The Coordinator is responsible for:

1. Understanding the current phase
2. Reading the project authority documents
3. Decomposing work into bounded tasks
4. Selecting the correct subagent for each task
5. Sequencing work
6. Collecting evidence
7. Deciding when a task is ready for verification
8. Deciding when a workstream is ready for review
9. Enforcing scope
10. Enforcing clean closure
11. Enforcing Git discipline

The Coordinator is **not** a normal coding worker. It does not implement
application code to avoid delegating. When a task is implementation work, it
routes to `builder`.

---

## 2. The agent family

Exactly five permanent project subagents exist.

| # | Agent | Mode | Model | Writes? | Purpose |
| --- | --- | --- | --- | --- | --- |
| 1 | `explorer` | subagent | `opencode-go/glm-5.3-flash` | No | Evidence and discovery only |
| 2 | `quant-researcher` | subagent | `opencode-go/glm-5.3` | No | Trading research and quantitative evidence |
| 3 | `builder` | subagent | `opencode-go/kimi-k2.7-code` | Yes | Implementation only |
| 4 | `tester` | subagent | `opencode-go/glm-5.3-flash` | Yes (minimal) | Verification and narrow corrective fixes |
| 5 | `reviewer` | subagent | `opencode-go/gpt-5.6-luna` | No | Final read-only technical and security review |

Definitions: `.opencode/agents/<name>.md`
Project config: `.opencode/opencode.jsonc`

No additional permanent agents are created in V0.

---

## 3. Model routing

All five **family member** models are **OpenCode Go** (`opencode-go/*`). **No
free-tier model is used for the primary project agent family.** The Coordinator
is outside this family and runs `opencode/big-pickle` on the `opencode`
provider; that is a deliberate exception, not an inconsistency.

| Agent | Model | Tier | Rationale |
| --- | --- | --- | --- |
| `explorer` | `opencode-go/glm-5.3-flash` | Fast | Search and summarisation; errors are cheap and re-runnable |
| `quant-researcher` | `opencode-go/glm-5.3` | Standard | Difficult evidence analysis, where a wrong conclusion is costly |
| `builder` | `opencode-go/kimi-k2.7-code` | Coding-specialised | Implementation is the main cost centre and needs code precision |
| `tester` | `opencode-go/glm-5.3-flash` | Fast | Running checks and classifying failures is mechanical |
| `reviewer` | `opencode-go/gpt-5.6-luna` | Highest | Final correctness and security gate; highest cost of an error |

### Model policy

Model selection is **task-based, not prestige-based**.

The strongest model is reserved for work where the cost of an error is high -
here, the final review. Routine exploration and verification use the fast tier
and are trivially re-runnable when wrong.

Do not escalate a routine task to a stronger model without a stated reason.

### Verified model identifiers

All five identifiers were confirmed present via `opencode models` on the
installed build. **If an identifier ever becomes unavailable, stop and report
the exact mismatch. Never substitute a different model silently.**

**How to verify these identifiers (corrected in V0.1).** The *unfiltered*
`models` listing enumerates **only the `opencode` provider**. If you rely on
that listing alone, the `opencode-go/*` identifiers will appear to be missing
and you may wrongly conclude the configuration is broken. Query the provider
explicitly instead:

- `opencode models` with `provider: "opencode-go"` - returns the `opencode-go`
  catalogue, including all three of `glm-5.3-flash`, `glm-5.3`, and
  `kimi-k2.7-code`.
- `opencode models` with a name query such as `gpt-5.6-luna` - returns
  `opencode-go/gpt-5.6-luna`, which has **no** `opencode`-provider equivalent
  and is therefore easy to miss.

**Runtime validation (V0.1, verified).** All five agents were launched with
read-only probes. All five launched successfully on their configured models,
with no substitution and no permission anomaly. Each agent separately reported
the model identity disclosed in its own session context; **all five matched
their frontmatter on both provider and model id**, and all five reported the
`opencode-go` provider. The two write-capable agents completed without using
`edit`, and the working tree was confirmed unchanged afterwards. The evidence is
agent self-report of session context, not a server-side attestation, and is
described with that limit in full under decision D-016 in the decision register.

---

## 4. Permissions

Permissions use the OpenCode V2 ordered-rule format
(`{ action, resource, effect }`). **The last matching rule wins**, so broad
denies are declared first and exceptions follow.

### Enforced boundary

| Agent | `edit` | `shell` | `subagent` | `read`/`glob`/`grep` | `webfetch`/`websearch` |
| --- | --- | --- | --- | --- | --- |
| `explorer` | **deny** | **deny** | **deny** | allow | allow |
| `quant-researcher` | **deny** | allow, destructive + network + git denied | **deny** | allow | allow |
| `builder` | allow | allow, push + destructive git denied | **deny** | allow | allow |
| `tester` | allow (minimal) | allow, push + destructive git denied | **deny** | allow | allow |
| `reviewer` | **deny** | **deny** | **deny** | allow | allow |

Notes:

- **`shell` is the V2 action name.** The legacy V1 `bash` key no longer exists
  and is not used. V1's `permission:` map is also not used; V2 requires a
  `permissions:` list of `{ action, resource, effect }`.
- **`list` is not a V2 permission action.** It is absent from the V2 action
  table, so a legacy `list: allow` is an inert string. Directory listing is
  covered by the `read` and `glob` actions, and no `list` rule is set here.
- **Rule order is load-bearing.** The OpenCode base policy is
  `* -> allow`, `external_directory: * -> ask`, `read: "*.env" -> ask`,
  `read: "*.env.*" -> ask`, `read: "*.env.example" -> allow`. Because the last
  matching rule wins, a trailing `read: "*" -> allow` would silently override
  the `.env` guard. **Every agent therefore re-asserts `*.env` and `*.env.*` as
  `deny` after its `read: *` allow**, and re-opens `*.env.example`.
- **Every agent denies `subagent: *` as its final `subagent` rule**, so the
  project-level allow-list cannot re-grant nesting to a subagent. This is what
  guarantees a subagent can never invoke another subagent.
- **Every agent denies `external_directory: *`.** Scope this precisely: it
  governs the `read`, `edit`, `write`, and `patch` **path** checks only. It does
  **not** constrain `shell`, which runs with the host user's full filesystem,
  process, and network authority. For the three read-only agents this is a real
  boundary, because their `shell` is denied outright. For `builder` and
  `tester` it is **not** a containment boundary.
- **The `.env` guard governs the `read` tool only.** `read: "*.env" -> deny`
  blocks reading a secret file *as a file*. It cannot stop a shell-capable agent
  from printing the same file with `type`, `cat`, or `Get-Content`. The
  instruction-level prohibition in `AGENTS.md` section 10 is the actual control
  for `builder`, `tester`, and `quant-researcher`.
- `builder` and `tester` deny `git push`, `git commit`, `git reset`,
  `git rebase`, `git clean`, and `git rm` in their agent rules, **and** the
  project config re-denies the same operations as hard-deny
  `experimental.policies` (see below). **Only the Coordinator commits, and
  nothing ever pushes automatically.**

### Hard-deny policies (not agent rules)

Agent `permissions` are ordinary last-match-wins rules, so a broader later rule
can override a narrower earlier deny. `experimental.policies` in
`.opencode/opencode.jsonc` are different: the V2 permissions guide states they
run **after** permission rules and saved approvals, turn `allow` or `ask` into
`deny`, and "never grant access".

They are used for the operations that are reliably matchable as command text
and unacceptable for **everyone**, including the Coordinator: `git push`,
`git reset`, `git rebase`, `git clean`, and `git filter-branch`. Verified as
accepted and resolved by `opencode debug config` on v2.0.13.

`git commit` is deliberately **not** in this list. Policies are global and
cannot be scoped per agent, and an agent rule cannot override them, so denying
`git commit` here would also block the Coordinator - the only role allowed to
commit. Committing is denied at agent level on all five subagents instead, which
is correctly scoped and verified to resolve to `deny`.

### What this configuration does NOT enforce

Stated plainly so no one over-reads the table above:

1. **It does not sandbox `shell`.** For `builder`, `tester`, and
   `quant-researcher`, `shell` is allowed. A command that does not textually
   match a deny pattern may still execute. The V2 permissions guide states that
   `shell` "runs with the host user's filesystem, process, and network
   authority", that "directory inference from command text is best effort", and
   recommends "a narrow shell allowlist instead of patterns intended to
   recognize every dangerous command".
2. **It does not stop a shell-capable agent from reading a secret file.** Only
   the read tool is protected. The written prohibition is the control.
3. **It does not stop a shell-capable agent reaching outside the repository.**
   `external_directory` does not apply to `shell`.
4. **It does not stop a well-formed but wrong command.** Permission rules match
   text, not intent.

**Consequence:** for the three shell-capable agents, the prohibitions in
`AGENTS.md` sections 10 and 11 and in each agent's instructions are the primary
security control. The configuration is defence in depth that raises the cost of a
mistake, not a containment boundary. A genuine containment boundary requires
running agents in an OS-level sandbox or container, which is out of V0 scope and
is recorded as open decision O-009.


### Coordinator invocation rights

`.opencode/opencode.jsonc` denies `subagent: *` and then re-allows exactly the
five family members. The main session can therefore launch precisely these five
and nothing else, including any agent inherited from a parent directory.

**Current state (V0.1, verified):** no inherited agent resolves at all. From
`F:\Trading`, `opencode debug agents` returns 12 agents and **none of them is
one of the four `orkestrix-*` agents** defined under the old
`F:\OpenProperty-master` tree. The allow-list described above is therefore
defence in depth rather than the only control. Were the project ever nested
inside another repository again, the ancestor-directory walk would resume and
the allow-list would become load-bearing once more. See D-011 and D-015 in the
decision register.

---

## 5. Invocation policy

**Subagents must never invoke other subagents.** Sequencing is the Coordinator's
responsive alone. This is enforced by `subagent: deny` on all five agents.

### Default execution order

```
Research / Discovery   ->  explorer
                           + quant-researcher  (when the task needs domain research)
Implementation         ->  builder
Verification           ->  tester
Final Review           ->  reviewer
Closure and commit     ->  Coordinator
```

### Concurrency rules

- **Never** run two write-capable agents (`builder`, `tester`) on the same
  files concurrently.
- Read-only agents (`explorer`, `quant-researcher`, `reviewer`) may run in
  parallel **only** when they share no mutable state.
- The Coordinator may skip any agent the current task does not require.

---

## 6. Role detail

### 6.1 `explorer`

Evidence and discovery only. Inspects structure, code, and documentation;
searches for reusable patterns; maps dependencies and coupling; identifies
risks; prepares implementation context.

- Never modifies files. Never implements. Never commits.
- Never decides architecture unilaterally.
- Reports `VERIFIED` / `INFERENCE` / `RECOMMENDATION` / `UNKNOWN`.

### 6.2 `quant-researcher`

Build-time trading research. Investigates strategy families, academic and
industry evidence, open-source implementations, transaction costs, execution
risks, and known failure modes; prepares strategy specifications for the
Strategy Registry.

**This agent is not the runtime Trading Agent.** It is a development-time
research agent (see `AGENTS.md` section 2).

Prohibited: implementing the trading core, submitting exchange orders, accessing
live exchange credentials, trading real money, unauthorised architecture changes,
and silently promoting a strategy to approved status.

Every result separates `VERIFIED FACT` / `INFERENCE` / `RECOMMENDATION` /
`UNKNOWN`, and every strategy claim names its source and evidence quality class
(`PEER-REVIEWED`, `REPLICATION`, `PREPRINT`, `PRACTITIONER`, `OPENSOURCE`,
`UNVERIFIED`).

### 6.3 `builder`

The only role that writes production code. Implements the single authorised
task, writes the task's tests, runs the required development commands, and
reports changed files, tests, and remaining issues.

Prohibited: live trading, real credentials, withdrawals, transfers, real-money
execution, scope expansion, unauthorised architecture redesign, arbitrary
dependency installation, `git push`, destructive Git commands, and committing
without explicit Coordinator instruction.

### 6.4 `tester`

Runs tests, type checks, and lint/build checks; reproduces failures; identifies
root causes; applies only the minimal fix needed to make the active task pass;
re-runs the checks.

Prohibited: new features, strategy invention, architecture redesign, unrelated
refactoring, broad cleanup, weakening tests, committing without instruction,
live trading, and real credentials.

Reports `FAILED` / `CAUSE` / `CHANGE` / `VERIFICATION` / `REMAINING`, and
distinguishes an `ENVIRONMENT` failure from an `APPLICATION` failure.

### 6.5 `reviewer`

Final read-only technical and security review. Reviews architecture, contracts,
security, scope leakage, regressions, test coverage, error handling, trading
safety boundaries, dependency changes, and documentation completeness.

Reports `CRITICAL` / `HIGH` / `MEDIUM` / `LOW` / `PASS`. **A workstream is not
cleanly closed while any `CRITICAL` or `HIGH` finding remains unresolved.**

---

## 7. Workflow

The proven pipeline, per workstream:

```
Baseline -> Audit -> Bounded Plan -> Controlled Execution -> Test
         -> Review -> Documentation -> Clean Closure -> Git Commit
```

- **Baseline** - record the starting `git status` and current behaviour.
- **Audit** - establish evidence with `explorer` and `quant-researcher`.
- **Bounded Plan** - write a task with explicit, finite scope.
- **Controlled Execution** - `builder` implements only that scope.
- **Test** - `tester` verifies and classifies failures.
- **Review** - `reviewer` applies the final gate.
- **Documentation** - update the authority documents to match reality.
- **Clean Closure** - confirm every gate below, then commit.

---

## 8. Closure rules

A workstream is closable only when **all** of the following hold:

1. The authorised scope is complete. Nothing is half-done.
2. Tests pass, and no test was weakened to achieve that.
3. `tester` reports no outstanding failure.
4. `reviewer` reports **no `CRITICAL` and no `HIGH` findings**.
5. Documentation matches actual behaviour.
6. `git status` shows only expected files.
7. The complete diff has been read, not just the summary.
8. No secret is present anywhere in the staged content.
9. No trading, exchange, database, or UI code was created outside scope.

If any check fails, the workstream is **not** closable. Fix it, or record it as
an open item with a named owner and next step.

Commit messages follow Conventional Commits. **Never push automatically.
Never rewrite Git history.**

---

## 9. Resume and checkpoint policy

On resuming an interrupted session:

1. `git status`
2. `git diff --stat`
3. current task state
4. latest checkpoint or decision record
5. re-run the previously failing verification, where applicable
6. continue only once the state is understood

If a task cannot be completed in the current session, record: work completed,
work incomplete, the exact next step, blockers, and relevant files.

**Never present an interrupted task as complete.**

---

## 10. Scope boundary of this phase

V0 delivers the work-engine only. It does not include, and no agent may begin:
trading code, the trading core, exchange integration, a database, a dashboard or
other UI, or strategy implementations.

Related: `Docs/OPENCODE_AGENT_FAMILY_DECISIONS.md`
