<p align="center"><img src="logo.png" width="120" alt="logo" /></p>

# debug-code

A portable Agent Skill for diagnosing hard production and legacy-code bugs when production access is partial or unavailable. It teaches a coding assistant to find the **earliest violated invariant**, not merely the line where the failure surfaces, and to produce the smallest safe fix with proof proportionate to the available access.

**[→ Live site](https://cskwork.github.io/debug-code-skill/)** · **[→ SKILL.md](SKILL.md)**

## Why this skill

Production bugs are hard because access is partial: you have the code, but logs are on request, the DB needs an operator, and tracing may be missing. The skill does not block on ideal access. It works in **code-only mode** and tightens proof as evidence becomes available.

Core behaviors the skill enforces:

- **Incident gate** — if the symptom suggests active harm (data corruption, duplicate financial effects, unbounded resource exhaustion), it recommends containment and evidence preservation first.
- **Access mapping** — asks once what evidence sources exist, then continues immediately. Code-only is valid.
- **Fact / Inference / Unknown** — separates observed production facts from code-only reasoning. Never presents an inference as a fact.
- **Bug chain** — traces `trigger → first invalid state → propagation → visible symptom`, citing concrete code at every link.
- **Signal hierarchy** — builds the strongest available reproduction loop: red test > production evidence loop > surrogate fixture > static-only analysis. Static-only lowers confidence but does not block.
- **Falsifiable hypotheses** — generates 3-5 hypotheses before editing, each with a prediction and a cheapest probe. Uses parallel read-only investigators when supported.
- **Surgical probes** — every probe maps to a prediction. Bounded, human-run, read-only production requests with explicit scope, safety, and return shape.
- **Regression test before fix** — locks the bug down at the closest truthful seam, then applies the smallest patch that repairs the earliest violated invariant.
- **Adversarial verification** — reruns the strongest signal, challenges the result, and reports local verification separately from production verification.
- **No symptom hiding** — rejects catch-alls, silent defaults, broad retries, and larger timeouts unless that behavior is the explicit contract.

## Install

```bash
# global, all projects
git clone https://github.com/cskwork/debug-code-skill ~/.claude/skills/debug-code

# this project only
git clone https://github.com/cskwork/debug-code-skill .claude/skills/debug-code
```

The folder name and frontmatter name must remain `debug-code`.

## Use

Invoke it explicitly or describe a concrete production symptom, error, intermittent failure, performance regression, or focused suspected legacy path. Example:

```text
/debug-code Production returns duplicate invoices after a worker retry. We have code,
Grafana, dev DB, and can ask an operator to run a bounded read-only MySQL query.
```

The skill works in code-only mode. Additional access improves proof strength but is not assumed.

## Package

```text
debug-code/
├── SKILL.md
├── references/
│   ├── production-probes.md
│   └── production-bug-patterns.md
└── evals/
    └── trigger-cases.md
```

## Design references

- Matt Pocock, `diagnosing-bugs`: https://github.com/mattpocock/skills
- Dietrich Gebert, Ponytail: https://github.com/DietrichGebert/ponytail
- Remy Sharp, "The Art of Debugging": https://remysharp.com/2015/10/14/the-art-of-debugging
- Google SRE, "Troubleshooting Methodology": https://sre.google/sre-book/effective-troubleshooting/
- OWASP Logging Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html

## License

MIT — see [LICENSE](LICENSE).
