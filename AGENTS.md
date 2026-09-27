# AGENTS.md - TRADING AGENT LAB

Binding operating rules for every agent working in this repository.
Read this file before any task. Authority: this file governs conduct;
`Docs/OPENCODE_AGENT_FAMILY.md` governs agent roles, model routing, and
permissions.

---

## 1. Project purpose

TRADING AGENT LAB is a controlled environment for **researching, specifying,
and eventually building** quantitative trading systems. It exists so that
trading strategy work is done from evidence, under review, and with an
explicit boundary between research, simulation, and real capital.

---

## 2. Two separate systems - never mix these

| System | What it is | Where it lives |
| --- | --- | --- |
| **A. OpenCode engineering agents** | The five development subagents in this repo | `.opencode/agents/` |
| **B. Runtime Trading Agent** | Application code that trades | Built later, **inside the app** |

The Runtime Trading Agent is **not** an OpenCode subagent and must never be
described as one. No OpenCode agent trades, places orders, or holds funds.

---

## 3. Architecture governance

- Architecture changes require explicit Coordinator authorisation, stated in
  writing, before any code is written.
- Public contracts are **stable**. Changing one is a scope change.
- Layering must be preserved. Logic does not move across layers to "just fix it".
- New dependencies require explicit approval, with a stated reason.
- If an agent believes the architecture is wrong, it **reports**; it does not
  unilaterally redesign.

---

## 4. Current scope: V0 - work-engine establishment

**V0 contains only the OpenCode work-engine and its governance documents.**

V0 explicitly does **NOT** include, and no agent may begin:

- trading code or the trading core
- Binance or any exchange integration
- a database or persistence layer
- a dashboard or any UI
- strategy implementations
- backtest engines
- live order execution of any kind

Do not begin application implementation. Wait for the Coordinator to open the
next phase.

---

## 5. Agent workflow

The Coordinator is the **main session itself**. It is not a subagent, and no
competing coordinator agent exists.

The proven pipeline, per workstream:

```
Baseline -> Audit -> Bounded Plan -> Controlled Execution -> Test
         -> Review -> Documentation -> Clean Closure -> Git Commit
```

Default routing:

| Stage | Agent |
| --- | --- |
| Research / discovery | `explorer` (+ `quant-researcher` when needed) |
| Implementation | `builder` |
| Verification | `tester` |
| Final review | `reviewer` |
| Closure and commit | Coordinator |

Rules:

- **Subagents never invoke subagents.** Sequencing belongs to the Coordinator.
- **Never run two write-capable agents on the same files concurrently.** Read-only
  agents may run in parallel when there is no shared mutable state.
- The Coordinator may skip any agent the current task does not need.

Full detail: `Docs/OPENCODE_AGENT_FAMILY.md`.

---

## 6. Evidence tags

Every report separates what is known from what is guessed. Use these tags
literally:

- `VERIFIED` - observed directly, with a file and line or an authoritative source.
- `INFERENCE` - derived from verified evidence; state the reasoning.
- `RECOMMENDATION` - a proposal, never a decision.
- `UNKNOWN` - not determined; state what would be needed.

Never present an `INFERENCE` or a `RECOMMENDATION` as a `VERIFIED` fact.

---

## 7. No scope expansion

**Implement the authorised task and nothing else.**

Forbidden without explicit instruction:

- unrequested features, "improvements", or speculative refactors
- adjacent cleanup, formatting sweeps, opportunistic renames
- extra dependencies
- contract or architecture changes
- work on a future phase

If you see something worth fixing outside the task, **report it. Do not fix it.**

---

## 8. Testing expectations

- A task without a test is not finished.
- Tests must assert real behaviour. Weak assertions that cannot fail are defects.
- Never weaken, delete, skip, or re-scope a test to force a pass.
- Always cover the failure path, not only the happy path.
- The `tester` applies **minimal, narrowly-scoped** fixes only, and must
  distinguish `ENVIRONMENT` failures from `APPLICATION` failures.
- Report `PASS`, `FAIL`, and `NOT RUN` honestly. Never claim an unrun test passed.

---

## 9. Git rules and clean closure

A workstream closes only in this order:

