# claude-skills

A growing library of Claude skill files for software engineers — covering system design, code review, documentation, and more.

Skill files teach Claude environment-specific best practices so you get consistently structured, opinionated output without re-explaining your preferences every time.

---

## What's a skill file?

A skill file (`SKILL.md`) is a markdown file you install into Claude. When you ask something that matches the skill's trigger, Claude reads the skill first and follows its instructions — producing output that's structured, thorough, and tailored for engineers.

Think of it as a reusable "how I want Claude to work" config that travels with you across projects and machines.

---

## Skills

| Skill | What it does | Trigger phrases |
|-------|-------------|-----------------|
| [`system-design-doc`](./skills/system-design-doc/) | Generates a full system design document — problem statement, architecture diagram, data model, API design, trade-offs, and gotchas | "design a X system", "create a design doc for Y", "how would you architect Z" |

More skills coming. Contributions welcome — see [CONTRIBUTING.md](./CONTRIBUTING.md).

---

## Installation

### Claude.ai (web / desktop / mobile)

1. Download the `.skill` file from the [releases page](../../releases) or the skill's folder
2. Open Claude.ai → **Settings** → **Skills** → **Install from file**
3. Select the `.skill` file
4. Done — Claude will now use the skill automatically when relevant

### Claude Code (CLI)

```bash
claude skill install system-design-doc.skill
```

---

## Usage

Once installed, just describe what you want — no special syntax needed.

**Examples that trigger `system-design-doc`:**

```
Design a URL shortener like bit.ly
```
```
How would you architect a real-time notification service for 10M users?
```
```
Create a design doc for a ride-sharing app backend
```
```
Help me think through the design for a distributed job scheduler
```

Claude will ask a few clarifying questions if context is missing, then produce a structured doc covering:

1. Problem Statement
2. Requirements (functional + NFRs)
3. Capacity Estimation
4. High-Level Architecture (with diagram, if you have connected tools)
5. Data Modeling
6. API Design
7. Scalability & NFR Mapping
8. Trade-off Analysis
9. Gotchas & Failure Modes
10. Out of Scope / Future Considerations

**Output format:** Ask for Markdown (default) or a Word doc (`.docx`) you can share with your team.

**Depth:** Default is interview-level (concise, readable in 10 minutes). Say "production-level" or "for my team" for a more detailed output.

---

## Example output

See [`skills/system-design-doc/examples/url-shortener.md`](./skills/system-design-doc/examples/url-shortener.md) for a full sample output.

---

## Building the `.skill` file yourself

If you've cloned the repo and want to package a skill locally:

```bash
# Requires the Anthropic skill creator scripts
# Clone this repo, then:
cd claude-skills
python -m scripts.package_skill skills/system-design-doc ./dist
```

Or just download the pre-packaged `.skill` file from [releases](../../releases).

---

## Contributing

Contributions are welcome — new skills, improvements to existing ones, or better example outputs.

See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines. The short version:

- Open an issue to propose a new skill before building it
- Each skill needs a `SKILL.md` and at least one example output
- PRs should include a brief description of what the skill does and 2–3 example trigger phrases

---

## Roadmap

Skills planned or in progress:

- `java-code-review` — structured Java code review with thread safety, exception handling, and complexity checks
- `unit-test-generator` — JUnit/Mockito test generation with edge cases and mocking strategy
- `pr-description-writer` — structured PR descriptions from a diff or change summary
- `adr-writer` — Architecture Decision Record generator

Have an idea? [Open an issue](../../issues/new).

---

## License

MIT — free to use, share, and build on.
