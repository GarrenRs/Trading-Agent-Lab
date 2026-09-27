# OPENCODE AGENT FAMILY - DECISION REGISTER

**Status:** Active
**Scope:** Phase V0 - OpenCode work-engine establishment.
**Updated under:** Phase V0.1 - documentation reconciliation and agent runtime
validation.
**Rule:** New decisions are recorded here, not in chat. An undocumented decision
is not an agreed decision.

**How to read the entries in this register.** Statements that were true when
written but are no longer true are **retained and marked `HISTORICAL` or
`SUPERSEDED`**, never silently deleted. A decision register that quietly removes
its own history cannot be audited. Each such entry names the decision that
replaces it.

---

## 1. Verified decisions

Verified means directly observed and confirmed against the installed tooling.

### D-001 - OpenCode V2 configuration syntax is used, not V1 legacy

- **Decision:** All agent definitions and project config use the OpenCode V2
  schema.
- **Verified:** Installed build is `opencode v2.0.13`. Schema confirmed against
  `https://opencode.ai/v2/docs/config` and `https://opencode.ai/v2/docs/agents`.
- **Consequence:** No legacy `permission:` map, no `bash` key, no `maxSteps`,
  no `tools`, no `disable`, no `temperature`, no `top_p`. Permissions are an
  ordered `permissions:` list of `{ action, resource, effect }`.

### D-002 - Project-local agent location is `.opencode/agents/` (plural)

- **Decision:** Agents live at `.opencode/agents/<name>.md`.
- **Verified:** The V2 agents guide documents `~/.config/opencode/agents/` and
  `.opencode/agents/` as the supported locations.
- **Note:** The baseline project this workflow was proven in uses the **V1
  singular** path `.opencode/agent/`. That path was deliberately **not** copied.

### D-003 - `mode: subagent` for all five family members

- **Decision:** All five agents are `mode: subagent`.
- **Verified:** V2 modes are `primary`, `subagent`, `all`. The Coordinator is the
  main session and uses the default `build` primary agent, so no family member
  needs `primary`.
- **Consequence:** No competing coordinator agent exists, as required.

### D-004 - System prompt lives in the Markdown body, not frontmatter

- **Decision:** The Markdown body is the system prompt. No `system:` frontmatter
  field is used.
- **Verified:** The V2 agents guide states a Markdown agent's body becomes its
  system prompt, and that a `system` frontmatter field should not be used in
  Markdown definitions.

### D-005 - All five model identifiers exist on the installed build

- **Decision:** The specified routing is used exactly as given.
- **Verified:** `opencode models` lists all of
  `opencode-go/glm-5.3-flash`, `opencode-go/glm-5.3`, `opencode-go/kimi-k2.7-code`,
  and `opencode-go/gpt-5.6-luna` under the `opencode-go` provider.
- **Consequence:** No model was substituted, and no mismatch was found.
- **Precision note added in V0.1 (verified):** the unfiltered `models` listing
  enumerates **only the `opencode` provider**, which makes the `opencode-go/*`
  identifiers appear to be missing. That is an artifact of the default listing,
  **not** a configuration fault. The `opencode-go` provider must be queried
  **explicitly by provider or by model name** to see it; queried that way, all
  four identifiers are present, including `opencode-go/gpt-5.6-luna`, which has
  no `opencode`-provider equivalent.
- **Runtime confirmation added in V0.1 (verified):** all five agents launched
  successfully on their configured models. See D-016.
- **Coordinator model, clarified in V0.1 (verified):** the main session runs
  `Big Pickle`, which resolves to **`opencode/big-pickle` on the `opencode`
  provider**, not on `opencode-go`. It is a real, executable identifier. The
  "all models are OpenCode Go" claim in the family document is scoped to the
  five-agent family and does not include the Coordinator.

### D-006 - The `instructions` config field is deliberately unset

- **Decision:** `.opencode/opencode.jsonc` does not set `instructions`.
- **Verified:** The V2 config guide states the field is accepted but **not
  loaded**, and directs projects to use `AGENTS.md` instead.
- **Consequence:** `AGENTS.md` is auto-discovered and applied. Confirmed working
  in practice: the file was injected as project instructions during V0 setup.

### D-007 - Subagent chaining is denied structurally, not just by instruction

- **Decision:** Every one of the five agents sets
  `{ action: subagent, resource: "*", effect: deny }`.
- **Verified:** The V2 agents guide confirms a parent's `subagent` permissions
  govern which agents it may launch, and that permission rules append with the
  last match winning. An agent-level deny therefore overrides the project-level
  allow-list.
- **Consequence:** "Subagents must not invoke subagents" is enforced by
  configuration, not by prompt text alone.