```
Baseline -> Work -> Verify -> Review -> Documentation -> Git Status
         -> Clean Closure -> Commit
```

**The Coordinator must not allow a commit when any of these hold:**

- tests are failing
- the authorised scope is incomplete
- documentation is inconsistent with the code
- any `CRITICAL` or `HIGH` reviewer finding is unresolved
- unexpected files are modified
- a secret is present

Also mandatory:

- Conventional Commit messages (`feat:`, `fix:`, `docs:`, `chore:`, `test:`, `refactor:`).
- **Never push automatically.** Push is always a human decision.
- **Never rewrite Git history.** No `reset`, `rebase`, or force-push.
- No destructive Git commands, from any agent.
- Subagents **never** commit. Only the Coordinator commits.
- Review the complete diff before committing, not just the message.

---

## 10. Security rules

- **No secrets, ever.** No API keys, tokens, private keys, or `.env` files in
  the repository.
- Reference secrets only through environment variables; commit a
  `.env.example` with empty values.
- `.gitignore` must cover `.env*`, `*.pem`, `*.key`, and credential directories.
- If you discover a secret, **stop and report its location**. Never quote its value.
- No live exchange credentials may be added at any point in V0.
- Treat any code path that could place a live order, withdraw, or transfer as
  forbidden. Finding one is a reportable finding, not a design detail.

### These rules are instructions, not a sandbox

**You are the control.** The permission configuration is defence in depth, not
containment. Know exactly what it does and does not stop:

- `read: "*.env" -> deny` blocks reading a secret file **with the read tool
  only**. A shell-capable agent can still print it with `cat`, `type`, or
  `Get-Content`. Never do so. If a command would surface a secret, do not run it.
- `external_directory -> deny` governs the `read`/`edit`/`write`/`patch` path
  checks only. It does **not** constrain `shell`. A shell-capable agent is not
  confined to this repository; stay inside it deliberately.
- `shell` deny patterns such as `git push*` and `rm *` are **best effort**.
  Permission rules match command text, not intent. An unmatched or
  reformulated command can still run.
- `git push`, `git reset`, `git rebase`, `git clean`, and `git filter-branch`
  are hard-denied for **everyone** by `experimental.policies`, which no
  permission rule and no saved approval can override. `git commit` is
  hard-denied for all five subagents at agent level. That protection is real,
  and it is limited to those operations.

Never rely on the configuration to stop you. Rely on your own judgement, and
escalate when an instruction and a command conflict.


---

## 11. Trading and live-execution prohibition (V0)

**During V0, the following are absolutely prohibited:**

- submitting, enabling, or even wiring up live order execution
- withdrawal capability of any kind
- transfer or fund-movement capability
- real-money execution
- holding, requesting, or storing live exchange credentials

Research, backtesting, and paper trading are the only permitted activity, and
only once the Coordinator opens the relevant phase. Even then, live trading
requires a separate, explicit, human decision that no agent may make on its own.

---

## 12. Documentation authority

| Document | Authority over |
| --- | --- |
| `AGENTS.md` | Conduct, scope, workflow, security, Git, testing |
| `Docs/OPENCODE_AGENT_FAMILY.md` | Agent roles, model routing, permissions, invocation, closure |
| `Docs/OPENCODE_AGENT_FAMILY_DECISIONS.md` | Verified decisions, assumptions, open decisions, rejected alternatives |

Rules:

- These three files are authoritative. When an agent prompt and a document
  conflict, **the document wins**; report the conflict to the Coordinator.
- Documentation must match actual behaviour. Stale documentation is a `HIGH`
  reviewer finding.
- A workstream is not closable while documentation is inconsistent.
- Record new decisions in the decisions register rather than in chat.
- If something is not documented, it is not agreed. Do not proceed on the
  assumption that it is.

---

## 13. Resume and checkpoint policy

When resuming an interrupted session, in order:

1. `git status`
2. `git diff --stat`
3. the current task state
4. the latest checkpoint or decision record
5. re-run the previously failing verification, when applicable
6. proceed only once the state is understood

If a task cannot be completed in this session, record: work completed, work
incomplete, the exact next step, blockers, and the relevant files.

**Never present an interrupted task as complete.**
