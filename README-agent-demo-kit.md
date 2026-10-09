# Broad Dissemination modernization: agent demo kit

Five agent definitions that form one pipeline. Each agent's output is the next agent's input, with a human gate between stages.

| Stage | Agent | File | Mode | Input | Output |
|---|---|---|---|---|---|
| 1 | Cartographer | `cartographer.agent.md` | IDE, live | One slice (e.g. an EMS topic) | `docs/inventory/<slice>-inventory.md` |
| 2 | RuleExtractor | `rule-extractor.agent.md` | IDE, live | Inventory + code | `docs/rules/<slice>-rules.csv` and `.md` |
| 2b | RuleReviewer | `rule-reviewer.agent.md` | IDE, live | Rules catalogue | `docs/rules/<slice>-review.md` |
| 3 | OracleBuilder | `oracle-builder.agent.md` | IDE, live or recorded | Approved rules + sanitised messages | Tests, golden master, `docs/oracle/<slice>-oracle-report.md` |
| 4 | ShadowDiffTriager | `shadow-diff-triager.agent.md` | Autonomous, pre-run | Shadow-run diff files | `shadow/triage/<date>/digest.md`, issue drafts, draft PRs |

## Human gates (say these out loud in the demo)
1. After stage 2b: an SME (research ops / product owner) sets each rule to Approved, Rejected or Obsolete. Agents never set this.
2. After stage 3: engineers review the generated tests and the traceability matrix.
3. After stage 4: a human reviews every PR and decides every `NEEDS_HUMAN` item. Agents never merge.

## Setup
1. Copy the `*.agent.md` files into `.github/agents/` in the repository you are demoing on.
2. Check the `tools:` lists in each file's frontmatter against what your IDE version and bank policy allow. Tool names change between Copilot versions. Remove any tool you do not have rather than leaving a stale name.
3. Pick **one narrow slice** and run stages 1 to 3 on it before the session. Keep the outputs.
4. For stage 4, run the new system in shadow against sanitised or synthetic traffic, produce diff files with the oracle compare mode, and run the triager before the session. Keep the digest, one issue draft and one draft PR.
5. If you can wire stage 4 to a scheduled workflow, do it as: scheduled job runs the compare, writes diffs to `shadow/diffs/<date>/`, then opens an issue assigned to the coding agent with the ShadowDiffTriager instructions. Confirm with your platform team what the coding agent is permitted to do in your org before promising this.

## Suggested demo run-of-show (about 15 minutes)
1. **Framing (1 min):** Faraz changed how he works. We tried to disprove it works inside UBS.
2. **Cartographer, live (3 min):** run on the slice. Open the hidden logic register, click through to a cited trigger or stored proc line.
3. **RuleExtractor + RuleReviewer (4 min):** show a few catalogue rows, one flagged `SUSPECTED_DEAD`, and the reviewer's verdict table including at least one row it marked `PARTIAL` or `UNSUPPORTED`. State that the SME decides.
4. **OracleBuilder (3 min):** show green tests against legacy, then show a mutated rule failing with its Rule ID. Show the traceability matrix.
5. **ShadowDiffTriager, pre-run (3 min):** show the digest headline, one needs-human question, one draft PR with passing checks, and your review comment.
6. **Close (1 min):** what was proved, what was not, and the ask: pilot scope, team, 60-day success measure.

## Evidence to capture beforehand
- Time taken by each agent versus your estimate for doing the same by hand.
- Counts: inventory items, rules, `SUSPECTED_DEAD` rules, reviewer verdicts, tests, diffs, clusters, PRs.
- At least one mistake the agents made and which control caught it.
- Screen recordings of every stage as a fallback.

## Controls to state explicitly
- Sanitised or synthetic data only, with the data classification confirmed.
- Read-only agents for discovery; write access limited to test code and branches.
- No agent merges or deploys. Every change goes through normal review.
- Every claim is cited to code, and a second agent re-checks the citations.
- Full audit trail: files read, commands run, branches and PRs created.