### D-008 - Coordinator invocation is allow-listed explicitly

- **Decision:** Project config denies `subagent: *` and then re-allows exactly
  the five family members.
- **Verified:** This is the documented OpenCode pattern for restricting which
  child agents a parent may launch.
- **Consequence:** The main session can launch precisely the intended five.

### D-009 - TRADING AGENT LAB is a standalone Git repository

- **Decision:** `git init` inside `Trading/`, creating an independent repository
  on branch `main`, separate from the surrounding OpenProperty repository.
- **Verified:** Before initialisation, `Trading/` was an empty, untracked
  subdirectory of the OpenProperty repo, whose remote is `GarrenRs/MCP` and
  whose working tree already held unrelated uncommitted work.
- **Rationale:** Approved explicitly by the project owner during V0 setup.
  Committing from inside `Trading/` would otherwise have written the trading
  work-engine into an unrelated product's history, on a remote belonging to
  that other project.
- **CORRECTION (verified after the fact):** initialising Git did **not** stop
  OpenCode from loading the parent project's legacy agents. See D-011.
- **HISTORICAL (superseded by D-015, V0.1):** the location `Trading/` described
  above is no longer the project root. The repository was subsequently moved to
  `F:\Trading`. The decision to hold an independent Git history **stands**; only
  the path in it is obsolete.

### D-010 - Role separation is preserved

- **Decision:** Exactly the five specified agents exist. No additional permanent
  agents are created in V0.
- **Rationale:** Preserves the proven baseline's non-overlapping roles:
  explorer researches, the builder implements, the tester verifies, and the
  reviewer gates.

### D-011 - Parent-project agents remain loaded; the allow-list is the control

> **STATUS: SUPERSEDED by D-015 (V0.1).** The premise of this decision no longer
> holds. The entry is retained verbatim as the historical record of what was
> verified **while the project sat inside the OpenProperty tree**. Do not read
> it as current state. Current state: the four `orkestrix-*` agents **no longer
> resolve at all** from `F:\Trading`.

