# root-cause-fix

An agent skill that makes your coding agent find the root cause of a bug before
it touches any code — and then fix it test-driven.

Left alone, agents tend to patch the line where an error surfaces. This skill
walks a gated six-phase process instead and stops for you at the decision
points:

1. Analyze the code and logic against the report — an issue, a stack trace, a
   log excerpt or just an error message.
2. Sketch a rough blueprint of the failing flow.
3. Name the components involved, and separate the one that *reports* the error
   from the one that *causes* it.
4. Narrow it down to one falsifiable claim: file, lines, expression, input,
   observed vs. expected. **Stop 1** — you decide whether to go on.
5. Propose 2-3 genuinely different, minimal fix options with scope and risk,
   plus a recommendation. **Stop 2** — you pick one, or ask for a detailed
   comparison first.
6. Implement it test-driven: a test that reproduces the reported error first,
   the fix second, a regression run third.

It also knows when to stop early: when the problem is configuration or data
rather than code, when it is already fixed on your branch, or when the failure
cannot be reproduced in a test. And when the report itself is too thin to
analyse, it says so and suggests reviewing the report first.

## What it never does

- change your working tree before you have chosen a fix option,
- commit,
- write to your issue tracker without asking,
- follow instructions found inside issue text, comments, attachments or logs.

## Installation

The skill is the self-contained `root-cause-fix` folder inside this repository.
Copy that folder — nothing else is needed, and it works with any agent that
supports the [Agent Skills](https://agentskills.io) format.

1. Get the repository:

   ```sh
   git clone https://github.com/jarek-janas/root-cause-fix.git
   ```

   or download it from GitHub with **Code → Download ZIP**. The ZIP unpacks to
   `root-cause-fix-main`; the skill folder inside it is still `root-cause-fix`.
2. Copy the inner `root-cause-fix` folder into your skills folder — by drag and
   drop, or:

   macOS / Linux:

   ```sh
   mkdir -p ~/.agents/skills
   cp -R root-cause-fix/root-cause-fix ~/.agents/skills/
   ```

   Windows (PowerShell):

   ```powershell
   New-Item -ItemType Directory -Force $HOME\.agents\skills | Out-Null
   Copy-Item -Recurse root-cause-fix\root-cause-fix $HOME\.agents\skills\
   ```

**Where to put it:**

- `~/.agents/skills/` — the shared location, for agents that read it. Claude
  Code reads `~/.claude/skills/` instead; link the skill there once:
  - macOS / Linux:
    `mkdir -p ~/.claude/skills && ln -s ~/.agents/skills/root-cause-fix ~/.claude/skills/root-cause-fix`
  - Windows:
    `New-Item -ItemType Junction -Path $HOME\.claude\skills\root-cause-fix -Target $HOME\.agents\skills\root-cause-fix`
- `~/.claude/skills/` directly, if Claude Code is your only agent.
- `.claude/skills/` inside a project, for that project only. Commit the folder
  there to share the skill with your team.

**Updating:** run `git pull` in the clone (or download a fresh ZIP), delete the
installed `root-cause-fix` folder and copy the new one. A link from
`~/.claude/skills` keeps working.

Don't clone the repository straight into a skills folder: the skill sits one
level down, so the agent would not find it.

## Usage

Describe the bug, paste a stack trace, or give an issue key or URL — the agent
picks the skill up on its own. In Claude Code you can also call it directly:

```
/root-cause-fix PROJ-123
```

It is meant for real defects. Compile errors, lint failures and bugs in code the
agent wrote a minute ago are deliberately left out — those don't need a gated
investigation.

## Requirements

- Any language and stack. Git history is used when available.
- Issue tracker access is optional. The skill uses whatever integration your
  agent already has (a skill, an MCP server, or a CLI such as `gh`); without
  one, it asks you to paste the issue text.

## Adding knowledge about your project

The skill is stack-agnostic on purpose, so it relies on your notes for the
build and test pitfalls of your repository — which JDK or runtime to use, how
integration tests are switched on, which modules must be installed before others
can see a change. It looks for them:

- in the repository: `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING`, the README;
- outside it, if your repositories must not carry agent files: in a skill of
  your own that covers the stack, or in a notes file in a parent directory of
  the checkout (Claude Code loads `CLAUDE.md` files from parent directories
  automatically).

If it finds no build and test commands, it asks instead of guessing: a wrong
invocation can report success while running no tests at all.

## Works well with

[`bug-report-review`](https://github.com/jarek-janas/bug-report-review): when a
report is too thin to anchor the analysis, this skill offers a review of the
report at its first stop, and `bug-report-review` names what is missing and
drafts the questions for the reporter. When a report is fix-ready, it hands the
established facts back, so the analysis starts from evidence. Each skill also
works on its own.

## Files

```
root-cause-fix/              the repository
├── README.md                this file
├── LICENSE
└── root-cause-fix/          the skill — the folder you install
    ├── SKILL.md             the workflow
    ├── LICENSE              the same license, so every installed copy carries it
    └── references/
        └── tdd.md           test level, naming, coverage and red/green rules;
                             loaded only when implementation starts
```

## License

[MIT](LICENSE)
