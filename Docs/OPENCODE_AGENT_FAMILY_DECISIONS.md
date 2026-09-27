# OPENCODE AGENT FAMILY - DECISION REGISTER

**Status:** Active
**Scope:** Phase V0 - OpenCode work-engine establishment.
**Rule:** New decisions are recorded here, not in chat. An undocumented decision
is not an agreed decision.

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

### D-010 - Role separation is preserved

- **Decision:** Exactly the five specified agents exist. No additional permanent
  agents are created in V0.
- **Rationale:** Preserves the proven baseline's non-overlapping roles:
  explorer researches, the builder implements, the tester verifies, and the
  reviewer gates.

### D-011 - Parent-project agents remain loaded; the allow-list is the control

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



---

## 2. Assumptions

Assumptions are not verified facts. Each is falsifiable, and each is a
candidate for a later decision.

### A-001 - `Trading/` is the intended permanent home of the project

- **Assumed:** the project remains at `F:\OpenProperty-master\Trading`.
- **Risk:** it sits physically inside an unrelated repository's working tree.
  The nested `.git` isolates history, but the parent repo's `git status` will
  keep showing `Trading/` as untracked.
- **To confirm:** owner decision on whether to relocate the project out of the
  OpenProperty tree entirely.

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


---

## 3. Open decisions

Unresolved. Each needs an owner decision. None blocks V0 closure.

| ID | Question | Blocks | Notes |
| --- | --- | --- | --- |
| O-001 | What runtime stack does V1 use? | V1 start | Node/TypeScript, Python, or a split system. Not chosen in V0. See A-004. |
| O-002 | Should `Trading/` move out of the OpenProperty working tree? | Nothing | See A-001. Purely a hygiene decision. |
| O-003 | Does `quant-researcher` keep local shell execution? | Nothing | See A-002. |
| O-004 | What defines Strategy Registry approval, and who grants it? | Strategy Registry work | Only the Coordinator may change status, on explicit instruction. The approval criteria themselves are undefined. |
| O-005 | Which exchange, if any, will be integrated after V0? | Exchange work | Explicitly out of scope for V0. |
| O-006 | What are the entry and exit criteria for V1? | V1 start | V0 exit criteria are met and recorded below. |
| O-007 | Should the baseline project's `orkestrix-*` agents be retired? | Nothing | They belong to the OpenProperty repository, not this project. D-011 records that they still resolve here. |
| O-008 | Should `Trading/` be moved out of the OpenProperty tree to remove the four inherited `orkestrix-*` agents entirely? | Nothing | The `subagent` allow-list already blocks launching them (D-011). Moving the project would also satisfy A-001. Purely a hygiene decision. |
| O-009 | Should shell-capable agents run inside an OS-level sandbox or container? | Nothing in V0 | **Partly resolved in V0.** The reliably matchable Git operations were upgraded to hard-deny `experimental.policies` (D-013), and the residual risk is now documented accurately rather than papered over (D-014). What remains is true containment: a sandbox or container so that a shell-capable agent cannot reach a `.env` file or a path outside the repository. The V2 permissions guide is explicit that command-text matching is best effort, so this cannot be solved with more patterns. Recommend deciding before V1 when the stack and execution model are known. |

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
3. All five model identifiers resolve on the installed build.
4. Each agent's permission boundary matches section 4 of the family document.
5. The Coordinator can launch the five, and no subagent can launch another.
6. `AGENTS.md` exists and is applied as project instructions.
7. This document and the family document exist and are consistent.
8. `.gitignore` exists and covers secrets and build artifacts.
9. `git status` shows only expected files.
10. No trading, exchange, database, UI, or strategy code exists.
11. No credential or secret is present.
12. The `reviewer` gate has run, and no `CRITICAL` or `HIGH` finding stands.
13. The Coordinator can launch all five subagents, and a live launch of an
    inherited foreign agent is **rejected** by the allow-list.
14. Exactly one commit closes the workstream.

