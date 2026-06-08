# Contributing to claude-skills

Thanks for wanting to contribute. This project grows by engineers sharing skills they've found useful — new skills, improvements to existing ones, and better examples are all welcome.

---

## Ways to contribute

- **New skill** — build and submit a skill for a workflow not yet covered
- **Improve an existing skill** — better instructions, edge case handling, or output quality
- **Add an example output** — run a skill on a real prompt and commit the output as a reference
- **Fix documentation** — README, skill descriptions, typos

---

## Before you build a new skill

Open an issue first using the **New Skill Proposal** template. This avoids duplicate work and lets us agree on scope before you invest time building it.

In the issue, describe:
- What the skill does
- Who it's for
- 2–3 example prompts that should trigger it
- What the output looks like

Once there's a rough agreement, go ahead and build it.

---

## Skill file structure

Each skill lives in its own folder under `skills/`:

```
skills/
└── your-skill-name/
    ├── SKILL.md          ← required: the skill instructions
    └── examples/
        └── example-1.md  ← required: at least one sample output
```

### SKILL.md format

Every `SKILL.md` must start with a frontmatter block:

```markdown
---
name: your-skill-name
description: >
  One or two sentences describing what this skill does.
  Include example trigger phrases so Claude knows when to activate it.
---
```

The rest of the file is the skill instructions — written for Claude to follow, not for humans to read. Be specific and opinionated. Vague instructions produce vague output.

**Good skill instructions:**
- Tell Claude what structure to follow
- Specify what to emphasize and what to skip
- Include a depth/calibration guide (when to be concise vs detailed)
- Call out common mistakes or gotchas to highlight

**Avoid:**
- Generic advice Claude already knows ("be helpful", "be clear")
- Instructions that are just a list of topics with no guidance on how to handle them
- Over-engineering — if it fits in 100 lines, don't write 300

### Example outputs

Include at least one real output in `examples/`. Run the skill on a concrete prompt and commit the result. This helps reviewers evaluate quality and gives users a preview before installing.

Name example files descriptively: `url-shortener.md`, `notification-service.md`, not `example-1.md`.

---

## Submitting a PR

1. Fork the repo and create a branch: `git checkout -b skill/your-skill-name`
2. Add your skill folder under `skills/`
3. Add a row to the skills table in `README.md`
4. Open a PR with:
   - What the skill does (one sentence)
   - Example prompts that trigger it
   - Link to the issue it addresses (if applicable)

### PR checklist

- [ ] `SKILL.md` has valid frontmatter (`name` and `description`)
- [ ] At least one example output in `examples/`
- [ ] README skills table updated
- [ ] Skill tested on 2–3 different prompts — outputs are consistent and useful
- [ ] No hallucinated library names or tool references in the skill instructions

---

## Improving an existing skill

For small fixes (wording, typos, minor instruction tweaks) — just open a PR directly.

For bigger changes (restructuring sections, changing default behavior, adding new output formats) — open an issue first to discuss, since it may affect existing users.

---

## Packaging a `.skill` file

Pre-packaged `.skill` files are attached to GitHub releases, not committed to the repo. When your PR is merged, a maintainer will package and attach it to the next release.

If you want to test packaging locally:

```bash
python -m scripts.package_skill skills/your-skill-name ./dist
```

This requires the Anthropic skill creator scripts. See the [skill creator docs](https://docs.claude.com) for setup.

---

## Code of conduct

Be direct, be helpful, assume good intent. Reviews will be honest — if a skill produces poor output, that'll be the feedback. Nothing personal.

---

## Questions?

Open an issue or start a discussion. Happy to help you get a skill across the finish line.