- **Decision:** Accept that the four legacy `orkestrix-*` agents from
  `F:\OpenProperty-master\.opencode\agent\` remain present in the resolved agent
  list, and rely on the project-level `subagent` allow-list to prevent the
  Coordinator from launching them.
- **Verified:** `opencode debug agents`, run from `Trading/`, still resolves
  `orkestrix-builder`, `orkestrix-explorer`, `orkestrix-reviewer`, and
  `orkestrix-tester`.
- **Why they still load:** OpenCode searches the current directory and **every
  ancestor up to the filesystem root**, including directories above the detected
  project or repository root. A nested Git repository does not stop that walk,
  so the earlier assumption in D-009 was wrong.
- **Why this is still safe:** the project config denies `subagent: *` and
  re-allows only the five family members, and the resolved Coordinator policy
  shows `subagent: * -> deny` followed by the five `allow` rules. Per the V2
  permissions guide, a parent's `subagent` rules govern which agents it may
  launch, so those four cannot be launched from this project.
- **Residual risk:** they remain *visible* in `opencode debug agents`. Visibility
  is not authority, but the list is misleading. See O-008.
- **To confirm:** a live launch test proving the four are rejected, in V1.
- **What actually resolved it (V0.1, verified):** the residual risk above was
  closed by **moving the repository out of the OpenProperty tree**, not by
  tightening permissions. The ancestor-directory walk described under "Why they
  still load" only reaches `F:\OpenProperty-master` when the project is a
  descendant of it. `F:\Trading` is not, so the walk no longer reaches those
  files. `opencode debug agents` from `F:\Trading` returns 12 agents, **none of
  which is named `orkestrix-*`**. The `subagent` allow-list from D-008 is
  therefore now defence in depth rather than the sole control. See D-015.
- **Consequence for the D-009 CORRECTION:** the finding that a nested Git
  repository does not stop the ancestor walk **remains true and is not
  retracted**. What changed is that the project is no longer nested.

### D-012 - The base `.env` read guard is explicitly re-asserted on every agent

- **Decision:** All five agent definitions re-assert, **after** their
  `read: * allow` rule, an explicit `read: *.env -> deny` and
  `read: *.env.* -> deny`, followed by `read: *.env.example -> allow`.
- **Why:** the OpenCode base policy ships
  `read: "*.env" -> ask` and `read: "*.env.*" -> ask`, and the permissions guide
  states that later global and agent rules can override these defaults. Because
  the last matching rule wins, a trailing `read: "*" -> allow` silently
  **overrides** the secret-file guard. This was verified in the resolved output
  of `opencode debug agents` before the fix: the base `*.env` rules appeared
  *above* a broader `read: *` allow.
- **Consequence:** `.env` files cannot be read by any of the five agents, while
  the committed `.env.example` template stays readable, which `AGENTS.md`
  section 10 requires.
- **Note:** the baseline project has the same latent regression; it was not
  inherited here.

### D-013 - Hard-deny `experimental.policies` adopted for Git operations

- **Decision:** `.opencode/opencode.jsonc` sets `experimental.policies` hard
  denies for `git push`, `git reset`, `git rebase`, `git clean`, and
  `git filter-branch`. It deliberately does **not** deny `git commit`.
- **Why:** the V2 permissions guide states that policies "can hard-deny a
  permission check after these rules and saved approvals run. It turns `allow`
  or `ask` into `deny` and never grants access." Unlike ordinary agent
  permissions, they cannot be overridden by a broader later rule or by a saved
  "allow always" approval.
- **`git commit` is excluded on purpose:** policies are global and cannot be
  scoped per agent, and an agent rule cannot override them. Denying `git commit`
  there would also block the **Coordinator**, which is the only role permitted to
  commit. Committing is instead denied at agent level on all five subagents,
  which is correctly scoped and verified to resolve to `deny`.
- **Verified:** `opencode debug config` on v2.0.13 parses the block and resolves
  every policy entry, with no error and no warning.
- **Scope limit, stated deliberately:** this covers only reliably matchable Git
  command text. It is **not** a shell sandbox, and it does not stop a
  shell-capable agent reading an `.env` file. See D-014 and O-009.

### D-014 - The security boundary is documented as instructions, not containment

- **Decision:** `AGENTS.md` section 10 and section 4 of
  `Docs/OPENCODE_AGENT_FAMILY.md` now state explicitly what the permission
  configuration does **not** enforce, and name the written prohibitions as the
  primary control for shell-capable agents.
- **Why:** the V0 review found the earlier wording **overclaimed**. It implied
  that `external_directory: deny` prevented access outside the repository, and
  that the `.env` guard protected secrets. Both claims are false for `builder`,
  `tester`, and `quant-researcher`, because `external_directory` and `read`
  govern the file tools only and do not constrain `shell`.
- **Corrected statements of fact:**
  - `external_directory` applies to `read`, `edit`, `write`, and `patch` paths,
    not to `shell`. It is a real boundary only for the three agents whose
    `shell` is denied outright.
  - `read: "*.env" -> deny` blocks the read tool only. A shell-capable agent can
    still print the file.
  - `shell` pattern rules match command text, not intent, and are best effort.
- **Consequence:** no document now claims a containment boundary that the
  configuration does not provide. A real containment boundary needs an OS-level
  sandbox, which is out of V0 scope (O-009).

### D-015 - `F:\Trading` is the independent repository root

- **Decision:** The canonical repository root for TRADING AGENT LAB is
  **`F:\Trading`**. It is an independent Git repository and is not nested inside
  any other repository.
- **Verified (V0.1):**
  - `git rev-parse --show-toplevel` returns `F:/Trading`;
    `--absolute-git-dir` returns `F:/Trading/.git`. The repository is a real
    standalone Git directory, not a worktree and not a submodule.
  - `git remote -v` returns **no output**. `.git/config` contains only `[core]`
    keys - no `[remote]`, `[user]`, or `[branch]` section. The project has no
    remote and is not published anywhere.
  - The single commit `69e5cd9` is a **root commit with no parents**, so the
    history is genuinely independent rather than grafted.
  - `F:\Trading` is **not visible** to the OpenProperty repository: it does not
    appear in `git -C F:\OpenProperty-master status --porcelain`.
  - `opencode debug agents` from `F:\Trading` resolves 12 agents, **none named
    `orkestrix-*`**. The four inherited agents defined at
    `F:\OpenProperty-master\.opencode\agent\` are no longer reachable, because
    OpenCode's ancestor-directory walk (D-011) only reaches them when the
    project is a descendant of `F:\OpenProperty-master`.
- **Consequence:** D-011 is superseded, and O-002 and O-008 are resolved in
  substance. Agent isolation is now **structural** - the inherited agents are
  not merely blocked by the allow-list, they are absent from the resolved list.
  The D-008 allow-list remains in force as defence in depth.
- **Does not change:** the commit history, the five agent definitions, the
  permission surface, or any governance rule. This decision relocates the
  project and records the verification; it authorises nothing else.

### D-016 - All five project agents launch successfully on their configured models

- **Decision:** The agent family is **runtime-validated**. No model was
  substituted, replaced, or downgraded to make a launch succeed.
- **Verified (V0.1, live launches):** each of the five agents was launched with
  a read-only probe that read known files and ran at most one read-only Git
  command. Every agent launched, returned a correct result, and reported no
  permission anomaly.

  | Agent | Configured model | Launch | Result correctness |
  | --- | --- | --- | --- |
  | `explorer` | `opencode-go/glm-5.3-flash` | SUCCESS | 353-line file and 255-line file reported correctly |
  | `quant-researcher` | `opencode-go/glm-5.3` | SUCCESS | 255-line file; `git rev-parse --abbrev-ref HEAD` returned `main` |
  | `builder` | `opencode-go/kimi-k2.7-code` | SUCCESS | 81-line file; root listing correct |
  | `tester` | `opencode-go/glm-5.3-flash` | SUCCESS | 74-line file; `git status --porcelain` correctly returned empty |
  | `reviewer` | `opencode-go/gpt-5.6-luna` | SUCCESS | 377-line file; glob returned exactly 5 agent files |

- **Model identity, verified rather than assumed:** all five agents were asked
  to report the model identity disclosed in their own session context, and to
  report it **separately** from the `model:` value in their own frontmatter, so
  that a configured value could not be mistaken for a served one. **All five
  matched, on both provider and model id:**

  | Agent | Reported provider | Reported model id | Configured frontmatter | Match |
  | --- | --- | --- | --- | --- |
  | `explorer` | `opencode-go` | `glm-5.3-flash` | `opencode-go/glm-5.3-flash` | YES |
  | `quant-researcher` | `opencode-go` | `glm-5.3` | `opencode-go/glm-5.3` | YES |
  | `builder` | `opencode-go` | `kimi-k2.7-code` | `opencode-go/kimi-k2.7-code` | YES |
  | `tester` | `opencode-go` | `glm-5.3-flash` | `opencode-go/glm-5.3-flash` | YES |
  | `reviewer` | `opencode-go` | `gpt-5.6-luna` | `opencode-go/gpt-5.6-luna` | YES |

  Reported display names were `GLM-5.3-Flash`, `GLM-5.3`, `Kimi K2.7 Code`,
  `GLM-5.3-Flash`, and `GPT-5.6 Luna` respectively.
- **Scope limit on that evidence, stated deliberately:** this is what each agent
  *reported about its own session context*. It is strong evidence that the
  configured model is the model that ran, and it independently confirms the
  `opencode-go` provider serves all four configured identifiers at runtime. It
  is **not** a server-side attestation, and it is not proof that no silent
  fallback could ever occur. No claim beyond this is made.
- **Method note:** the first probe round asked for model identity but did not
  require the provider field, and two agents returned only an observability
  flag. The V0.1 `reviewer` gate raised that as a `HIGH` overclaim, correctly.
  The affected agents were re-probed with an explicit provider requirement, and
  the table above is the result. The finding and its resolution are recorded in
  the section 2b review record.
- **Permissions, verified:** no probe tool call or shell command was denied,
  blocked, refused, or escalated for approval. The probes exercised only
  `read`, `glob`, and read-only `shell`, all of which are expected to be
  permitted for the agent that used them.
- **Non-modification, verified:** `git status --porcelain`, `git diff --stat`,
  and `git ls-files --others --exclude-standard` were all **empty** after all
  five probes. The two write-capable agents (`builder`, `tester`) launched and
  completed **without exercising `edit`**, which is the specific result this
  probe existed to demonstrate.
- **Coordinator model, verified separately:** the main session runs `Big
  Pickle`, which resolves to **`opencode/big-pickle` on the `opencode`
  provider**. It is a real, executable identifier and is **not** on
  `opencode-go`. It was left unchanged. See the D-005 precision note.
- **Consequence:** V0 exit criteria 3 and 5 are now satisfied by **live
  execution** rather than by configuration inspection alone. This closes the
  "To confirm" line that D-011 deferred to V1 for the launch question, though
  the separate inherited-agent rejection test is now moot per D-015.



---

## 2. Assumptions

Assumptions are not verified facts. Each is falsifiable, and each is a
candidate for a later decision.

### A-001 - `Trading/` is the intended permanent home of the project

> **STATUS: REJECTED BY REALITY (V0.1).** The location this assumption named no
> longer exists as the project root. Retained as the historical record of the
> risk that was correctly identified before it materialised. Superseded by
> D-015.

- **Assumed:** the project remains at `F:\OpenProperty-master\Trading`.
- **Risk:** it sits physically inside an unrelated repository's working tree.
  The nested `.git` isolates history, but the parent repo's `git status` will
  keep showing `Trading/` as untracked.
- **To confirm:** owner decision on whether to relocate the project out of the
  OpenProperty tree entirely.
- **Outcome (V0.1, verified):** the project was relocated to `F:\Trading`. The
  risk named above was real: `F:\OpenProperty-master\Trading` still exists and
  is still reported as `?? Trading/` by the parent repository. See D-015 for the
  new root, and O-010 for the duplicate that the move left behind.

### A-002 - `quant-researcher` needs local shell execution

- **Assumed:** "controlled research and quantitative analysis" justifies running
  non-destructive local computation (arithmetic checks, hypothesis simulation).
- **Implemented as:** `edit` fully denied, so the agent can compute but can never
  write a result into the project; `shell` allowed only with destructive
  filesystem, network (`curl`, `wget`), dependency-install, and Git-history
  operations denied.
- **Alternative if rejected:** deny `shell` entirely, matching `explorer`, and
  route every computation to `tester`.
- **To confirm:** owner decision on whether research should be strictly
  non-executing.

### A-003 - Fast-tier models are adequate for exploration and verification

- **Assumed:** `glm-5.3-flash` for `explorer` and `tester` is sufficient because
  both produce re-runnable, low-consequence output.
- **Risk:** a subtle dependency or coupling finding could be missed, and would
  surface later during `builder` or `reviewer` work.
- **To confirm:** observe the quality of the first few `explorer` reports.

### A-004 - No programming language or toolchain is fixed by V0

- **Assumed:** V0 establishes the work-engine only, so no language, runtime,
  package manager, test runner, or exchange SDK is selected yet.
- **Note:** `.gitignore` covers common Node and Python tooling so the repository
  is safe under either choice. This is deliberately not a language decision.
- **To confirm:** owner decision on the V1 stack.

### A-005 - Shell `resource` patterns match as intended

- **Assumed:** OpenCode matches `shell` resources as raw command text with
  wildcard support, so patterns such as `git push*` and `rm *` deny the intended
  commands.
- **Verified so far:** the V2 permissions guide confirms `*` matches zero or more
  characters, that a shell pattern ending in ` *` also matches the bare command,
  and that the last matching rule wins.
- **Known limitation, verified:** the guide also warns that "`shell` runs with
  the host user's filesystem, process, and network authority. Directory
  inference from command text is best effort", and explicitly recommends
  "a narrow shell allowlist instead of patterns intended to recognize every
  dangerous command."
- **Consequence:** the `shell` deny-lists in `builder`, `tester`, and
  `quant-researcher` are **best effort, not a hard security boundary**. A
  command that does not textually match a pattern may still execute. The
  instruction-level prohibitions in each agent prompt and in `AGENTS.md` are
  therefore load-bearing, not decorative.
- **To confirm:** a live permission test in V1 that attempts a denied action and
  observes the rejection. The V0 reviewer's suggestion that documenting the
  limitation is not sufficient is accepted; see D-014.

### A-006 - `list` is not an OpenCode V2 permission action

- **Assumed:** legacy `list: allow` from the V1 baseline is inert, and directory
  listing is covered by `read` and `glob`.
- **Verified:** the V2 permissions guide's action table lists `read`, `edit`,
  `glob`, `grep`, `shell`, `subagent`, `skill`, `question`, `webfetch`,
  `websearch`, `external_directory`, `<server>_<tool>`, and `execute`. `list` is
  absent. Unknown action names are accepted as inert strings rather than
  rejected, which is why the legacy files load without error.
- **Consequence:** no `list` rule is set in this project, and none is needed.

## 2a. V0 review record

The `reviewer` agent (`opencode-go/gpt-5.6-luna`) reviewed this workstream
before closure, per the pipeline in `AGENTS.md` section 5. Findings and
dispositions:

| # | Severity | Finding | Disposition |
| --- | --- | --- | --- |
| 1 | CRITICAL | None reported | - |
| 2 | HIGH | Shell permissions do not enforce the documented security boundary; the `external_directory` and `.env` claims were overstated for shell-capable agents. | **Resolved.** Hard-deny policies adopted (D-013); documentation corrected so the boundary is stated as instructions plus defence in depth, not containment (D-014); residual risk recorded as O-009. |
| 3 | HIGH | Closure evidence incomplete: no complete diff, no `git status`, and the inherited-agent rejection was proven only syntactically. | **Resolved.** `git status` and the full staged diff were captured and reviewed. The inherited-agent rejection is now proven **live**: an actual `orkestrix-explorer` launch returned `Permission denied: subagent`. |
| 4 | MEDIUM | None reported | - |
| 5 | LOW | None reported | - |

The reviewer returned `Not cleanly closable` while the two HIGH findings stood.
Closure proceeded only after both were resolved. This record exists so the
review gate is auditable rather than a discarded formality.

**Note added in V0.1:** finding 3's live test was performed while the project was
nested inside the OpenProperty tree, and its result is retained as history. After
the move (D-015) the four `orkestrix-*` agents no longer resolve at all, so the
rejection test can no longer be reproduced and is now moot rather than pending.
The test did its job at the time: it proved the allow-list held before the
stronger structural fix made it redundant.

## 2b. V0.1 review record

Scope of V0.1: reconcile the documentation with the filesystem and Git reality
after the move to `F:\Trading`, record the legacy duplicate as an owner decision,
and prove live launch of all five configured agents. No source code, no tests,
and no trading material were involved, so `tester` had no executable test suite
to run; the validation performed was live agent-launch verification, recorded as
D-016.

### Gate 1 - `reviewer` (`opencode-go/gpt-5.6-luna`)

The gate was run against the documentation diff and the runtime evidence. The
verdict was **`NOT cleanly closable`**. Recorded totals: **0 `CRITICAL`,
2 `HIGH`, 0 `MEDIUM`, 0 `LOW`, 8 `PASS`.**

| # | Severity | Finding | Disposition |
| --- | --- | --- | --- |
| 1 | CRITICAL | None reported | - |
| 2 | HIGH | **Evidence overclaim on model identity.** The draft asserted that three agents reported both model *and* provider, but the supplied evidence confirmed the provider for only two. It also implied that a successful launch proved the configured model served the request for the two agents that never echoed a model string. | **Resolved.** The claim was not merely softened - the gap was closed. `explorer`, `reviewer`, and `builder` were re-probed with an explicit requirement to report the provider field separately from the configured frontmatter value. All five agents then reported both fields, and all five matched their frontmatter on provider and model id. The record now states the evidence class and its limit explicitly as agent self-report, not server-side attestation. See D-016. |
| 3 | HIGH | **The V0.1 review record was stale and would have been internally dishonest.** The draft stated that the reviewer gate had run while every row of its findings table still read `pending - gate not yet run`, so the document asserted a completed gate that had not been recorded. | **Resolved.** The placeholder table was replaced with this actual Gate 1 record: the real verdict, both `HIGH` findings, and each disposition. The placeholder wording now exists only in the Gate 2 subsection, where it is accurate at the moment of writing. |
| 4 | MEDIUM | None reported | - |
| 5 | LOW | None reported | - |

The gate's positive findings were: only the two authorised files were modified;
`AGENTS.md` retains no stale external references and its V0 scope, Git, and
security rules are intact; D-011 and A-001 are preserved and clearly marked
rather than deleted; O-010 remains an open owner decision with no text marking
it resolved; no secret guard is weakened and no secret value appears in the
diff; all cross-references resolve to existing decision IDs; and no trading,
exchange, runtime, or dependency work was introduced.

### Gate 2 - `reviewer` (`opencode-go/gpt-5.6-luna`)

Run after the gate 1 findings were resolved, to confirm the fixes were real
rather than cosmetic. Verdict: **`NOT cleanly closable`**. Recorded totals:
**0 `CRITICAL`, 1 `HIGH`, 0 `MEDIUM`, 0 `LOW`, 7 `PASS`.**

| # | Severity | Finding | Disposition |
| --- | --- | --- | --- |
| 1 | CRITICAL | None reported | - |
| 2 | HIGH | **The new Gate 1 record was internally inconsistent.** Its prose stated two `HIGH` findings while the table carried only one `HIGH` row, because the H2 row had been omitted when the placeholder table was replaced. | **Resolved.** The table was renumbered to five rows and the missing H2 row was added, so the stated count and the table now agree. |
| 3 | MEDIUM | None reported | - |
| 4 | LOW | None reported | - |

Gate 2 confirmed both gate 1 findings as resolved, and its positive findings
covered the corrected identity evidence, agreement between the two documents,
resolving cross-references, intact V0 scope, Git, and security rules in
`AGENTS.md`, the retained intended models and permission boundaries in the five
agent definitions, the still-open O-010, and the still-reported absent
`.env.example`.

### Gate 3 - `reviewer` (`opencode-go/gpt-5.6-luna`)

Run to verify the gate 2 fix and complete a final consistency sweep. Verdict:
**`cleanly closable`.** Recorded totals: **0 `CRITICAL`, 0 `HIGH`, 1 `MEDIUM`,
0 `LOW`, 7 `PASS`.**

| # | Severity | Finding | Disposition |
| --- | --- | --- | --- |
| 1 | CRITICAL | None reported | - |
| 2 | HIGH | None reported | - |
| 3 | MEDIUM | The Gate 1 record did not state its `PASS` total explicitly, so the audit record was not fully quantitative. | **Resolved.** The `0 CRITICAL, 2 HIGH, 0 MEDIUM, 0 LOW, 8 PASS` tally was added to the Gate 1 record. |
| 4 | LOW | None reported | - |

Gate 3 confirmed the Gate 1 record is self-consistent, that the transient Gate 2
placeholder was acceptable for an append-only log and was not a false statement
about the past, and that the documented exit criteria, model routing, permission
surface, and security limitations are all reconciled with the evidence.

**Closure statement for V0.1.** Three gates ran. The first two returned `NOT
cleanly closable` and each `HIGH` finding was resolved before the next gate was
run; the third returned `cleanly closable` with no `CRITICAL` and no `HIGH`
outstanding. Per `AGENTS.md` section 9, no `CRITICAL` or `HIGH` finding remains
unresolved at the time of the commit.


---

## 3. Open decisions

Unresolved. Each needs an owner decision. None blocks V0 closure.

**Status column values:** `OPEN` (still needs a decision), `RESOLVED` (decided,
see the linked decision), `MOOT` (the question no longer applies).

| ID | Question | Status | Blocks | Notes |
| --- | --- | --- | --- | --- |
| O-001 | What runtime stack does V1 use? | OPEN | V1 start | Node/TypeScript, Python, or a split system. Not chosen in V0. See A-004. |
| O-002 | Should `Trading/` move out of the OpenProperty working tree? | **RESOLVED** (D-015) | - | **Done in V0.1.** The project now lives at `F:\Trading`. A-001 is rejected by reality. |
| O-003 | Does `quant-researcher` keep local shell execution? | OPEN | Nothing | See A-002. Not tested in V0.1 beyond a successful read-only shell probe. |
| O-004 | What defines Strategy Registry approval, and who grants it? | OPEN | Strategy Registry work | Only the Coordinator may change status, on explicit instruction. The approval criteria themselves are undefined. The registry does not exist on disk. |
| O-005 | Which exchange, if any, will be integrated after V0? | OPEN | Exchange work | Explicitly out of scope for V0. |
| O-006 | What are the entry and exit criteria for V1? | OPEN | V1 start | V0 exit criteria are met and recorded below. |
| O-007 | Should the baseline project's `orkestrix-*` agents be retired? | **MOOT** (D-015) | - | Nothing to retire **in this project**. They still exist at `F:\OpenProperty-master\.opencode\agent\`, but they belong to that repository and no longer resolve from `F:\Trading`. |
| O-008 | Should `Trading/` be moved out of the OpenProperty tree to remove the four inherited `orkestrix-*` agents entirely? | **RESOLVED** (D-015) | - | **Done in V0.1.** The move happened and it achieved the stated goal: `opencode debug agents` from `F:\Trading` resolves no `orkestrix-*` agent. |
| O-009 | Should shell-capable agents run inside an OS-level sandbox or container? | OPEN | Nothing in V0 | **Partly resolved in V0.** The reliably matchable Git operations were upgraded to hard-deny `experimental.policies` (D-013), and the residual risk is now documented accurately rather than papered over (D-014). What remains is true containment: a sandbox or container so that a shell-capable agent cannot reach a `.env` file or a path outside the repository. The V2 permissions guide is explicit that command-text matching is best effort, so this cannot be solved with more patterns. Recommend deciding before V1 when the stack and execution model are known. |
| O-010 | What should happen to the legacy duplicate at `F:\OpenProperty-master\Trading`? | **OPEN - OWNER DECISION REQUIRED** | Nothing technically | See below. **Not resolved by any agent, deliberately.** |

### O-010 - Legacy duplicate repository (owner decision required)

- **Verified state (V0.1):** two copies of this project exist on disk.
  - **`F:\Trading`** - the **current canonical repository**. Independent Git
    repository, no remote, branch `main`, single commit `69e5cd9`, clean working
    tree. Recorded as canonical by D-015.
  - **`F:\OpenProperty-master\Trading`** - a **legacy duplicate pending human
    resolution**. It is a *separate* Git repository on the *same* commit
    `69e5cd9`, also with a clean working tree. Its index was compared against
    `F:\Trading` and the two carry **identical blob SHAs for all 10 tracked
    files**, so the copies are byte-equivalent today.
- **Why this needs a human, not an agent.** The copies are equivalent now, but
  nothing keeps them that way. The options each carry consequences that are not
  the agent's to weigh: **retain** both and accept divergence risk; **delete**
  the legacy copy, which is irreversible and destroys the pre-move working copy
  of record; **archive** it by moving it somewhere neutral; or **reconcile** it
  deliberately. `AGENTS.md` section 9 forbids history-rewriting Git operations,
  and section 7 forbids acting outside an authorised scope. Neither copy may be
  deleted or modified by an agent on its own initiative.
- **Risk while unresolved:** the legacy copy sits inside an unrelated repository
  (`F:\OpenProperty-master`, remote `https://github.com/GarrenRs/MCP`) as an
  **untracked** directory. That parent repository already has a dirty working
  tree, so a `git clean` or an accidental recursive operation there could
  destroy the pre-move copy with no trace in that repository's history.
