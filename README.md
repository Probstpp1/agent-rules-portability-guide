# Portable Repository Rules for Coding Agents


> **Decision:** Use this guide when one canonical repository policy must be adapted across coding-agent products with different discovery and precedence rules; skip it if identical filenames are being mistaken for identical behavior.
>
> **For:** multi-agent repository maintainers, platform teams, and policy reviewers  ·  **Not for:** teams looking for a universal instruction file or a place to store secrets

Copying one instruction file into eight filenames does not create equivalent behavior. Tools differ in discovery, scope, precedence, path targeting, imports, context limits, and product surfaces.

The portable unit should be a small **canonical rule inventory** plus reviewed adapters. Treat repository instructions as untrusted input: never place secrets, account data, tokens, or blanket authority to deploy/delete/publish in them.

## Canonical core versus adapters

Keep durable facts in a neutral inventory:

```yaml
id: test-command
scope: repository
rule: Run `python -m pytest` for Python changes.
evidence: pyproject.toml
safety: command is local; no network or credentials
```

Adapters translate only what the target mechanism can express. Example:

- Canonical: “Files under `docs/` use the docs link checker; do not publish.”
- Codex/Claude directory adapter: scoped instruction in the nearest supported project file.
- Copilot/Cursor path adapter: path-targeted rule where officially supported.
- Aider adapter: concise convention text, noting it is context rather than an enforcement policy.
- Unsupported feature: keep in the inventory as `not represented`; do not silently broaden it repository-wide.

The fillable inventory is [`assets/instruction-inventory.csv`](assets/instruction-inventory.csv).

## Compatibility worksheet—not universal facts

Complete this from the exact installed/tool documentation version before migration:

| Tool | Documented mechanism | Scope/precedence questions to verify | Path/import questions | Evidence used here |
|---|---|---|---|---|
| OpenAI Codex | `AGENTS.md` project instructions | root-to-working-directory discovery, override/fallback behavior in installed version | nested scope and fallback filenames | [official implementation source](https://raw.githubusercontent.com/openai/codex/main/codex-rs/core/src/agents_md.rs) |
| GitHub Copilot | repository custom instructions and documented path-specific mechanisms | which Copilot surface consumes each type | path pattern applicability | [GitHub documentation](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot) |
| Claude Code | project memory/instruction files | hierarchy, user/project/local interaction | imports and directory scope in current release | [Anthropic documentation](https://docs.anthropic.com/en/docs/claude-code/memory) |
| Cursor | project rules with metadata | activation/always-apply behavior and legacy migration | patterns/rule references in current docs | [Cursor rules documentation](https://docs.cursor.com/en/context/rules) |
| Windsurf | workspace rules/memories | workspace/global interaction and limits | scope/activation in current release | [Windsurf documentation](https://docs.windsurf.com/windsurf/cascade/memories) |
| Aider | configured conventions file | when/how file enters context | no assumption of hierarchy | [Aider conventions documentation](https://aider.chat/docs/usage/conventions.html) |
| Continue | rules | applicability in installed extension/mode | current rule format | [Continue documentation](https://docs.continue.dev/customize/rules) |
| Cline | repository/user rules | mode/release precedence | repository and user rule behavior | [Cline documentation](https://docs.cline.bot/features/cline-rules) |

Copying the same prose never proves equivalent execution. The compatibility worksheet and canonical-to-adapter workflow are **author recommendations in this guide**, not portability guarantees made by the eight tools.

## Harmless nested-fixture test

Before migrating real rules, create a disposable repository:

```text
fixture/
  <root instruction>: RULE_ID=root ; expected answer ROOT
  src/
    <nested instruction>: RULE_ID=nested ; expected answer NESTED
    probe.txt
  docs/
    probe.txt
```

Ask the agent, without tool or network access, to report the visible `RULE_ID` for each file and explain where it came from. Then test:

1. root-only discovery;
2. nested override/merge;
3. path-targeted activation;
4. import/reference behavior if documented;
5. conflicting rule handling;
6. maximum-size/truncation behavior using harmless numbered lines;
7. behavior after changing working directory/tool mode.

Record tool version, configuration, working directory, loaded-file evidence if exposed, and result. Do not test precedence using destructive commands or secrets.

## Migration workflow

1. Inventory every current instruction and its source/evidence.
2. Remove duplicated policy and resolve contradictions in the canonical core.
3. Mark each rule as durable fact, workflow preference, safety boundary, or tool-specific behavior.
4. Map the rule to each tool: exact, adapted, unsupported, or needs fixture test.
5. Generate concise adapters; avoid stuffing all tool docs into every context.
6. Run the harmless fixture and ordinary build/test task.
7. Human-review adapters before commit. Repository text cannot authorize external publication, account changes, or credential access.
8. Re-run after tool upgrades and preserve prior results.

## Security boundary

A cloned repository may contain malicious instructions. Agents and maintainers should review instruction-file diffs like executable configuration, apply least privilege, and require explicit human approval for external side effects. Rules should describe project workflows—not carry credentials or instruct the agent to bypass user/system policy. A rule copied from another tool is data until its scope and safety are verified.

## Evidence hierarchy

A documentation page supports only the mechanism/version it describes. The worksheet deliberately says “verify” where behavior may drift. Prefer current official docs for public guidance; implementation source is useful where docs are blocked or ambiguous but can change and may not describe every product surface. Local behavior tests are version-specific observations. Label each claim accordingly; do not flatten all three into “officially portable.”

## Maintenance

Re-run the compatibility worksheet and fixture after tool/version/config changes. Change the canonical core only when the project fact changes. Adapter-only changes must not silently rewrite the underlying policy.

Sources: [`sources/sources.json`](sources/sources.json).

## Adjacent guides

- [repo-agent-prompt-injection-tests](https://github.com/Ozekmiami/repo-agent-prompt-injection-tests) — Use it to test whether repository text can override the authorized task after adapters are installed.
- [coding-agent-eval-starter](https://github.com/Dallashubgit/coding-agent-eval-starter) — Use it when portable rules need to be exercised across comparable coding-agent task lanes.
- [ai-patch-review-playbook](https://github.com/ColomboEA/ai-patch-review-playbook) — Use it when an agent operating under those rules returns a patch for human review.

## License

Original guide, fixture protocol, and inventory template are available under CC BY 4.0. Tool names/docs retain their terms.

If this saved you a decision, star the repo — it is how the next reader finds it.