- **Explicitly not done by V0.1:** neither copy was deleted, modified, or
  renamed. The duplicate is **recorded, not resolved.**

---

## 4. Rejected alternatives

Recorded so the reasoning is not silently lost and re-litigated later.

### R-001 - Reuse the baseline's V1 legacy agent files verbatim

- **Rejected.** The proven `orkestrix-*` definitions use `permission:` (V1
  singular map), the `bash` action, and the singular `.opencode/agent/` path.
  Copying them would have produced configuration the V2 loader does not
  interpret as intended, and the task explicitly forbade blind copying of
  legacy syntax. The *workflow* was preserved; the *syntax* was not.

### R-002 - Define a custom `coordinator` primary agent

- **Rejected.** The main session already coordinates. A second coordinator
  would create two competing sources of authority, and the task specified that
  exactly five subagents exist. The default `build` primary agent coordinates,
  and project config scopes what it may launch.

### R-003 - Commit the work-engine into the surrounding OpenProperty repository

- **Rejected.** See D-009. It would have mixed two unrelated products into one
  history, on a remote belonging to the other project, alongside an already
  dirty working tree. A standalone repository keeps history and closure
  auditable.
- **HISTORICAL note (V0.1):** retained because it records why the project holds
  an independent history at all. The rejection stands and the independent
  history is now held at `F:\Trading` (D-015), not at the original
  `F:\OpenProperty-master\Trading`.

### R-004 - Use the V1 singular `.opencode/agent/` path for consistency with the baseline

- **Rejected.** Consistency with a legacy layout is not a goal; correctness
  against the installed V2 schema is. See D-002.

### R-005 - Set `default_agent` to a custom agent

- **Rejected.** `default_agent` changes the primary agent, not the available
  subagents, and would change the Coordinator's own identity for no benefit. The
  subagent allow-list in D-008 is the correct control.

### R-006 - Deny `shell` for every agent, including `builder` and `tester`

- **Rejected.** Both roles are defined by their ability to run development and
  verification commands. Denying it would leave them unable to satisfy their own
  mandate. The Git and network guards in section 4 of the family document are
  the proportionate boundary.

### R-007 - Block the subagent allow-list behind the built-in `general` and `explore` agents

- **Rejected.** Keeping built-ins available would leave an ungoverned route to
  a broadly-privileged generalist agent, which defeats the allow-list in D-008.

---

## 5. V0 exit criteria

V0 is complete when:

1. OpenCode project config parses against the V2 schema.
2. Exactly five agent definitions exist and load as subagents.
3. All five model identifiers resolve on the installed build. **Upgraded in
   V0.1 from configuration inspection to live launch - see D-016.**
4. Each agent's permission boundary matches section 4 of the family document.
5. The Coordinator can launch the five, and no subagent can launch another.
   **The first half is now proven by live launch - see D-016.**
6. `AGENTS.md` exists and is applied as project instructions.
7. This document and the family document exist and are consistent.
8. `.gitignore` exists and covers secrets and build artifacts.
9. `git status` shows only expected files.
10. No trading, exchange, database, UI, or strategy code exists.
11. No credential or secret is present.
12. The `reviewer` gate has run, and no `CRITICAL` or `HIGH` finding stands.
13. The Coordinator can launch all five subagents, and a live launch of an
    inherited foreign agent is **rejected** by the allow-list. **Superseded in
    V0.1:** the inherited-agent half of this criterion is now **moot** - after
    the move to `F:\Trading` the four `orkestrix-*` agents do not resolve at
    all (D-015), so there is nothing left to reject. The test did pass while the
    project was nested; that result is retained in the section 2a review record.
14. Exactly one commit closes the workstream.

### V0.1 closure notes

- **Criterion 14 note.** The V0 workstream closed with commit `69e5cd9`. V0.1 is
  a separate workstream and closes with its own single commit, so the repository
  will hold two commits. "Exactly one commit" is read per workstream, not
  cumulatively for the repository.
- **Standing item, not a criterion failure.** `AGENTS.md` section 10 requires a
  committed `.env.example` with empty values, and `.gitignore` pre-authorises one
  via `!.env.example`, but **no such file exists**. This was observed in the
  V0.1 baseline audit and is **reported, not fixed**, because creating it was not
  in the authorised scope of this workstream. It is not treated as a V0 exit
  criterion failure, since criterion 8 concerns `.gitignore` coverage, which is
  satisfied. It should be picked up as an explicit, separately authorised task.

